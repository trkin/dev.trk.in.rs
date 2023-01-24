---
layout: post
---

We started with multiseat
https://www.apalrd.net/posts/2022/multiseat_intro/

```
man loginctl
loginctl list-seats
loginctl seat-status seat0 > seat_status_initial.txt

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
Also need to attach `render` and `fb` (frame buffer)

```
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/card1
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/drm/renderD129
sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.0/graphics/fb1

sudo loginctl attach seat1 /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/0000:02:10.0/0000:04:00.1/sound/card2

loginctl attach seat1 /sys/devices/pci0000:00/0000:00:1d.0/usb2

loginctl seat-status seat1
# this should show all 04 devices

loginctl seat-status seat0
# this should not show any 04 devices
```

For usb you should attach arduino and find Bus for each port
```
lsusb | grep Arduino
Bus 003 Device 008: ID 2341:0043 Arduino SA Uno R3 (CDC ACM)

loginctl seat-status seat0 | grep ttyACM
		  │ │ └─/sys/devices/pci0000:00/0000:00:14.0/usb3/3-1/3-1:1.0/tty/ttyACM0
```
for example on my comp front bottom is usb1, front top is usb2, back bottom is
usb3, back middle and top are usb1

When someone logs out than all users will log out and only seat0 will get
login screen. Solution is to log in and log out as seat0

When you remove graphic card, than that seat will be discarted and all it's
devices will be assigned to default seat seat0.
When you want to remove all seats and attach to default
```
loginctl flush-devices
```

To remove one specific device from seat, you need to attach to another seat (you
need to know it's path)


# USB Dock

Download Display Link software from https://www.synaptics.com/products/displaylink-graphics/downloads/ubuntu
and follow https://support.displaylink.com/knowledgebase/articles/1944022-how-to-install-displaylink-software-on-ubuntu-20-0

```
sudo ./displaylink-driver-5.3.0.xx.run
```

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

Driver page is https://github.com/DisplayLink/evdi

You can stop start the driver with
```
sudo service displaylink-driver stop
```

You can attach specific evdi drm to new seats https://askubuntu.com/a/1304701/40031
```
```
