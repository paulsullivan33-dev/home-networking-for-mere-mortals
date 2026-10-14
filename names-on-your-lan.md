# Names on Your LAN

**The one-sentence version:** You don't have to memorize
`192.168.1.10` — three different mechanisms (mDNS, router
hostnames, local DNS records) let you reach home services by
name, each with its own trade-offs.

## Why names beat numbers

Addresses change (see [DHCP and Reservations](dhcp-and-reservations.md));
names don't have to. `http://nas:8000` in a bookmark survives a
rebuilt network; `http://192.168.1.10:8000` survives only until
the next re-addressing. The goal: type a name, get the machine.

Three ways to get there, simplest first.

## 1. mDNS: names that announce themselves

**mDNS** (multicast DNS) is the zero-configuration option: a
device shouts \"I'm `printer.local`, my address is
`192.168.1.25`!\" to the local network, and every other device
hears it. No server, no configuration. Apple's **Bonjour** and
most Linux **Avahi** setups speak it; Windows 10 and later
understand `.local` names too.

Try it:

```bash
ping printer.local
```

If your printer (or anything else) answers, mDNS is working and
you just got a free name.

**The catches, honestly:**

- The name is always `<something>.local` — you don't get to
  choose a prettier suffix, and `.local` names never work over
  the internet.
- It only works on the *same* broadcast network. Devices on a
  guest network or a different VLAN won't hear the shouts (see
  [Guest Networks and VLANs](guest-networks-and-vlans.md)).
- Name clashes resolve themselves awkwardly (`printer-2.local`).
  Fine for a home; not a system to build on.
- Some VPN software and \"smart\" switches filter the
  announcements. When `.local` names mysteriously stop working,
  that's usually why.

mDNS is the right answer for: printers, and quickly reaching a
new device before you've set up anything proper.

## 2. Router hostnames: the DHCP name list

Most routers keep a list of DHCP clients *with the names the
devices reported* (`DESKTOP-ABC123`, `android-xyz`, or whatever
the device calls itself) and answer DNS queries for those names
from the LAN. If your router does this, `ping
livingroom-pc` may just work with zero setup.

**The catches, honestly:**

- Device-reported names are often ugly or duplicated
  (`android-7f3a2c` × three phones).
- Not all routers serve this list over DNS — some only show it
  in the web UI. If `ping <name>` fails, yours doesn't.
- Renaming has to happen *on the device* (computer name
  settings), and the router picks it up at the next DHCP renew.

This is the right answer for: casually reaching a PC by name
when it already works. Don't build services on it.

## 3. Local DNS records: names you control

This is the grown-up version, and the one this guide
recommends for anything you use regularly: explicit records in
*your* DNS (Pi-hole/AdGuard \"Local DNS,\" a dnsmasq
`address=/` line, or the router's static DNS entries) mapping
names you choose to addresses you chose:

| Name | Address | What it reaches |
|---|---|---|
| `nas.home.arpa` | `192.168.1.10` | File server |
| `printer.home.arpa` | `192.168.1.25` | Printer |
| `media.home.arpa` | `192.168.1.10` | Media server on the same box |

Two names can point at the same address (the NAS runs both
services — the *name* picks the service, which is exactly what
[Reverse Proxies](reverse-proxies.md) exploits). The
[dnsmasq line from the DHCP chapter](dhcp-and-reservations.md#the-same-thing-in-dnsmasq)
(`address=/home.arpa/192.168.1.1`, validated with
`dnsmasq --test`) is this mechanism in miniature; Pi-hole and
AdGuard Home expose the same thing as \"Local DNS records\" in
their web UIs (per-project docs — the UI steps weren't verified
here).

Use the `.home.arpa` suffix (reserved for home networks — see
[DNS at Home](dns-at-home.md)). These names resolve only inside
your house, which is the point: they're the *inside* half of
split-horizon DNS.

**Check:** look up one of your names from a LAN device and
confirm the private address comes back:

```bash
getent hosts nas.home.arpa
```

```text
192.168.1.10      nas.home.arpa
```

(Example — yours shows your own address. If it fails, your
device isn't using your local DNS: check the DHCP-advertised
DNS server, and see [Troubleshooting](troubleshooting.md).)

## Which to use when

| Situation | Use |
|---|---|
| Printer, quick ad-hoc reach | mDNS (`printer.local`) |
| \"What's that PC called?\" | Router hostname list, if it works |
| Services you bookmark, automate, or give TLS certs | Local DNS records (`*.home.arpa`) |

The pattern that scales: **reservations** (chapter 2) pin the
address, **local DNS records** (this chapter) pin the name, and
a **reverse proxy** (next chapter) lets one address serve many
named services. Each layer removes one thing you used to
memorize.

**Next:** [Reverse Proxies](reverse-proxies.md)
