# Raspberry Pi NAS (Samba)

Instructions for setting up a Raspberry Pi NAS with an external drive.

## Overview

The objective was to setup a NAS, accessible anywhere, using an external drive. 

The storage is setup using Samba. 

## Hardware
- Raspberry Pi 4 B
- Raspberry Pi 4 Case
- Raspberry Pi Power Supply
- SD Card (16 GB minimum) for the Pi OS
- External drives

## Instructions

First update and upgrade your OS to the latest version.

```
sudo apt-get update
sudo apt-get upgrade
```

Then, install Samba packages.
```
sudo apt-get install samba samba-common-bin
```
You will be asked whether the settings should be used by DHCP. Answer yes.

Next we will create a new folder that points to the contents of the storage device and give your user (pi for this example) ownership rights to it. The older will be located in the `media/shared` directory.

```
sudo mkdir /media/shared
sudo chown -R pi:pi /media/shared/*
```

Now we will mount each drive. To do so, first identify the drive's path with `sudo fdisk -l`. It should appear something like `/dev/sda1`. 


Next, mount the drive with `sudo mount /dev/sda1 /media/shared -o uid=pi,gid=pi`. Make sure to replace the disk path and optionally the shared folder path if you have changed it. 

Finally, we will want to mount the drive automatically. To do this, edit the fstab file:
```
sudo nano /etc/fstab
```

Add a line that should look like this (make sure to use the correct shared folder name):
```
/dev/sda1 /mnt/shared auto noatime umask=0 0 0
```

We will now setup the samba configuration. 
First, replace the old configuration with a new one (keeping the old one for reference).
```
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_backup
sudo nano /etc/samba/smb.conf
```

Add the following at the bottom of the file: 
```
[shared]
Comment = Raspberry Pi Shared Folder
Path = /media/shared
Browseable = yes
read only = no
create mask = 0777
directory mask = 0777
Guest ok = no
```

Guest access is disabled here. 

Next we will create a samba user. It has to be a unix user, here I've used `pi` as an example.

```
sudo smbpasswd -a pi
```
You will prompted to give the user a password.

Finally, restart samba: 
```
sudo service smbd restart
```

You can follow guides online to connect to the server. 

### Common errors

You may encounter the following error when mounting a drive: 
```
Mount is denied because the NTFS volume is already exclusively opened.
The volume may be already mounted, or another software may use it which
could be identified for example by the help of the 'fuser' command.
```
To fix this issue, run `lsblk` to list mountpoints. For the relevant mountpoint, run `sudo umount <mountpoint>`, the run the mount command again. 

### Remote access with Tailscale (optional)

Without a service like Tailscale, you cannot access the NAS outside your local network. To fix this, we use a service like Tailscale which makes the setup easy and secure. 

First, go to the [tailscale website](https://tailscale.com/) and create an account. 
<br/>
The go to the machines dashboard and add a linux server using the script provided. 
<br/>
Run `sudo tailscale up`.
<br/>
Although this is not recommended, you can _disable key expiry_ to prevent remote access loss when the key expires (this avoids you having the connect to the Rpi when the key expires.). It is however, a lot less secure and depending on the sensitivity of your data, it is not recommended. 

You will then be able to access the samba server via tailscale. 

## Future improvements
- RAID setup
- Fuse multiple drives into one storage point

### Useful guides
[Raspberry Pi Samba Server: Share Files in the Local Network](https://tutorials-raspberrypi.com/raspberry-pi-samba-server-share-files-in-the-local-network/)