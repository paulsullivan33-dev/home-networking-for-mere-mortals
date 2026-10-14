# DHCP and Reservations

**The one-sentence version:** DHCP loans addresses to devices
that join your network; a **reservation** pins one address to one
device so servers and printers stop moving. Use reservations for
things you reach *by address*, static IPs on the device only
when the device demands it.

## Leases in plain terms

When a device joins your network — Wi-Fi connected, cable
plugged in — it shouts \"I need an address!\" (that's the DHCP
**discover**). Your router's DHCP server answers with a
**lease**: an address, the network mask, the gateway (the router
itself), and the DNS server, plus an expiry time. Before the
lease expires, the device quietly asks to renew. Nobody notices.

Why *leases* instead of permanent addresses? Because devices come
and go. Your guest's phone doesn't need a permanent seat at the
table; it borrows one for the evening. Typical home lease times
are 12–24 hours. A device that sleeps past its lease (looking at
you, printer) can wake up to find its old address given to
someone else — and every bookmark, app, and port-forward rule
pointing at the old address breaks.

**The refresher, in two sentences:** this is the DHCP half of
[Job 2 in the router model](what-your-router-does.md). If you
want the protocol mechanics, the
[fundamentals guide](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/your-lan/)
has them.

## Reservations: the right default

A **DHCP reservation** tells the router: \"whenever *this*
device asks, always give it *this* address.\" It ties the
device's **MAC address** (its hardware identifier, burned into
the network card) to one IP. The device still uses DHCP — it
doesn't know anything special happened — but it always lands in
the same seat.

Why this beats the alternatives:

| Approach | How it works | Downside |
|---|---|---|
| Nothing (plain DHCP) | Address may change after sleeps | Bookmarks and rules break |
| **Reservation (router UI)** | Router always assigns the same IP | None, for most cases |
| Static IP on the device | Device ignores DHCP, uses a fixed IP | You must track assignments by hand; two devices can collide |

**Give reservations to:** servers, printers, cameras, anything
you reach by address or that other rules point at. Don't bother
for phones and laptops — let them roam the pool.

### Setting one up (typical router UI)

Router interfaces differ — this is the shape they all share:

1. Find your router's **DHCP** or **LAN** settings page (often
   under \"Advanced → LAN\" or \"Network → DHCP Server\").
2. Find the list of **connected devices** (or \"DHCP clients\")
   and locate the device — match by its current IP or its name.
3. Add a **DHCP reservation / static lease**: pick the device's
   MAC address, choose an IP *outside the DHCP pool but inside
   your network* — for example, with a pool of
   `192.168.1.100`–`192.168.1.199`, use something like
   `192.168.1.10`.
4. Save. Reconnect the device (or wait for its lease to renew)
   so it picks up the reserved address.

**Check:** ask the router for its DHCP client list after the
device reconnects — the device's MAC should now show your
chosen address, and it will survive reboots and long sleeps.

### The same thing in dnsmasq

If your DHCP server is a Linux box or a Pi (Pi-hole-style
setups run dnsmasq or its equivalent), the reservation lives in
a config file instead of a web UI. This snippet was validated
with the real dnsmasq (`dnsmasq --test` reported \"syntax check
OK\"):

```text
# Example dnsmasq config: DHCP pool, one reservation, one local DNS entry
interface=eth0
dhcp-range=192.168.1.100,192.168.1.199,24h
dhcp-host=AA:BB:CC:DD:EE:FF,nas,192.168.1.10,infinite
address=/home.arpa/192.168.1.1
```

Line by line:

- `dhcp-range` — the pool: 100 addresses
  (`192.168.1.100` through `192.168.1.199`), 24-hour leases.
- `dhcp-host` — the reservation: the device whose MAC is
  `AA:BB:CC:DD:EE:FF` always gets `192.168.1.10` (named `nas`
  here) with an effectively permanent lease.
- `address=` — a local DNS entry, covered in
  [Names on Your LAN](names-on-your-lan.md).
- `interface=eth0` — which network card serves DHCP; match it
  to your box.

**Find a device's MAC address** (typical output — yours will
differ):

```bash
# Linux
ip link show | grep -A1 "state UP" | grep link/ether
```

```text
link/ether aa:bb:cc:dd:ee:ff brd ff:ff:ff:ff:ff:ff
```

On Windows: `ipconfig /all` and look for \"Physical Address.\"
On a Mac: System Settings → Wi-Fi → Details → the hardware
(MAC) address. Most router UIs also show it next to each
connected device.

## When a static IP on the device is right

Rarely, but it happens: some devices (old printers, some
cameras, a server whose DHCP client you don't trust) let you
type an address directly into *their* settings. If you do this:

1. Pick the address from **outside the DHCP pool** (else the
   router may hand \"your\" address to a guest's phone and you
   get a collision — two devices, one address, both broken).
2. Enter the mask (`255.255.255.0` for a `/24`), the gateway
   (your router, e.g. `192.168.1.1`), and DNS (the router, or
   your local DNS server).
3. **Write it down.** Static IPs are invisible to the router's
   DHCP list, so they're the first thing forgotten during the
   next troubleshooting session.

If the device supports DHCP at all, a reservation is the same
result with none of the bookkeeping. Reserve first; go static
only when the device gives you no choice.

## What DHCP can't do for you

- DHCP doesn't give names — it hands out numbers. Names are the
  next two chapters.
- DHCP doesn't cross subnets by itself. Devices on your guest
  network get addresses from *that* network's pool (see
  [Guest Networks and VLANs](guest-networks-and-vlans.md)).
- A reservation doesn't make a device reachable *from the
  internet* — that's NAT and firewall territory
  ([Port Forwarding](port-forwarding.md)).

**Next:** [DNS at Home](dns-at-home.md)
