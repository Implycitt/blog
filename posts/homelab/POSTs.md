# POST

I'm just going to talk a little bit about a small sub-project I've been helping with that involves me being able to use my homelab in a useful way. I'm not going to describe a step-by-step guide here, just a high level (ish) overview of what I did and why.

## Motivation

As mentioned previously, one of the recent uses (as of writing) for the server has been holding a `.db` file. This database stores temperature, humidity, and CO2 readings which you can [download and read yourself](https://github.com/Implycitt/scifair/tree/main/Data). The idea behind this project is that an ESP32 will connect to the network and push the readings that it takes to the NAS since the microcontroller has no storage (it actually has ~0.5MiB but still). In an ideal world, my code would be able to just take data and pass it directly to the NAS. Reality is often disappointing:

1. The ESP can't see the container which lives on `192.168.100.0/24`. This is a private bridge inside Proxmox.
2. The ESP has no SMB client. This means that it can't just open a file and start writing to it.

There are a couple other things that make taking a different approach much more attractive such as SQLite locking, idempotency, and acknowledgement.

Instead, a NAT port-forward is being hosted on a separate container. The container acts like an endpoint that listens for POSTs from the ESP32. Now, the ESP POSTs to `192.168.100.204:8080` and then the host DNATs it inward.

## Setup

A small python server runs on a second container which has a POST route `/ingest`. This route accepts readings and returns `202 {"accepted"}` on success. Server secrets are stored in an .env file on the container. I hit an issue with permissions where the database would be readonly until `ProtectSystem=strict` was removed as well as setting `ReadWritePaths`. 

The ESP32 posts a batch to `/ingest` with a bearer token and a payload looks something like this:

```json
{
  "device": "esp32-01",
  "boot": "some_number_here",
  "rows": [
    { "seq": 1, "temp": 67, "humidity": 42.0, "co2": 666 }
  ]
}
```

The server writes to a `.db-wal` file first and then merges the new changes into the NAS. This file is a Write-Ahead Log file for SQLite databases that run in WAL mode. This is nice so that you can have processes read the database and have changes be put in at the same time. It's also like a git commit in that it's a bit of a "checkpoint" or "save". It's also supposedly a performance optimization.

## Dashboard?

The container also does serve a small dashboard on the network as a quick check to see
1. the data isn't completely garbage and is making sense
2. the ESP is still **UP** and taking data.

<div align="center">
  <img src="https://github.com/Implycitt/scifair/blob/main/Data/Images/dash.png?raw=true" alt="clarp clarp clarp" />
</div>

The dash runs on a CRON job to refresh every 5 minutes and serves it as a static page on `192.168.100.204:8090`.