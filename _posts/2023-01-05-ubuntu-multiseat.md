---
layout: post
---

# Introduction

Here is a nice blog <https://www.apalrd.net/posts/2022/multiseat_intro/>
You can read some info about command being used
```
man loginctl
sudo loginctl list-seats
# when you want to remove all seats and attach to default seat0
sudo loginctl flush-devices
```

Let's store configuration in tmp folder which is tracked with git, for example
```
mkdir ~/config/loginctl_seats
cd ~/config/loginctl_seats
git init .
```

You can use this script to get all statuses
```
# ~/config/loginctl_seats/seat_statuses.sh
sudo loginctl seat-status seat0 > ~/config/loginstl_seats/seat0.seat-status.txt
sudo loginctl seat-status seat1 > ~/config/loginstl_seats/seat1.seat-status.txt
sudo loginctl seat-status seat2 > ~/config/loginstl_seats/seat2.seat-status.txt
sudo loginctl seat-status seat3 > ~/config/loginstl_seats/seat3.seat-status.txt
```

and call with
```
sudo pwd # just to check if sudo access is available
./seat_statuses.sh
```

List radeon cards (order does not follow position on PCIe slots)
```
lspci | grep VGA
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Cape Verde PRO [Radeon HD 7750/8740 / R7 250E]
04:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Cape Verde PRO [Radeon HD 7750/8740 / R7 250E]

radeontop -b 03
# Graphic pipe 10,00%
# this should remain in seat0 which will be default
radeontop -b 04
# Graphic pipe 0,00%
```

## Steps

Lets generate info
```
./seat_statuses.sh
git add .
git diff --staged
```
You need to attach MASTER devices (ie graphic cards) and frame buffers otherwise
new seat will not be created.
Search for `drm` and look at last number before (eg `0000:04:00:0`)
Also need to attach `render` and `fb` (frame buffer, it does not exists if
monitor is unplugged) and `sound` that is going throuhgh graphic card.
Also find the ports that you want to assign. If you have usb dock station that
is connected with usb3 port than you need to connect and find on which port it
is connected (it is different than old usb device is connected to the same
input port).
All devices attached to docking station will be under that port (also its sound
and microphone). You can use headset with microphone and with 3.5mm connector.
You can also use webcam for microphone, and monitor for sound speaker.
Put that in `attach_seats.sh` file so we can modify and run easilly
Note that you need to connect all devices and run `./attach_seats.sh` since if
device is not connected it will not attach to seat when script is run.

```
# attach_seats.sh
# run with: sudo ./attach_seats.sh
#
# seat1
# PCI on the bottom
loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/card1
# note that on reboot card1 could become card2 so we are attaching both, so
# ignore warning that device does not exists. When number changes on reboot it
# will attach to proper seat anyway, but I got blank login screen
loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/card1
loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/renderD129
loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/graphics/fb1

loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.1/sound/card2

# those usb porst are two first rows on the back
# note that usb2 could become usb1 on reboot, but still, it will attach 1d
# device to seat1, so you can just update git with ./seat_statuses.sh
loginctl attach seat1 /sys/devices/pci0000:00/0000:00:1d.0/usb2
```

When you run attach you should not see any 04 devices on seat0
```
sudo ./attach_seats.sh

# restart
sudo reboot

./seat_statuses.sh && git diff
git add . # if everything OK, otherwise git checkout
```

## Issues

When someone logs out than all users will log out and only seat0 will get
login screen. Sometimes it helps that seat0 log in and log out. The best is to use Lock instead of Log out.

When you remove graphic card, than that seat will be discarded and all it's
devices will be assigned to default seat seat0.

To remove one specific device from seat, you need to attach to another seat (you
need to know it's path)

# USB Dock

Download Display Link software from <https://www.synaptics.com/products/displaylink-graphics/downloads/ubuntu>
and follow <https://support.displaylink.com/knowledgebase/articles/1944022-how-to-install-displaylink-software-on-ubuntu-20-0>

Uninstall with
https://support.displaylink.com/knowledgebase/articles/683699-how-to-uninstall-displaylink-ubuntu-software
```
sudo displaylink-installer uninstall
```
Install with

```
sudo ./displaylink-driver-5.3.0.xx.run
# this will install libdrm-dev lipciaccess-dev

Installing
[ Installing EVDI ]
[[ Installing EVDI DKMS module ]]
Creating symlink /var/lib/dkms/evdi/1.12.0/source -> /usr/src/evdi-1.12.0

Kernel preparation unnecessary for this kernel. Skipping...

Building module:
cleaning build area...
make -j24 KERNELRELEASE=5.19.0-35-generic all INCLUDEDIR=/lib/modules/5.19.0-35-generic/build/include KVERSION=5.19.0-35-generic DKMS_BUILD=1...
Signing module:
 - /var/lib/dkms/evdi/1.12.0/5.19.0-35-generic/x86_64/module/evdi.ko
EFI variables are not supported on this system
/sys/firmware/efi/efivars not found, aborting.
cleaning build area...

evdi.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/5.19.0-35-generic/updates/dkms/

depmod....
[[ Installing module configuration files ]]
[[ Installing EVDI library ]]
make: Entering directory '/tmp/tmp.XS3WN4Qt0Q/evdi/library'
cc -I../module -std=gnu99 -fPIC -D_FILE_OFFSET_BITS=64    -c -o evdi_lib.o evdi_lib.c
cc evdi_lib.o -shared -Wl,-soname,libevdi.so.0 -o libevdi.so.1.12.0 -lc -lgcc 
cp libevdi.so.1.12.0 libevdi.so
make: Leaving directory '/tmp/tmp.XS3WN4Qt0Q/evdi/library'
[ Installing x64-ubuntu-1604/DisplayLinkManager ]
[ Installing libraries ]
[ Installing firmware packages ]
[ Installing licence file ]
[ Adding udev rule for DisplayLink DL-3xxx/4xxx/5xxx/6xxx devices ]
[ Adding upstart and powermanager sctripts ]

Please read the FAQ
http://support.displaylink.com/knowledgebase/topics/103927-troubleshooting-ubuntu

Installation complete!

Please reboot your computer if you're intending to use Xorg.
```

Read FAQ
https://support.displaylink.com/knowledgebase/topics/103927-troubleshooting-ubuntu

When you move the between seats only difference is that it moves sound:card and
usb (all attached usb devices will show up under this usb device)
```
		  │   │ ├─/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.2/1-1.2.2
		  │   │ │ usb:1-1.2.2
		  │   │ │ ├─/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.2/1-1.2.2/1-1.2.2.1/1-1.2.2.1:1.2/sound/card6
		  │   │ │ │ sound:card6 "Dock"
		  │   │ │ └─/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.2/1-1.2.2/1-1.2.2.4
		  │   │ │   usb:1-1.2.2.4
```
other devices stays the same.
And there is no change when we start or stop the driver.

Driver page is <https://github.com/DisplayLink/evdi>

You can stop start the driver with
```
sudo service displaylink-driver stop
```

You can attach specific evdi drm to new seats <https://askubuntu.com/a/1304701/40031>
```
```
