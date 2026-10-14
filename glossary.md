# Glossary

Every jargon term used in this guide, defined once, in plain
language.

**Access point** — A box (or a router in access-point mode)
that adds Wi-Fi coverage to an existing network. Unlike a
second router, it doesn't do NAT or DHCP — it just bridges
wireless clients onto the LAN.

**AdGuard Home** — An open-source, self-hosted DNS server with
built-in ad/tracker blocking, similar in purpose to Pi-hole.
Runs on a Raspberry Pi or any always-on computer.

**Avahi** — The mDNS implementation most Linux systems use
(Apple's is called Bonjour). It lets Linux machines both
announce and discover `.local` names.

**Band (Wi-Fi)** — A slice of radio spectrum Wi-Fi uses:
2.4 GHz, 5 GHz, or 6 GHz. Lower frequencies travel farther
through walls; higher ones carry more data but fade faster.

**Band steering** — A router feature that nudges each device
toward the best Wi-Fi band it supports, when all bands share
one network name.

**Blocklist** — A published list of domains (ads, trackers,
malware) that a filtering DNS server refuses to resolve.

**Bridge mode** — A setting on an ISP gateway that turns off
its routing, NAT, DHCP, and Wi-Fi so it acts as just a modem,
leaving those jobs to your own router. The standard cure for
double NAT.

**Broadcast** — A packet addressed to every device on the
local network at once. mDNS announcements and DHCP discovery
use broadcasts (or the closely related multicast).

**Certificate (TLS)** — A digital document that proves \"this
server really is `files.home.arpa`\" and enables encrypted
HTTPS. Issued by authorities like Let's Encrypt; browsers
trust the authority, so they trust the server.

**CGNAT (carrier-grade NAT)** — Address translation done by
your ISP, shared across many customers. Your router's WAN
address comes from `100.64.0.0/10` and isn't reachable from
the internet, so inbound connections (port forwarding) can't
work.

**Channel (Wi-Fi)** — A sub-slice of a Wi-Fi band. Nearby
networks on the same channel take turns transmitting. On
2.4 GHz, only channels 1, 6, and 11 avoid overlapping each
other.

**Cloud relay** — A remote-access method where your device and
your phone both connect out to the vendor's server, which
introduces them. No router configuration needed; the vendor
is in the middle.

**DHCP (Dynamic Host Configuration Protocol)** — The system by
which a server (usually your router) loans network settings —
address, mask, gateway, DNS — to devices as they join.

**DHCP pool / range** — The set of addresses the DHCP server
is allowed to hand out, e.g. `192.168.1.100`–`192.168.1.199`.

**DMZ (router feature)** — A router setting that forwards
*all* inbound ports to one inside machine. Occasionally
useful for putting your own router behind an ISP box;
dangerous as a general \"make it work\" button, because it
removes the firewall for that machine entirely.

**DNS (Domain Name System)** — The phone book: translates
human names (`example.com`) into IP addresses.

**DNS forwarder** — A DNS server (usually in your router)
that doesn't look names up from scratch — it passes queries
to upstream servers and caches the answers for the LAN.

**DNS-over-HTTPS (DoH) / DNS-over-TLS (DoT)** — Ways of
encrypting DNS queries so they can't be read or altered in
transit. Changes who can see your lookups, not whether sites
are safe.

**Double NAT** — Two layers of address translation in a row
(your router behind your ISP's router, or behind CGNAT).
Breaks or complicates inbound connections and port
forwarding.

**Firewall** — The router's bouncer: allows outbound
connections and their replies, drops unsolicited inbound
connections, except where you've explicitly poked holes.

**Forward proxy** — A server that fetches the web *on behalf
of clients*. The opposite direction from a reverse proxy;
not covered in this guide beyond the distinction.

**Gateway** — The device a machine sends non-local traffic
to — in a home, your router (e.g. `192.168.1.1`).

**Hairpin NAT** — A router feature that lets a LAN device reach
another LAN device via the router's *public* address (the
traffic goes out and \"hairpins\" back in). Not all routers do
it — which is why split-horizon DNS (answering with the inside
address directly) is the more reliable fix.

**Guest network** — A second Wi-Fi network (separate name and
password) that gets internet access but is isolated from
your main LAN.

**home.arpa** — The domain suffix reserved for home
networks. Use it for private names (`nas.home.arpa`); those
names only resolve inside your house.

**Hostname** — A device's own name on the network
(`livingroom-pc`), as distinct from a DNS name. Routers often
list DHCP clients by hostname.

**Internal CA** — A certificate authority you run yourself
(Caddy can do this) for issuing TLS certificates to
internal-only names that public authorities can't verify.

**IoT (Internet of Things)** — Network-connected gadgets:
bulbs, plugs, cameras, thermostats. Computers you didn't
choose, on your network anyway — the reason for
segregation.

**IP address** — A device's numeric network address, e.g.
`192.168.1.10`.

**IPv4 / IPv6** — The two address formats in use. Home LANs
are overwhelmingly IPv4 (`192.168.1.x`); IPv6 adoption on
the ISP side varies.

**Lease (DHCP)** — A borrowed IP address with an expiry
time. Devices renew automatically; a device asleep past
expiry can wake up with a different address.

**Link-local address** — An address a device assigns itself
when no DHCP server answers (`169.254.x.x` on IPv4). Seeing
one means "I asked for an address and nobody replied" —
useful as a diagnostic, useless for actual networking.

**Let's Encrypt** — A free, automated certificate authority.
Caddy and certbot obtain TLS certificates from it without
manual paperwork.

**Local DNS record** — A name→address entry in *your* DNS
server (Pi-hole/AdGuard \"Local DNS,\" a dnsmasq line, or
the router's static DNS) that only resolves inside your
house.

**MAC address** — A network interface's hardware identifier
(`aa:bb:cc:dd:ee:ff`), burned in at the factory. DHCP
reservations tie an address to a MAC.

**mDNS (multicast DNS)** — Zero-configuration naming: devices
announce their own `<name>.local` addresses to the LAN. No
server needed; doesn't cross subnets.

**Mesh VPN** — A VPN where every device connects directly to
every other device (no central hub), e.g. Tailscale. Covers
remote access without port forwarding.

**Mesh Wi-Fi** — Multiple Wi-Fi units cooperating to cover a
house. Different \"mesh\" from a mesh VPN — it extends
wireless coverage; it doesn't encrypt remote access.

**NAT (Network Address Translation)** — The router's
translator: rewrites private LAN addresses to its one public
address on outbound traffic (keeping a table), and reverses
it for replies. The reason many devices share one public IP
— and the reason unsolicited inbound traffic gets dropped.

**NAT traversal** — Techniques (used by mesh VPNs and
voice/video calls) for establishing direct connections
between devices that are both behind NAT, without port
forwarding — typically by having both sides dial out to a
coordinator that introduces them.

**Pi-hole** — An open-source, self-hosted DNS server (usually
on a Raspberry Pi) that blocks ads and trackers
house-wide by refusing to resolve blocklisted domains.

**Port** — A numbered sub-address on a machine (0–65535)
identifying *which service* should get a packet: 80/443 for
web, 53 for DNS, 22 for SSH, 67/68 for DHCP.

**Port forwarding** — A standing router rule sending
internet traffic on a given port to a specific inside
machine. A hole in the firewall, open to the whole internet.

**Powerline adapter** — A gadget that carries network traffic
over your house's electrical wiring. Works, but speed varies
a lot with the wiring.

**Private address** — An IP address valid only inside
private networks (`192.168.x.x`, `10.x.x.x`,
`172.16.x.x`–`172.31.x.x`). Not routable on the internet;
NAT translates them.

**Public address** — An IP address reachable on the
internet, assigned to your router by your ISP.

**Reservation (DHCP)** — A router setting that always
assigns the same IP address to a given MAC address. The
device still uses DHCP; it just always lands in the same
seat.

**Reverse proxy** — A server that sits in front of your
services: it takes web requests on the standard ports,
matches the requested *name*, and forwards each request to
the right inside machine. One front door, many services —
and one place to terminate TLS.

**Server block (nginx)** — One named-site section of an
nginx config: which name it serves and where to forward
those requests.

**Split-horizon DNS** — Giving different answers to the same
name depending on who's asking: the private address to LAN
devices, nothing (or the public address) to the internet.

**SSID** — A Wi-Fi network's broadcast name (`MyHouse`,
`MyHouse-Guest`).

**Static IP** — An address typed directly into a device's
settings instead of obtained via DHCP. Invisible to the
router's DHCP list — write it down, and keep it outside the
DHCP pool.

**Subnet** — A subdivision of an address range, written like
`192.168.1.0/24` (254 usable addresses,
`192.168.1.1`–`192.168.1.254`).

**Tag (VLAN)** — The number stamped on a packet saying which
virtual network it belongs to. A **tagged port** carries
packets for several VLANs (each tagged); an **untagged**
(aka access) port belongs to exactly one VLAN, and anything
plugged into it is on that VLAN with no configuration.

**TLS (Transport Layer Security)** — The encryption behind
HTTPS. A certificate lets the browser verify the server's
identity before the encrypted conversation starts.

**Certbot** — The standard client for obtaining free TLS
certificates from Let's Encrypt, commonly paired with nginx
(which, unlike Caddy, doesn't fetch certificates itself).

**TTL (time to live)** — How long a DNS answer may be
cached. Why DNS changes take time to take effect.

**Trunk (VLAN)** — A cable carrying several tagged VLANs
between network boxes (router ↔ switch, switch ↔ access
point).

**Tunnel** — An encrypted path carrying one network's
traffic across another network. VPNs are tunnels.

**UPnP (Universal Plug and Play)** — Lets programs on your
LAN open their own port-forwarding rules automatically.
Convenient; indiscriminate — it can't tell your game from
malware.

**Upstream DNS** — The servers your router's DNS forwarder
asks when it doesn't know an answer: your ISP's, or a
public one like `1.1.1.1` / `8.8.8.8`.

**VLAN (Virtual LAN)** — Multiple separate logical networks
sharing the same physical cables, kept apart by numeric
tags on each packet (802.1Q tagging). Devices on different
VLANs can't talk unless a router allows it.

**VPN (Virtual Private Network)** — An encrypted tunnel
making a remote device behave as if on a private network.
The classic hub-and-spoke kind tunnels everything through
one office/home endpoint; mesh kinds connect devices
directly.
