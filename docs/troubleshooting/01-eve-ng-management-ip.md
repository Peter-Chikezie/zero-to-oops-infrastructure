# 01: EVE-NG lost its IPv4 address after a reboot

*2026-11-25*

First real problem of the lab, and it showed up before I'd built anything.

I'd just finished the EVE-NG first-boot wizard (DHCP, domain `oops.local`) and rebooted. The console banner came back with an `fe80::` address and nothing else. No IPv4, which means no web UI.

## Figuring out what fe80 meant

`fe80::` is IPv6 link-local. Every interface makes one up for itself as soon as it has link, without asking anyone. So pnet0 was physically up. It just never got an answer from DHCP. It's the same idea as a Windows box falling back to 169.254.x.x.

My first guess was VMware bridging to the wrong adapter, which apparently happens a lot. So I ran DHCP by hand to see what happened:

```
dhclient -v pnet0
...
bound to 10.154.x.x -- renewal in 1646 seconds
```

It got an address straight away, so the bridge was fine. I checked `/etc/network/interfaces` and `iface pnet0 inet dhcp` was there too. So the config was right and DHCP worked by hand. It just didn't work at boot.

My laptop was on my phone's hotspot. At boot pnet0 asks for an address once, and if the hotspot isn't awake yet, nothing answers and it gives up.

## What I changed

I took the VM off Bridged and put it on VMware NAT (VMnet8) with a static IP. VMnet8 is a private network that lives inside the laptop, so EVE-NG keeps the same address no matter what Wi-Fi I'm on, and it still reaches the internet through VMware's NAT. Bonus: the EVE-NG login page is no longer sitting on a shared network for anyone to find.

My VMnet8 was 192.168.24.0/24. Here's what's on it:

* `.1` is the laptop's own VMnet8 adapter
* `.2` is VMware's NAT gateway (it also does DNS)
* `.10` is EVE-NG
* `.128` to `.254` is VMware's DHCP pool

The pool was `.3` to `.254` by default, so I moved the start up to `.128`. If EVE-NG's `.10` had stayed inside the pool, VMware could have handed `.10` to another VM later and I'd have a duplicate IP to chase.

The pnet0 block now looks like this:

```
iface pnet0 inet static
    address 192.168.24.10
    netmask 255.255.255.0
    gateway 192.168.24.2
    dns-nameservers 192.168.24.2
    pre-up ip link set dev eth0 up
    bridge_hw eth0
    bridge_ports eth0
    bridge_stp off
```

Don't delete the `bridge_*` lines. pnet0 is a Linux bridge sitting on top of eth0. Without them you get the right IP on an interface that's connected to nothing.

## Where it ended up

After a full power-off and boot, pnet0 comes up on 192.168.24.10 by itself and the web UI loads from the laptop. I also changed the default admin password before doing anything else.
