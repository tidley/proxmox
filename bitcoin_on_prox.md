
# Bitcoin on Debian 12 (Bookworm) Container in Proxmox

Heavily influenced by https://github.com/raspibolt/raspibolt

## Preparation
- Find data drive (assuming it's external) `lsblk`
- Mount drive using UUID found via `blkid /dev/sdc1`
- `nano /etc/fstab`
- Append: `UUID=610c1a1b-2683-419e-8ce5-eb824f7d53f7 /mnt/bitcoin ext4 defaults,nofail,x-systemd.device-timeout=1 0 2`
- Give permissions `chown -R 100000:100000 /mnt/bitcoin`
- Install debian container

## Container Config

- Update `sudo apt get update`
- Upgrade `sudo apt get upgrade`
- Install sudo `sudo apt install sudo`
- Install gpg `sudo apt install gpg`
- Install curl `sudo apt install curl`
- Create 5 passwords (A - E) a la https://raspibolt.org/guide/raspberry-pi/preparations.html#write-down-your-passwords

        [ A ] Master user/admin password
        [ B ] Bitcoin RPC password
        [ C ] LND wallet password
        [ D ] BTC-RPC-Explorer password (optional)
        [ E ] Ride The Lightning password

## User accounts

- Login using root privileges
- Create user 'admin' using password A `adduser admin`
- Add to sudo `usermod -aG sudo admin`

## Install git
- Login to `admin`
- Install git `sudo apt install git`

## Data directory
- Mount to `/data/bitcoin`
- Give ownership to admin `sudo chown admin:admin /data`

## Network Security
From admin:
- Configure ufw

        sudo apt install ufw
        sudo ufw default deny incoming
        sudo ufw default allow outgoing
        sudo ufw allow ssh
        sudo ufw logging off
        sudo ufw enable
        sudo systemctl enable ufw

- Install fail2ban `sudo apt install fail2ban`

TODO https://raspibolt.org/guide/raspberry-pi/security.html#prepare-nginx-reverse-proxy

Install tor: https://raspibolt.org/guide/raspberry-pi/privacy.html#installation using these for x86 systems:

    deb     [arch=arm64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org bookworm main
    deb-src [arch=arm64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org bookworm main


TODO https://raspibolt.org/guide/raspberry-pi/privacy.html#ssh-remote-access-through-tor-optional

## Install Bitcoin Core

https://raspibolt.org/guide/bitcoin/bitcoin-client.html#installation

### Download

https://raspibolt.org/guide/bitcoin/bitcoin-client.html#preparations

As admin
- `cd /tmp`
- `VERSION="27.1"`
- `wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/bitcoin-$VERSION-x86_64-linux-gnu.tar.gz`
- Follow other steps

### Install
https://raspibolt.org/guide/bitcoin/bitcoin-client.html#installation-1

- Change to downloaded file `tar -xvf bitcoin-27.1-x86_64-linux-gnu.tar.gz`
- Follow other steps