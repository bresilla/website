+++
type = "post"
status = "published"
date = "2020-08-09"
readingtime = 8

slug = "usbip"
title = "Set up a Raspberry Pi as a USB-over-IP server"
thumbnail = "thumbnail.png"
foot = "It's all right letting yourself go as long as you can let yourself back - Mick Jagger"
categories = ["FUSE"]
series = ["USB"]
part = "1"
tags = ["usbip", "linux", "hardware", "usb", "network", "FUSE"]

description = "Share a USB device from a Raspberry Pi and attach it to another Linux machine over a trusted network."
punchline = "USB/IP carries USB requests over a TCP connection. The device stays plugged into the Raspberry Pi, but the client talks to it as if it were local."
tldr = "Export on the Pi, attach on the client, and keep TCP port 3240 away from untrusted networks."

credits = [
    "https://docs.kernel.org/usb/usbip_protocol.html",
    "https://manpages.debian.org/unstable/usbip/usbip.8.en.html",
    "https://manpages.debian.org/unstable/usbip/usbipd.8.en.html",
]

[style]
    accent = "#04f977"
    theme = "dark"
    width = "60%"
+++

# Why I use USB/IP

Sometimes the machine that needs a USB device is nowhere near the device. A virtual machine may not have useful USB passthrough, a home server may sit in a cupboard, or a Raspberry Pi may simply be the only computer in the right room.

USB/IP solves that awkward little problem. The Pi exports a physical USB device, and another Linux machine imports it through a virtual host controller. Applications on the client see an ordinary USB device; they do not need a special network-aware driver.

There is one important catch: USB/IP is transport, not a secure tunnel. Do not expose its TCP port to the internet. I use it only on a trusted LAN or through a VPN, with a firewall limiting which client may connect.

# The two halves

The names become much easier once the direction is clear:

- The **server** has the physical USB device and runs `usbipd`.
- The **client** loads `vhci-hcd` and attaches the remote device.
- The device's normal driver runs on the client.

The examples below use Raspberry Pi OS or another Debian-family system on both ends. Package names differ on some distributions, but the USB/IP commands are the same.

# Configure the Raspberry Pi

Install the userspace tools:

```console
$ sudo apt update
$ sudo apt install usbip
```

Load the server-side kernel modules:

```console
$ sudo modprobe usbip-core
$ sudo modprobe usbip-host
```

If `modprobe` reports that a module does not exist, check that the running kernel includes USB/IP support and that its matching modules package is installed. Mixing a newly installed module package with an old, still-running kernel is an easy way to waste an afternoon; reboot into the new kernel before debugging anything stranger.

Now inspect the local devices:

```console
$ usbip list --local
 - busid 1-1.2 (1050:0407)
   Yubico.com : Yubikey 4/5 OTP+U2F+CCID (1050:0407)
```

The value after `busid` is what matters. Mine is `1-1.2`; yours will probably be different. Before exporting a storage device, unmount every filesystem on it. Before exporting anything else, stop local programs that are using it.

Bind the device to the USB/IP host driver, then start the daemon:

```console
$ sudo usbip bind --busid 1-1.2
$ sudo usbipd --daemon
```

Binding disconnects the device from its normal driver on the Pi. That is expected: while exported, it belongs to the remote client rather than both machines at once.

Confirm that it is available:

```console
$ usbip list --remote 127.0.0.1
```

By default `usbipd` listens on TCP port `3240`. Allow that port only from the client or the VPN subnet. The exact firewall command depends on what already manages the Pi's firewall; do not paste an `iptables` rule into a machine whose rules are owned by nftables, UFW, or a router.

# Attach the device from Linux

On the client, install the tools and load the virtual host controller:

```console
$ sudo apt update
$ sudo apt install usbip
$ sudo modprobe vhci-hcd
```

Ask the Pi what it exports. Replace `pi-usb.lan` with its hostname or address:

```console
$ usbip list --remote pi-usb.lan
```

Attach the bus ID reported by that command:

```console
$ sudo usbip attach --remote pi-usb.lan --busid 1-1.2
```

At this point `lsusb`, `udevadm`, and the application's normal tools should see the device locally. `usbip port` shows the imported devices and their virtual port numbers:

```console
$ usbip port
```

To disconnect cleanly, use the port number from that output, not the remote bus ID:

```console
$ sudo usbip detach --port 0
```

Then release the device on the Pi so its local driver can claim it again:

```console
$ sudo usbip unbind --busid 1-1.2
```

# Survive a reboot

The modules can load automatically at boot. On the Pi:

```console
$ printf '%s\n' usbip-core usbip-host | sudo tee /etc/modules-load.d/usbip-server.conf
```

On the client:

```console
$ echo vhci-hcd | sudo tee /etc/modules-load.d/usbip-client.conf
```

Some distributions ship a service for `usbipd`; enable that if it exists. Otherwise a tiny local systemd unit is enough for the daemon:

```ini
[Unit]
Description=USB/IP daemon
After=network.target

[Service]
Type=forking
ExecStart=/usr/sbin/usbipd --daemon

[Install]
WantedBy=multi-user.target
```

Save it as `/etc/systemd/system/usbipd.service`, then run:

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now usbipd.service
```

I deliberately keep the `usbip bind` step separate. A bus ID describes the device's position in the current USB topology and can change when hubs or ports move. For a permanent appliance, bind from a carefully tested udev rule that matches the device's vendor, product, and preferably serial number. A service hard-coded to `1-1.2` will eventually export the wrong thing or nothing at all.

# What works, and what gets weird

Keyboards, security tokens, serial adapters, programmers, and modest storage devices are usually straightforward. Latency-sensitive audio, webcams, and other isochronous devices are much less forgiving. Network jitter still exists even when the USB device appears local.

When an attach fails, I check these in order:

1. `usbip list --remote HOST` can reach the daemon.
2. TCP port `3240` is allowed only along the intended path.
3. `usbip-host` is loaded on the Pi and `vhci-hcd` on the client.
4. The device is bound on the Pi and not already attached elsewhere.
5. `journalctl -u usbipd`, `dmesg`, and `usbip port` agree about what happened.

That is the whole trick. The network link may be remote, but ownership is still exclusive and the usual USB rules still apply. Treat the cable as longer, slower, and considerably less trustworthy.
