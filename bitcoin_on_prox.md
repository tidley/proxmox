
# Bitcoin on Debian 12 (Bookworm) Container in Proxmox

Heavily influenced by https://github.com/raspibolt/raspibolt

## Preparation
- Find data drive (assuming it's external) `lsblk`
        - Mount drive using UUID found via `blkid /dev/sdc1`
        - `nano /etc/fstab`
        > new ssd /dev/sdc1: UUID="428d6f9f-e796-4c6d-95c1-06408d764e24" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="74f517f8-01"
        - Append: `UUID=428d6f9f-e796-4c6d-95c1-06408d764e24 /mnt/bitcoin ext4 defaults,nofail,x-systemd.device-timeout=1 0 2`
### Had to make a systemd process
  GNU nano 7.2                                             /etc/systemd/system/mnt-bitcoin.mount                                                       
[Unit]
Description=Mount /mnt/bitcoin

[Mount]
What=UUID=428d6f9f-e796-4c6d-95c1-06408d764e24
Where=/mnt/bitcoin
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target

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

bitcoin@btc:~/.bitcoin$ python3 rpcauth.py raspibolt QneXLhhKxeamlHRw1ZjHPH1JNARI1UaELCTP
String to be appended to bitcoin.conf:
rpcauth=raspibolt:74abeb5b4eba364f01a637e56c975991$52ffe28b3a4b3b8209a7ce42675f9b21302bdb7c1701dc382520b2ee35c477f4
Your password:
QneXLhhKxeamlHRw1ZjHPH1JNARI1UaELCTP



### Install
https://raspibolt.org/guide/bitcoin/bitcoin-client.html#installation-1

- Change to downloaded file `tar -xvf bitcoin-27.1-x86_64-linux-gnu.tar.gz`
- Follow other steps

## Install Electrum

As described [here](https://raspibolt.org/guide/bitcoin/electrum-server.html) except additional permissions were required.

As admin:
```
sudo chmod 750 /home/bitcoin/.bitcoin
sudo chmod 750 /home/bitcoin
```

## Install LND

Download path:

`wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/lnd-linux-amd64-v$VERSION-beta.tar.gz`

Install:

```
tar -xvf lnd-linux-amd64-v$VERSION-beta.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-amd64-v$VERSION-beta/*
```

To do: [using a password manager with named pipe](https://github.com/lightningnetwork/lnd/blob/master/docs/wallet.md#more-secure-example-with-password-manager-and-using-a-named-pipe)

## Install LND Connect

Download path:
`
wget https://github.com/LN-Zap/lndconnect/releases/download/v0.2.0/lndconnect-linux-amd64-v0.2.0.tar.gz
`

Unpack `tar -xvf lndconnect-linux-amd64-v0.2.0.tar.gz`

Install `sudo install -m 0755 -o root -g root -t /usr/local/bin lndconnect-linux-amd64-v0.2.0`


## Cloning dodgy ssd

dd if=/dev/sde of=/dev/sdc bs=64K conv=noerror,sync status=progress