# Safely mount a drive

Edit (via nano)

    /etc/fstab

and add

    /dev/sdc /mnt/bitcoin ext4 defaults,nofail,x-systemd.device-timeout=1 0 2

Where
- `/dev/sdc` = physical location of drive as shown by lsblk
- `/mnt/bitcoin` = mount point on VM
- `ext4` = format of disk
- `defaults` keeps standard mount options
- `nofail` ensures the system will boot even if the drive is not present
- `x-systemd.device-timeout=1` limits the time systemd waits for the device to 1 second