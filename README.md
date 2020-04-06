# ansible-raspi

The setup of my Raspberry Pi used for home-automation:

* Raspbian Lite (headless, no UI)
* docker
* piVCCU3 (Homematic IP)
* ioBroker
* MQTT for e.g. Tasmota plugs
* InfluxDB for logging from ioBroker
* Grafana for visualizations


## Prepare the SD card

### Install Raspbian Lite

1. Download the image

   ```sh
   curl --location https://downloads.raspberrypi.org/raspbian_lite_latest --output raspbian.zip
   unzip raspbian.zip
   rm raspbian.zip
   ```

1. Prepare SD card for flashing

   Insert SD card and check its device name and potentially mounted partitions
   with `lsblk -p`. Unmount partitions that got mounted automatically by the OS.

    Linux:

    ```sh
    # Find device assigned to the SD card, e.g. /dev/sdc.
    lsblk -p
    # Unmount.
    umount /dev/<from above>
    ```

    macOS:

    ```sh
    # Look for something like /dev/disk2 (external, physical).
    diskutil list
    # Find mount point.
    mount | grep /dev/<from above>
    # Unmount.
    diskutil unmount /Volumes/<path>
    ```

1. Flash the SD card

   ```sh
   sudo dd if=*-raspbian-buster-lite.img of=/dev/<from above> bs=4M
   ```

1. Enable SSH

   SSH is the major way to connect to the Pi, but it is not enabled by default.

   ```sh
   # The mount point may be /media/$USER or /Volumes/boot depending on your OS.
   touch <boot volume mount point>/ssh
   ```

## Optional: Boot from SSD

Connect your external drive or SSD to your computer and perform the same steps
as above. You can omit creating the `ssh` file because the disk's boot partition
will not be used.

## First boot

Unmount all drives.

```sh
umount /dev/<SD card>
umount /dev/<external disk> # Optional.

diskutil unmount /Volumes/<SD card>
diskutil unmount /Volumes/<external disk> # Optional.
```

Remove the SD card, plug it into the Raspberry Pi. Do **not** connect the
external drive if you want to boot from it (see below). Boot the Pi with
connected ethernet. Check its reachability with ping.

```sh
ping raspberrypi

ssh pi@raspberrypi
```

You may run `raspi-config` to e.g. set up a Wi-Fi connection or a different
hostname.

## Enable Ansible

```sh
$ ssh-copy-id pi@raspberrypi

$ ansible raspberries -a '/bin/echo It works' --become
raspberrypi | CHANGED | rc=0 >>
It works
```

## Optional: Boot from SSD

In order to boot the Raspberry Pi from an external disk we need to do some
preparations that are packaged as an ansible role. The role is based on [this
guide](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/).

1. Connect the external drive
1. Run the following command:

   ```sh
   ansible-playbook -t external-boot site.yml
   ```
