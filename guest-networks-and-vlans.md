# Guest Networks and VLANs

**The one-sentence version:** A **guest network** is a second
Wi-Fi that can't see your stuff; a **VLAN** is the general
version of that idea — separate virtual networks on the same
wires — and **IoT segregation** (quarantining your smart bulbs
from your laptop) is why you'd want one.

## Guest Wi-Fi: the easy win

Almost every router offers a **guest network**: a second SSID
(name) like `MyHouse-Guest` with its own password. Guests join
it, get internet, and — crucially — **can't see your LAN**.
Their phones can't reach your NAS, your printer, or your
laptop. Your main network can't see them either.

This is the single highest-value, lowest-effort network
segmentation a home can do. Turn it on:

1. Find **Guest Network** in the router's Wi-Fi settings.
2. Enable it, give it a distinct name and a real password (not
   the same as your main Wi-Fi).
3. Confirm **\"allow guests to see each other\" is off** if
   offered (it usually is by default — guests isolated from
   each other too), and that **access to the local network is
   denied** (the whole point).

**Check:** join the guest network with your phone and try to
reach something on your LAN — your router's admin page, or
`ping` your laptop's address. It should fail; the internet
should work. If the router page loads, the isolation isn't on.

When guest mode isn't enough: some routers' guest networks are
2.4 GHz-only, or they block the mDNS announcements your
Chromecast needs. That's a sign you've outgrown the simple
version — read on.

## What a VLAN is, without jargon

A **VLAN** (Virtual LAN) is a way to run *multiple separate
networks* over the *same* physical cables and Wi-Fi radios. The
switch or access point **tags** each packet with a number
saying which virtual network it belongs to — that's the entire
trick, standardized as 802.1Q tagging. Devices on VLAN 10 can't
talk to devices on VLAN 20 unless a router explicitly allows
it, even though they share the same wire.

The home analogy: one apartment building, but each floor's
hallway doors only open with that floor's key. Same building,
separate hallways.

| Concept | Plain meaning |
|---|---|
| VLAN ID (a number, e.g. 10, 20, 30) | Which virtual network a packet belongs to |
| Tagged port | A port carrying multiple VLANs (router ↔ switch, switch ↔ access point) |
| Untagged / access port | A port belonging to exactly one VLAN (plug a camera in, it's on that VLAN, done) |
| Trunk | A cable carrying several tagged VLANs between boxes |

Your ISP router probably doesn't do VLANs. \"Prosumer\" gear
(Ubiquiti UniFi, TP-Link Omada, MikroTik, OpenWrt-flashed
routers) does. This is enthusiast territory: powerful, and
easy to misconfigure into \"nothing can reach anything.\"

## IoT segregation: the motivating example

Here's why anyone bothers. Your smart bulbs, cameras, TV, and
thermostat are computers you didn't choose, running software
you can't update, made by vendors with mixed security records —
and they sit on the *same* network as the laptop with your
banking. One compromised camera shouldn't be able to scan your
laptop. But on a flat network, it can.

The standard three-network design:

| VLAN | Who lives here | Can reach |
|---|---|---|
| Main (`192.168.1.0/24`) | Your PCs, phones, NAS | Everything (you allow it) |
| IoT (`192.168.3.0/24`) | Cameras, bulbs, plugs, TV | The internet only — not your main network |
| Guest (`192.168.2.0/24`) | Visitors | The internet only |

(Each is a full `/24` — 254 usable addresses — verified with
Python's `ipaddress` module during writing. You will not fill
them; the roominess is free.)

The firewall rules between them are the actual security:

- IoT → internet: **allow** (the bulb needs its cloud).
- IoT → main: **deny** (the compromised camera can't reach
  your laptop).
- Main → IoT: **allow** (you still want to open the camera
  app and see the feed — connections *you* start are fine).

That last line surprises people: blocking is directional.
Your phone on the main network can open a connection *to* the
camera; the camera can't open one *to* your phone. Same idea as
the router firewall's default in
[What Your Router Does](what-your-router-does.md), applied
between your own networks.

## The honest costs

- **mDNS breaks across VLANs.** Your phone on main can't
  discover the Chromecast on IoT via `.local` names, because
  the announcements don't cross the boundary. Fixes exist
  (an mDNS *reflector*/*repeater* on the router), but this is
  the #1 \"I segmented and now casting doesn't work\"
  complaint. Budget for it.
- **It's a second network to administer.** DHCP pools, DNS,
  firewall rules — everything in this guide, per VLAN.
- **Cheap gear lies.** Some \"VLAN-capable\" switches only do
  port-based separation without real tagging. If devices on
  different VLANs can still ping each other, the tagging isn't
  actually happening.

**Start here, in order:** guest Wi-Fi on the router you own
(five minutes, real protection for visitors) → a separate IoT
SSID if your router offers multi-SSID → full VLANs only when
you've outgrown those and enjoy this kind of project. Each
step is useful on its own; none requires the next.

**Next:** [Wi-Fi That Matters](wifi-that-matters.md)
