+++
title = 'Adding Bluetooth Support for Archer TX55E AX3000'
date = 2024-12-27T11:40:24-06:00
+++
## A living room PC with no bluetooth
Recently I built a living room PC for usage as a media station
(movies, shows, games, etc). After installing Fedora 41 and doing
some basic configuration with a keyboard and mouse plugged in
I decided to test out my Xbox controller and new bluetooth keyboard:

**< TODO > picture of the failure would be nice here.**

rats, that's not good -- I've got no good interface to the
machine without the Bluetooth up and running. I poked at
it for a few more minutes before deciding (or being told :P)
that I had worked on this for too long. I snagged the little
TP-Link UB500 I had in the basement (which luckily worked out
of the box) and took things for a test run with Baldurs Gate 3.

Everything worked alright, but as if to twist the knife further
bluetooth would sometimes disconnect/reconnect on the Xbox controller
I had. In the past I've had wireless issues from that corner of the living
room, so I was thinking the whole time that the Archer TX55E AX3000 that's
installed but not working (at least for Bluetooth) would probably perform
better...

## Troubleshooting
After a night full of urges to "check just one more thing" I awoke
sleepy eyed and full of hatred for the thing. Iced coffee in hand
I started to probe.

### Getting a decent mental model of the situation
Doubling checking the connections, etc, everything seemed
alright which made me step back and picture the system at
play here.

From a hardware point of view, I've got a PCIe device plugged
in (for the wifi parts of the Archer TX55E AX3000) and a
USB header attached to the device as well. The wifi works great,
and the Bluetooth is supposed to be provided over the USB connection.

![Archer TX55E / AX3000 Box](/archer-tx55e-box.jpg)

{{< mermaid >}}
graph TD
    B[Motherboard]
    C[CPU]
    D[PCIe Slot]
    E[F_USB Header]

    D <--> B
    E <--> B
    B <--> C

    subgraph Archer TX55E AX3000
        F[Wi-Fi Module]
        G[Bluetooth Module]
    end

    F <--> D
    G <--> E
{{< /mermaid >}}

*A crummy block diagram showing the components
at play*

With that in mind, where are things going wrong?

### Checking USB and PCIe
To get a good look at the two buses (USB and PCIe) I checked out
lsusb and lspci.

#### USB
lsusb looked ok, I had to guess (and then later verified by hotplugging
the TX55E) which device was associated with the TX55E, but that was
fairly obvious due to the name being the only logical one:
```
[ajhalaney@livingroom ~]$ sudo lsusb -v -d 13d3:3610

Bus 001 Device 004: ID 13d3:3610 IMC Networks Wireless_Device
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.10
  bDeviceClass          239 Miscellaneous Device
  bDeviceSubClass         2 [unknown]
  bDeviceProtocol         1 Interface Association
  bMaxPacketSize0        64
  idVendor           0x13d3 IMC Networks
  idProduct          0x3610 Wireless_Device
  bcdDevice            1.00
  iManufacturer           5 MediaTek Inc.
  iProduct                6 Wireless_Device
  iSerial                 7 000000000
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x00fe
    bNumInterfaces          3
    bConfigurationValue     1
    iConfiguration          8 Config_01
    bmAttributes         0xe0
      Self Powered
      Remote Wakeup
    MaxPower              100mA
    Interface Association:
      bLength                 8
      bDescriptorType        11
      bFirstInterface         0
      bInterfaceCount         3
      bFunctionClass        224 Wireless
      bFunctionSubClass       1 Radio Frequency
      bFunctionProtocol       1 Bluetooth
      iFunction               4 BT_FUNCTION
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              1 BT_ACL_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0010  1x 16 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0000  1x 0 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0000  1x 0 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       1
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0009  1x 9 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0009  1x 9 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       2
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0011  1x 17 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0011  1x 17 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       3
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0019  1x 25 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0019  1x 25 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       4
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0021  1x 33 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0021  1x 33 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       5
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0031  1x 49 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0031  1x 49 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       6
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              2 BT_SCO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x003f  1x 63 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x03  EP 3 OUT
        bmAttributes            1
          Transfer Type            Isochronous
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x003f  1x 63 bytes
        bInterval               4
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        2
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              3 BT_ISO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x8a  EP 10 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x0a  EP 10 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               1
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        2
      bAlternateSetting       1
      bNumEndpoints           2
      bInterfaceClass       224 Wireless
      bInterfaceSubClass      1 Radio Frequency
      bInterfaceProtocol      1 Bluetooth
      iInterface              3 BT_ISO_If
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x8a  EP 10 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x0a  EP 10 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               1
Binary Object Store Descriptor:
  bLength                 5
  bDescriptorType        15
  wTotalLength       0x000c
  bNumDeviceCaps          1
  USB 2.0 Extension Device Capability:
    bLength                 7
    bDescriptorType        16
    bDevCapabilityType      2
    bmAttributes   0x00000000
      (Missing must-be-set LPM bit!)
Device Status:     0x0003
  Self Powered
  Remote Wakeup Enabled
```
Things seem fine here, and you can see its a MediaTek provided
device under the hood and has bluetooth interfaces.

I also checked dmesg with nothing interesting showing up:
```
$ sudo dmesg | grep -i usb | grep 1-11
[    2.403556] usb 1-11: new high-speed USB device number 4 using xhci_hcd
[    2.528621] usb 1-11: New USB device found, idVendor=13d3, idProduct=3610, bcdDevice= 1.00
[    2.528624] usb 1-11: New USB device strings: Mfr=5, Product=6, SerialNumber=7
[    2.528625] usb 1-11: Product: Wireless_Device
[    2.528626] usb 1-11: Manufacturer: MediaTek Inc.
[    2.528626] usb 1-11: SerialNumber: 000000000
```

#### PCIe
Wifi was working fine, (and the device manual is clear that the USB
is needed for Bluetooth) but I decided to also see what the TX55E
showed up as and make sure there were no errors on that front before
moving up the stack.

lspci looked normal, and so did the kernel's dmesg output:
```
$ lspci | grep -i mediatek
05:00.0 Network controller: MEDIATEK Corp. MT7922 802.11ax PCI Express Wireless Network Adapter
$ dmesg | grep "05:00.0\|wlp5s0\|wlan0"
[    0.285304] pci 0000:05:00.0: [14c3:7922] type 00 class 0x028000 PCIe Endpoint
[    0.285330] pci 0000:05:00.0: BAR 0 [mem 0x4410100000-0x44101fffff 64bit pref]
[    0.285346] pci 0000:05:00.0: BAR 2 [mem 0x70c00000-0x70c07fff 64bit]
[    0.285469] pci 0000:05:00.0: PME# supported from D0 D3hot D3cold
[    7.238213] mt7921e 0000:05:00.0: enabling device (0000 -> 0002)
[    7.251561] mt7921e 0000:05:00.0: ASIC revision: 79220010
[    7.327221] mt7921e 0000:05:00.0: HW/SW Version: 0x8a108a10, Build Time: 20241106163228a
[    7.710842] mt7921e 0000:05:00.0: WM Firmware Version: ____000000, Build Time: 20241106163310
[    8.900553] mt7921e 0000:05:00.0 wlp5s0: renamed from wlan0
[   12.716017] wlp5s0: authenticate with 40:ed:00:12:11:bd (local address=fa:6e:5e:d9:9f:c4)
[   13.190474] wlp5s0: send auth to 40:ed:00:12:11:bd (try 1/3)
[   13.269143] wlp5s0: authenticate with 40:ed:00:12:11:bd (local address=fa:6e:5e:d9:9f:c4)
[   13.280342] wlp5s0: send auth to 40:ed:00:12:11:bd (try 1/3)
[   13.302046] wlp5s0: authenticated
[   13.302444] wlp5s0: associate with 40:ed:00:12:11:bd (try 1/3)
[   13.325829] wlp5s0: RX AssocResp from 40:ed:00:12:11:bd (capab=0x511 status=0 aid=5)
[   13.358518] wlp5s0: associated
[   13.525040] wlp5s0: Limiting TX power to 27 (30 - 3) dBm as advertised by 40:ed:00:12:11:bd
[   24.750496] wlp5s0: disconnect from AP 40:ed:00:12:11:bd for new auth to 40:ed:00:12:09:b1
[   24.982663] wlp5s0: authenticate with 40:ed:00:12:09:b1 (local address=fa:6e:5e:d9:9f:c4)
[   25.135609] wlp5s0: send auth to 40:ed:00:12:09:b1 (try 1/3)
[   25.217559] wlp5s0: authenticate with 40:ed:00:12:09:b1 (local address=fa:6e:5e:d9:9f:c4)
[   25.251773] wlp5s0: send auth to 40:ed:00:12:09:b1 (try 1/3)
[   25.256963] wlp5s0: authenticated
[   25.258980] wlp5s0: associate with 40:ed:00:12:09:b1 (try 1/3)
[   25.279954] wlp5s0: RX ReassocResp from 40:ed:00:12:09:b1 (capab=0x511 status=0 aid=2)
[   25.313960] wlp5s0: associated
[   25.333190] wlp5s0: Limiting TX power to 27 (30 - 3) dBm as advertised by 40:ed:00:12:09:b1
```

With all that in mind, physically things seem ok, so I need
to move up the stack.

### Checking hci0
I don't know much about bluetooth, and someone wiser probably
would have taken the time to do more background research before
proceeding. Sometimes I like to read a lot before poking at stuff,
sometimes I like to just poke.

The Arch Linux bluetooth [page](https://wiki.archlinux.org/title/Bluetooth)
had a bunch of good pointers for someone who isn't that familiar
with the stack and wanted to start poking at it. I have a super basic
understanding of HCI over UART/USB as the main protocol bluetooth
controllers speak (here's a [diagram](https://naehrdine.blogspot.com/2021/03/bluez-linux-bluetooth-stack-overview.html)).
Let's use that to poke around further.

First I poked at things with bluetoothctl:
```
$ bluetoothctl
[bluetooth]# Agent registered
[bluetooth]# list
[bluetooth]# devices
No default controller available
[bluetooth]# power off
No default controller available
[bluetooth]# power on
No default controller available
[bluetooth]# exit
```
but that yields no controller really.
Then I tried running bluetoothd verbose (-d flag added to the systemd service):
```
$ systemctl cat bluetooth | grep ExecStart
ExecStart=/usr/libexec/bluetooth/bluetoothd -d
```
but the logs don't reveal any real bluetooth controller interaction
happening.

No luck, it seems the bluetooth controller doesn't really
appear to exist from this point of view. I know that hci
is the protocol used to talk to bluetooth devices, so I
start checking the kernel dmesg logs again this time for that:
```
$ dmesg | grep hci0
[    7.563126] fedora kernel: Bluetooth: hci0: Opcode 0x0c03 failed: -110
```

ah ha! Something failed. Where is that and what does it mean?

#### Analyzing the failing command

First, what's that errno mean?
```
$ errno 110
ETIMEDOUT 110 Connection timed out
```

Ok, timing out can mean a lot of things, but means we're failing to get a
response at that point. So either its stuck in that opcode or not hearing us
at all?

Looking at the opcode it tried to send (`0x0c03`).
A quick google shows me that this 16 bit opcode has two parts:
```
Bit: 15 14 13 12 11 10 | 09 08 07 06 05 04 03 02 01 00
-------------------------------------------------------
      0  0  0  0  1  1 |  0  0  0  0  0  0  0  0  1  1
------------------------------------------------------
      OGF field        |     OCF field
```
With that in mind, this means we've got an OGF of `3` and an OCF of `3`.
Looking up what those [mean](https://www.lisha.ufsc.br/teaching/shi/ine5346-2003-1/work/bluetooth/hci_commands.html)
it seems the OGF is a "Host Controller and Baseband" command, and the specific
command is "Reset", i.e. a:
> Command to reset the host controller, link manager and the radio module.

Failing to reset at the beginning makes me think I'm still barking up the
right tree. Let's see if we can make the kernel module louder by enabling
dynamic debug for this module when it loads:
```
$ dmesg -c # clear the logs out
$ modprobe -r btusb
$ modprobe btusb dyndbg=+pfmlt
$ dmesg
[  281.352844] usbcore: deregistering interface driver btusb
[  299.261881] [3963] btusb:btusb_probe:3641: intf 0000000003df02a8 id 000000006f9ad2fe
[  299.261965] [3291] btusb:btusb_open:1835: hci0
[  299.261967] [3291] btusb:btusb_submit_intr_urb:1365: hci0
[  299.261975] [3963] btusb:btusb_probe:3641: intf 0000000065012e3e id 000000006f9ad2fe
[  299.261978] [3291] btusb:btusb_submit_bulk_urb:1487: hci0
[  299.261981] [3291] btusb:btusb_submit_bulk_urb:1487: hci0
[  299.261987] usbcore: registered new interface driver btusb
[  299.262042] [3284] btusb:btusb_send_frame:2084: hci0
[  299.262167] <intr> btusb:btusb_tx_complete:1784: hci0 urb 000000004e8add07 status 0 count 3
[  301.303902] Bluetooth: hci0: Opcode 0x0c03 failed: -110
[  301.303917] [3291] btusb:btusb_flush:1935: hci0
[  301.303920] [3291] btusb:btusb_close:1898: hci0
[  301.303984] <intr> btusb:btusb_intr_complete:1318: hci0 urb 00000000efa42015 status -2 count 0
[  301.304092] <intr> btusb:btusb_bulk_complete:1442: hci0 urb 0000000052323af2 status -2 count 0
[  301.304173] <intr> btusb:btusb_bulk_complete:1442: hci0 urb 00000000a25e07f7 status -2 count 0
```

Yeah, things are failing almost immediately during our probe it seems. Let's
fine that line in the kernel tree:
```
$ git grep "Opcode.*failed"
net/bluetooth/hci_sync.c:                       bt_dev_err(hdev, "Opcode 0x%4.4x failed: %ld", opcode,
```

### A stroke of luck
Now I know where things are failing, but am not sure why.
It is basically the very first shot at talking at the HCI level,
so things seem fundamentally off.

I consider if it's:
1. A power sequencing thing
1. A protocol quirk for the device

I decide that 2 is more likely. Looking around at the USB VID/PID
(`13d3:3610`) that this TX55E reports, I find some threads that others
have this issue and a myriad of people complaining about MediaTek support
in general. Everyone seems to run from the device. I hope to make it work
for them (and myself) instead of returning it.

The file we're failing in has a ton of MediaTek devices defined, and they
use special callbacks for all the HCI stuff with a similar name (MT7922) as
was shown in the prior section!
```
...
        /* MediaTek MT7922 Bluetooth devices */
        { USB_DEVICE(0x13d3, 0x3585), .driver_info = BTUSB_MEDIATEK |
                                                     BTUSB_WIDEBAND_SPEECH },
...
static int btusb_probe(struct usb_interface *intf,
                       const struct usb_device_id *id)
{
        ...
        if (IS_ENABLED(CONFIG_BT_HCIBTUSB_MTK) &&
            (id->driver_info & BTUSB_MEDIATEK)) {
                hdev->setup = btusb_mtk_setup;
                hdev->shutdown = btusb_mtk_shutdown;
                hdev->manufacturer = 70;
                hdev->cmd_timeout = btmtk_reset_sync;
                hdev->set_bdaddr = btmtk_set_bdaddr;
                hdev->send = btusb_send_frame_mtk;
                set_bit(HCI_QUIRK_BROKEN_ENHANCED_SETUP_SYNC_CONN, &hdev->quirks);
                set_bit(HCI_QUIRK_NON_PERSISTENT_SETUP, &hdev->quirks);
                data->recv_acl = btmtk_usb_recv_acl;
                data->suspend = btmtk_usb_suspend;
                data->resume = btmtk_usb_resume;
                data->disconnect = btusb_mtk_disconnect;
        }
        ...
}
```
Okay, MediaTek stuff is *definitely* expected to be quirky it seems!

There's no hit for this VID/PID in the file, so we're just using the generic
callbacks:
```
static int btusb_probe(struct usb_interface *intf,
                       const struct usb_device_id *id)
{
        ...
        hdev->open   = btusb_open;
        hdev->close  = btusb_close;
        hdev->flush  = btusb_flush;
        hdev->send   = btusb_send_frame;
        hdev->notify = btusb_notify;
        hdev->wakeup = btusb_wakeup;
        ...
}
```

Let's see what happens if we treat this as a MediaTek device.

### The patch
Here's the diff, super simple:
```
diff --git a/drivers/bluetooth/btusb.c b/drivers/bluetooth/btusb.c
index 279fe6c115fac51dd941eee2d496392917e083cd..ce534c212431157a382fce75f2b6657b6c9c6c96 100644
--- a/drivers/bluetooth/btusb.c
+++ b/drivers/bluetooth/btusb.c
@@ -610,6 +610,8 @@ static const struct usb_device_id quirks_table[] = {
 	/* MediaTek MT7922 Bluetooth devices */
 	{ USB_DEVICE(0x13d3, 0x3585), .driver_info = BTUSB_MEDIATEK |
 						     BTUSB_WIDEBAND_SPEECH },
+	{ USB_DEVICE(0x13d3, 0x3610), .driver_info = BTUSB_MEDIATEK |
+						     BTUSB_WIDEBAND_SPEECH },
```

I grabbed the related fedora kernel version, applied that, and installed
to minimize changing my testing point.
```
$ # Grab the fedora kernel tree and appropriate branch
$ git clone https://gitlab.com/cki-project/kernel-ark
$ git checkout origin/fedora-6.12
$
$ # Use the same config, accept the defaults on any new changes (olddefconfig)
$ cp /boot/config-6.12.5-200.fc41.x86_64
$ make olddefconfig
$
$ # make the patch...
$
$ # Build, install, reboot, and select from grub
$ make -j`nproc && sudo make -j`nproc` modules_install install
$ reboot
```

I won't detail it much, but I also ran into this [bug](https://lore.kernel.org/lkml/r242oj55chpgo4j5i5ofqwgv5bm7ujruldhf22hnrji7iktogc@5xvd6kkn2tom/)
when building which I chimed in on the thread in hopes to push it to be resolved:
```
arch/x86/tools/insn_decoder_test: error: malformed line 3566260: 2_>:ffffffff81bdd110
```

And bingo! Bluetooth is now working, and I can use the Xbox controller:
```
$ dmesg | grep -i "hci0\|xbox"
    [    7.047388] Bluetooth: hci0: HW/SW Version: 0x008a008a, Build Time: 20241106163512
    [    9.583883] Bluetooth: hci0: Device setup in 2582842 usecs
    [    9.583895] Bluetooth: hci0: HCI Enhanced Setup Synchronous Connection command is advertised, but not supported.
    [    9.644780] Bluetooth: hci0: AOSP extensions version v1.00
    [    9.644784] Bluetooth: hci0: AOSP quality report is supported
    [  876.379876] input: Xbox Wireless Controller as /devices/virtual/misc/uhid/0005:045E:0B13.0006/input/input27
    [  876.380215] hid-generic 0005:045E:0B13.0006: input,hidraw3: BLUETOOTH HID v5.15 Gamepad [Xbox Wireless Controller] on c0:bf:be:27:de:f7
    [  876.429368] input: Xbox Wireless Controller as /devices/virtual/misc/uhid/0005:045E:0B13.0006/input/input28
    [  876.429423] microsoft 0005:045E:0B13.0006: input,hidraw3: BLUETOOTH HID v5.15 Gamepad [Xbox Wireless Controller] on c0:bf:be:27:de:f7
```

#### Submitting the patch
I fixed it locally, but I have no intention of treating this living
room PC as a special computer, it needs to able to get the distro provided
kernel and just work. So I need to get it fixed in the upstream linux kernel.

Since kernel development can seem foreign, I'll give a quick recap
here in hopes maybe someone finds it useful. You:

1. Develop the git patch
1. Test it
1. Verify the git patch is meeting some guidelines with checkpatch.pl
1. Send it via email
1. Review's handled in plain text email
1. Eventually a maintainer may pull it in
1. Maintainers bubble up all the way to torvalds who makes the release

There's more comprehensive guides for [kernel development](https://docs.kernel.org/process/development-process.html)
and [b4](https://b4.docs.kernel.org/en/latest/contributor/prep.html).
Let me give you a quick b4 run through.

For patch/series management, b4 is really wonderful. It is what I use
and handles cover letters, "below the trimmed line patch comments" (must be
a proper name for this but basically comments in the initial patch that
won't be applied by git), versioning, comparing versions, email recipients,
checkpatch, etc.

Here I pretend to use it again like I did to develop and submit the patch:
```
$ # Have b4 make a new series to manage, call it tx55e-bluetooth
$ # a corresponding git branch is made etc
$ b4 prep -n tx55e-bluetooth
$
$ # Do my development and commit like normal
$ git add drivers/bluetooth/btusb.c
$ git commit -s
$
$ # verify my patch is following best practices
$ b4 prep --check
$
$ # Automatically add the appropriate mailing lists and maintainers/reviewers
$ b4 prep --auto-to-cc
$
$ # Edit the cover letter (or in a one patch series, the patch comment
$ # section I failed to come up with a real name for earlier)
$ b4 prep --edit-cover
$
$ # Dry send it for myself to double check, then really do it
$ b4 send -d
$ b4 send
$
$ # the versions tagged, a new cover letter is generated (based on the old
$ # one) with a link to the first version even!
$ # so if changes are needed I just repeat the work and b4 check/send loop
```
And here's a lore [link](https://lore.kernel.org/all/20241224-tx55e-bluetooth-v1-1-e83ebc81507a@gmail.com/)
to the submitted patch.

## Conclusion
A little tour from the bottom of the stack and up a bit further lead me to find
out why the Arch TX55E AX3000 didn't have working Bluetooth out
of the box on linux kernel v6.12.5. It simply wasn't talking the MediaTek
specific way the device expected, so all that was needed was to teach the upstream
kernel to treat this USB device (based on the VID/PID) as a MediaTek variant.

After that HCI worked properly, and now my Bluetooth works! Hopefully
others who in the past had issues with this device now can use it
without issue.
