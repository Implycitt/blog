# NAS

A couple months back I got my hands on the old family computer. It isn't very powerful:
```sh
         .://:`              `://:.             root@proxmox
       `hMMMMMMd/          /dMMMMMMh`           ------------
        `sMMMMMMMd:      :mMMMMMMMs`            OS: Proxmox VE 9.2.21 x86_64
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`    Host: p6516f
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`    Kernel: Linux 7.0.14-20-pve
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`      Packages: 788 (dpkg)
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.        Shell: bash 5.2.37
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.          Terminal: termproxy
        -+ooooooo/.`sMMs`./ooooooo+-            CPU: AMD Athlon(tm) II X4 630 (4) @ 2.80 GHz
          :oooooooo/`..`/oooooooo:              GPU: AMD Radeon HD 4200
          :oooooooo/`..`/oooooooo:              Memory: 1.75 GiB / 5.54 GiB (32%)
        -+ooooooo/.`sMMs`./ooooooo+-            Swap: 1000.00 KiB / 5.54 GiB (0%)
      .+ooooooo+-`oNMMMMNo`-+ooooooo+.          Disk (/): 5.59 GiB / 93.93 GiB (6%) - ext4
    ./ooooooo+- +NMMMMMMMMN+ -+ooooooo/.        Disk (/srv/nas): 178.81 MiB / 294.23 GiB (0%) 
  `/oooooooo:`:mMMMMMMMMMMMMm:`:oooooooo/`      Locale: en_US.UTF-8
`:oooooooo/`-hMMMMMMMyyMMMMMMMh-`/oooooooo:`    
`-/+oo+/:`.yMMMMMMMh-  -hMMMMMMMy.`:/+oo+/-`    
        `sMMMMMMMm:      :dMMMMMMMs`
       `hMMMMMMd/          /dMMMMMMh`                                   
         `://:`              `://:`                                     
```

I have more hardware that I will be able to eventually cluster together another day, but I wanted to get started working on hosting my own services locally.

## Setup Adventures

My homelab uses the Proxmox Virtual Environment (VE) for the Virtual Machines (VM) and containers. The setup was quite simple. I flashed the Proxmox ISO onto a flashdrive and then followed the installation process like you would with any standard linux distro. I chose Proxmox since it integrates a hypervisor and the linux containers to make it really easy to manage VMs and containers.

I also did run into some strange issue with the BIOS for the computer. The BIOS was set to a strange mode that would not properly read newer ISOs.

### Wifi Troubles

The main issue I ran into was internet connection issues. I don't have direct access to ethernet in the room where I have my homelab. The reason that this is a problem is that Proxmox does not come with wireless network drivers by default. 

This meant that I first had to buy and install a Wi-Fi card. Then, to solve the ethernet problem, I used my phone to network tether the homelab via USB. With the tether I was able to install the wireless network drivers so that proxmox could use the Wi-Fi card.

Briefly, two config changes were needed to get Proxmox to fully work with Wi-Fi:
* `/etc/network/interfaces/` interface update
* Making vmbr0 a NAT bridge instead of a switch

## Actual Setup

Now that Proxmox was ready and worked, I could actually set up a NAS. To accomplish this I created a container that would run Samba, "an open source implementation of the SMB/CIFS protocol" ([Samba](https://www.samba.org/)), which lets you share files over the local network. A user was created inside the container to have Samba actually work due to file permissions and sharing.

Once samba was set up, any device on the network is able to see the files on `\\nas\files`; nas is the hostname of the container and files is set by the section header in the `smb.conf` file.

## Fin

I've been meaning to set up the NAS for a while so that I could transfer files across devices NOT with email. It is also useful for my [sister's project](https://github.com/Implycitt/scifair).

The next main project will be working on a tunneling vpn so that I can access the server or NAS outside of my home network.