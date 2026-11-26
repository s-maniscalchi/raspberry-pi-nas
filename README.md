# Raspberry Pi NAS

I built my own NAS using a Raspberry Pi and a USB drive.

The idea was to have my own local storage that I can access from my other computers without using cloud services.

## Hardware

- Raspberry Pi
- Kingston DataTraveler 70 (128 GB) (had it laying around at home)
- USB storage

## Software

- Raspberry Pi OS
- Linux ext4 filesystem
- Samba for file sharing
- SSH for remote access

## How it works

The USB drive is connected to the Raspberry Pi and mounted at `/mnt/nas`.

Samba shares this folder over the local network, allowing other computers to access the files.

```text
          Raspberry Pi
                |
             Samba
                |
             /mnt/nas
                |
           USB drive
                |
        Local network
           /      \
        Ubuntu   macOS
```

## Setup

### 1. Mount the USB drive

Find the USB drive and its UUID:

```bash
lsblk -f
```

Create the mount directory:

```bash
sudo mkdir -p /mnt/nas
```

Add the drive to `/etc/fstab`:

```text
UUID=YOUR-UUID /mnt/nas ext4 defaults,nofail 0 2
```

Mount the drive:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

### 2. Set permissions

```bash
sudo chown USERNAME:USERNAME /mnt/nas
sudo chmod 775 /mnt/nas
```

### 3. Install Samba

```bash
sudo apt update
sudo apt install samba samba-common-bin
```

Add a Samba user:

```bash
sudo smbpasswd -a USERNAME
```

### 4. Configure Samba

Edit the Samba configuration:

```bash
sudo nano /etc/samba/smb.conf
```

Add:

```ini
[NAS]
    path = /mnt/nas
    browseable = yes
    read only = no
    valid users = USERNAME
    create mask = 0664
    directory mask = 0775
```

Check the configuration and start Samba:

```bash
testparm
sudo systemctl enable --now smbd
sudo systemctl restart smbd
```

### 5. Connect to the NAS

Find the Raspberry Pi's IP address:

```bash
hostname -I
```

On another computer, connect using:

```text
smb://RASPBERRY_PI_IP/NAS
```

Log in with the Samba username and password.

## Useful commands

Check storage:

```bash
df -h /mnt/nas
```

Check Samba:

```bash
sudo systemctl status smbd
```

Check connected drives:

```bash
lsblk -f
```

## Future ideas

- Add automatic backups
- Set up access not only from my network but from everywhere 
