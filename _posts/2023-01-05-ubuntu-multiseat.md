---
layout: post
---

You can easilly start with multiseat. Here is a nice blog
<https://www.apalrd.net/posts/2022/multiseat_intro/>
You can use this script to get all statuses
sudo loginctl seat-status seat3 > ~/config/loginctl/seat3.seat-status.txt                                                                                       
```
# ~/config/loginctl_seats/seat_statuses.sh
set -x # Show command being executed              

sudo loginctl seat-status seat0 > ~/config/loginstl_seats/seat0.seat-status.txt
sudo loginctl seat-status seat1 > ~/config/loginstl_seats/seat1.seat-status.txt
sudo loginctl seat-status seat2 > ~/config/loginstl_seats/seat2.seat-status.txt
sudo loginctl seat-status seat3 > ~/config/loginstl_seats/seat3.seat-status.txt                                                                                       
```
Before setting the seats you can read some info about command
```
man loginctl
loginctl list-seats
# when you want to remove all seats and attach to default
loginctl flush-devices
```

Let's store configuration in tmp folder which is tracked with git, for example
```
mkdir ~/config/loginctl_seats
cd ~/config/loginctl_seats
git init .
```

## Steps

Lets generate info
```
./seat_statuses.sh
git add .
git diff --staged
```
here is example seat status
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

You need to attach MASTER devices (ie graphic cards) otherwise new seat will not
be created.
Search for `drm` and look at last number (in this case `04`)
Also need to attach `render` and `fb` (frame buffer, if exists) and sound.
Framebuffer is created when you plug the monitor, so it is not neccessary.
Sound card could change the number so the best is to use * like
```
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.1/sound/*
```
Also find the ports that you want to assign. If you have usb dock station then
you do have to allocate only one port. All devices attached to docking station
will be under that port (also the sound and microphone). You can use headset
with microphone and with one 3.5mm cable you have stable sound connection.
Also hdmi sound is also stable since it is connected to your seat.
Put that in `attach_seats.sh` file so we can modify and run easilly

```
# attach_seats.sh
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/card1
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/renderD129
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/graphics/fb1

sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.1/sound/*

# those usb porst are two first rows on the back
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:1d.0/usb2
```

When you run attach you should not see any 04 devices on seat0
```
./attach_seats.sh

# restart
./seat_statuses.sh && git diff
git add .
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

or in `Activity Monitor` CPU tab search for `DisplayLinkUserAgent` and double
click > Quit > Force quit

You can attach specific evdi drm to new seats <https://askubuntu.com/a/1304701/40031>
```
```
# Error

On logout you could get scrambled screen on other seats. Solution is to click on
suspend, since when it wake up, it will provide normal login screen on all seats
