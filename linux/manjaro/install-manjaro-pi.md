# Installing Manjaro Linux on Raspberry Pi 3b

1. Make installer SD
    1. download image from https://manjaro.org/download/arm8-raspberry-pi-3-minimal/
    1. unzip
    1. flash to µSD card using Etcher
1. Install Manjaro
    1. insert µSD card
    1. boot up Pi
        1. if you see a lightning bolt in the top right of the screen, that mean your power supply isn't providing enough amps. You'll need to replace it with one that can provide 2.5 amps before continuing.
    1. follow on-screen prompts for initial setup
    1. the pi should reboot and you'll have a partially functional installation
1. Setup WiFi
    1. We'll be using `netctl` for this (you could probably also use `wpa_cli`)
    1. `$ sudo ip link set dev wlan0 down` (disables the WiFi interface for now)
    1. <code>$ sudo cp /etc/netctl/examples/wireless-wpa /etc/netctl/_**SSID HERE**_</code>
    1. <code>$ sudoedit /etc/netctl/_**SSID HERE**_</code>
            - This will use `vi` by default. If you're uncomfortable with `vi`, you can use `nano` instead by adding `EDITOR=nano ` to the front of the command.
        1. set `ESSID` to your network's name
        1. set `Key` to your network's password
    1. `$ sudo netctl enable orleans`
    1. `$ sudo netctl start orleans`
        1. At this point, you should have internet, but you likely won't have a working DNS setup
        1. Try pinging 8.8.8.8 to see if you have internet. **If you do**, continue. **If you don't**, you'll need to check your netctl config to make sure it's set up correctly and try again.
        1. Now try pinging google.com. **If this works**, awesome, skip the rest of these sub-steps and continue on to the next major step. **If it doesn't work**, continue to the next sub-step
        1. `$ drill google.com` Do you see an A record with an IP address listed for google.com? **If so**, continue. **If not**, start googling ¯\\\_(ツ)\_/¯
        1. `$ sudo systemctl disable systemd-resolved.service` (disables the systemd DNS resolver, which should clear up our DNS issues)
        1. `$ sudo reboot`
        1. Try pinging google.com again. **if this works**, awesome, continue on. **If this doesn't work**, start googling ¯\\\_(ツ)\_/¯
1. Update
    1. `$ sudo pacman -Syu` (updates all packages)
    1. `$ sudo reboot` (reboot to ensure we're running the potentially newly installed kernel)

At this point, you've got a fully functional Manjaro install. You can stop here if you want, but I'm going to continue and install BTRFS in place of the default EXT4 file system.

## Move install to BTRFS

1. resize EXT4 partition
1. install required tools `$ sudo pacman -Syu btrfs-progs uboot-tools`
1. create btrfs partition
    1. **insert instructions here**
1. mount partition
    1. `$ sudo mkdir -p /mnt/ZROOT`
    1. `$ sudo mount /dev/disk/by-label/ZROOT /mnt/ZROOT`
1. `$ btrfs subv create /mnt/ZROOT/\@root` (creates root volume)
1. `$ mkdir /mnt/ZROOT/snapshots` (makes folder to partition our snapshots from our root dir)
1. `$ sudo rsync -avxHW / /mnt/ZROOT/@root/`
1. update new fstab
    1. **Add more detailed instructions here**
    ```
    # <file system>      <dir>           <type> <options> <dump> <pass>
    LABEL=BOOT           /boot           vfat   defaults  0      0
    LABEL=ZROOT          /mnt/ZROOT      btrfs  defaults,ssd_spread,noatime,noauto 0 0
    LABEL=ZROOT          /               btrfs  defaults,ssd_spread,noatime,subvol=@root 0 0
    PARTUUID=2136610e-02 /mnt/ext4_root  ext4   defaults,noatime,noauto
    ```
1. update initramfs for BTRFS support
    1. `$ sudoedit /etc/mkinitcpio.conf`
        - add `btrfs` to the `MODULES` list
    1. `$ sudo mkinitcpio -p linux-aarch64`
1. edit `/boot/boot.txt`
    1. `$ sudo cp /boot/boot.txt /boot/boot.ext4.txt`
    1. `$ sudo cp /boot/boot.scr /boot/boot.ext4.scr`
    1. `$ sudoedit /boot/boot.txt` on the `setenv` line, change the following
        - change `root=PARTUUID=${uuid}` to <code>root=UUID=_**partition's UUID**_</code>
        - change `rootfstype=ext4` to `rootfstype=btrfs`
        - add `rootflags=subvol=@root`
        - if present, remove `fsck.repair=yes` since BTRFS will handle that
    1. `$ sudo ./mkscr`
1. `$ sudo reboot`

### Finishing touches (optional)

- Create an initial snapshot
    1. `$ sudo mount /dev/disk/by-label/ZROOT /mnt/ZROOT`
    1. `$ sudo btrfs subvolume snapshot / /mnt/ZROOT/snapshots/@root-$(date --iso-8601=minutes)`
- Set up Snapper
    1. `$ sudo pacman -Syu snapper snap-pac`
    1. `$ sudo snapper -c root create-config /`



Credit to https://hackmd.io/FP-7sHiPTJGaJvzSa3nw8A#Managing-snapshots for guiding me with a Raspbian BTRFS setup
