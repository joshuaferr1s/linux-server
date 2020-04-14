# Ubuntu Server Setup

## To get up and running for a openssh server

### Allocating the server a static IP

* IPv4
  * Subnet: 192.168.1.0/24
  * Address: 192.168.1.X
  * Gateway: 192.168.1.1
  * Name Servers: 8.8.8.8
  * Search Domains:

### Update Packages and Distribution

```bash
sudo apt upgrade
sudo apt dist-upgrade
sudo reboot # If necessary
```

### Create new user with sudo permissions

```bash
sudo adduser -G sudo <user>
```

### Open SSH Server

```bash
sudo apt install openssh-server
```

* Copy sshd_config from this repo to /etc/ssh/sshd_config
* `sudo systemctl restart ssh`

### Firewall

```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow 22/tcp #opens ssh port
```

## Nicer Experience

### zsh

```sh
sudo apt install zsh
sudo nano ~/.zshrc
sudo chsh --shell=/bin/zsh $USER
```

```sh
# ~/.zshrc
setopt AUTO_CD
setopt PROMPT_SUBST
autoload -Uz vcs_info
precmd() { vcs_info }
zstyle ':vcs_info:git:*' formats '%F{011}[%b]%f'

EMOJIS=(👾 🤖 🐝 🌵 🙈 🦑)
EMOJI=$EMOJIS[$RANDOM%$#EMOJIS+1]

PROMPT='
$EMOJI $USER@$HOST | %F{031}%~%f ${vcs_info_msg_0_}
> '
```

## Setup Raid for drives and Samba Share

### Raid

```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT # List drives
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdX /dev/sdY # Creates a raid1 of 2 drives X and Y called md0
sudo cat /proc/mdstat # Should show md0 as active raid1
sudo mkfs.ext4 -F /dev/md0 # Set file system for raid1 drive
sudo mkdir -p /mnt/md0 # Create /mnt/md0
sudo mount /dev/md0 /mnt/md0 # Mounts the raid drive
df -h -x devtmpfs -x tmpfs # Should show all mounted drives
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf # Auto mount drives
update-initramfs # Auto mount drives
echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
sudo reboot
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT # Drive should show as mounted
cd /mnt/md0
sudo mkdir <foldername> # Will be the folder we share
```

### Samba
```bash
sudo apt update
sudo apt install samba
sudo nano /etc/samba/smb.conf # Copy contents of smb.conf to the bottom of existing configuration
sudo smbpasswd -a <user> # For each allowed user
sudo service smbd restart
```

## Update motd and Banner

### Banner
```bash
sudo nano /etc/ssh/ssh-banner # Paste banner
```

### motd
```bash
sudo rm /etc/update-motd.d/<file> # all files in directory
sudo nano /etc/update-motd.d/<file> # for each in update-motd.d in this repo
sudo chmod +x /etc/update-motd.d/<file> # for each new file
```
