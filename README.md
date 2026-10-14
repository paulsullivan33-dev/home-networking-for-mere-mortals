# Home Networking for Mere Mortals

**Home networking** is everything between your devices and the
internet — the router on the shelf, the Wi-Fi in the walls, the
names you type instead of numbers, and the services you run on
the little computer in the spare room. This guide is the
*practical sequel* to
[Networking for Mere Mortals](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/):
that guide teaches what networking *is*; this one teaches how
to *run* a home network on purpose — addresses that stay put,
names instead of numbers, one front door for many services,
Wi-Fi that reaches the back room, and remote access without
regret.

No CS degree required. Ideas first, then real commands — and
honest notes about what each idea costs you.

## What this guide covers

- **What your router does** — one box, four jobs: NAT, DHCP,
  firewall, DNS forwarder. The mental model everything else
  hangs on.
- **DHCP and reservations** — leases in plain terms; pinning
  addresses to MACs in the router UI (or dnsmasq); reservations
  vs static IPs.
- **DNS at home** — the router's forwarder, Pi-hole/AdGuard
  blocking concepts, split-horizon DNS (inside name vs outside
  name).
- **Names on your LAN** — mDNS (`.local`), router hostnames,
  and local DNS records you control.
- **Reverse proxies** — one port in, many services: Caddy and
  nginx, with validated example configs.
- **Port forwarding** — what it is, how to think about it, and
  why to avoid it in favor of tunnels.
- **Guest networks and VLANs** — guest Wi-Fi in plain terms;
  VLANs without jargon; IoT segregation as the reason.
- **Wi-Fi that matters** — bands in practical terms, placement,
  channels, and when to stop tweaking.
- **Remote access, compared** — port forwarding vs VPN vs mesh
  vs cloud relay, in one honest table.
- **Double NAT and gotchas** — ISP router + your router, CGNAT,
  UPnP: what breaks and why.
- **Troubleshooting** — symptom → fix for the failures you'll
  actually meet.

## What it doesn't cover

The fundamentals already taught in
[Networking for Mere Mortals](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/)
— what networking is, IP addresses, subnetting, DNS basics,
ports and protocols, LAN basics, Wi-Fi basics, the internet
path, and diagnostic tools. This guide *links* to those
chapters instead of re-teaching them; each chapter opens with
at most a two-sentence refresher where a practical step needs
one. It also doesn't teach Tailscale setup (that's the
[Tailscale guide](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/)),
general security hygiene (the
[cybersecurity guide](https://paulsullivan33-dev.github.io/cybersecurity-for-mere-mortals/)),
or enterprise networking — this is a home, not a data center.

## How to use this guide

Read chapter 1 first — the four-jobs model is the vocabulary
the rest of the guide speaks. After that, chapters stand
mostly alone: jump to the problem you have. Chapters end with
a **Next** link if you want the guided path.

**Validation notes, honestly:** every subnet/IP calculation
was checked with Python's `ipaddress` module (a `/24` really
is 254 usable addresses; the DHCP pool `192.168.1.100`–`199`
really is 100 addresses). The Caddyfile example was validated
with the real Caddy binary (`caddy validate`, exit 0); the
nginx server block was validated with the real nginx binary
(`nginx -t`: \"syntax is ok\", \"test is successful\"); the
dnsmasq snippet was validated with the real dnsmasq
(`dnsmasq --test`: \"syntax check OK\"). Router UI steps are
described as **typical** — menus vary by vendor — and command
outputs shown are **examples** (your addresses will differ),
not captured output. Pi-hole/AdGuard install steps are from
the projects' docs, not verified here. The RFC 6598 citation
(CGNAT shared address space `100.64.0.0/10`) was verified
against the published RFC. Anything else not runnable in this
environment is labeled **\"from the docs\"** or **\"typical
output.\"** Full detail is in [A note on verification](#a-note-on-verification)
below.

## Chapters

1. [What Your Router Does](what-your-router-does.md) — one
   box, four jobs: NAT, DHCP, firewall, DNS forwarder.
2. [DHCP and Reservations](dhcp-and-reservations.md) —
   leases, pinning addresses to devices, reservations vs
   static IPs.
3. [DNS at Home](dns-at-home.md) — the forwarder, Pi-hole /
   AdGuard concepts, split-horizon DNS.
4. [Names on Your LAN](names-on-your-lan.md) — mDNS, router
   hostnames, local DNS records you control.
5. [Reverse Proxies](reverse-proxies.md) — one port in, many
   services; validated Caddy and nginx examples.
6. [Port Forwarding](port-forwarding.md) — what it is, why to
   avoid it, and the better answers.
7. [Guest Networks and VLANs](guest-networks-and-vlans.md) —
   guest Wi-Fi, VLANs without jargon, IoT segregation.
8. [Wi-Fi That Matters](wifi-that-matters.md) — bands,
   placement, channels; when to care and when not to.
9. [Remote Access, Compared](remote-access-compared.md) —
   forwarding vs VPN vs mesh vs relay, honestly compared.
10. [Double NAT and Gotchas](double-nat-and-gotchas.md) —
    ISP router + your router, CGNAT, UPnP.
11. [Troubleshooting](troubleshooting.md) — symptom → fix for
    the common failures.
12. [Glossary](glossary.md) — every jargon term, defined once.

*Written to the [nine-point writing standard](criteria.md)
shared by every guide in this series.*

## A note on verification

Exactly what was executed during writing, and what was
labeled instead of executed:

**Executed and validated:**

- **Subnet math** — every network/broadcast/host-count claim
  checked with `python3 -c "import ipaddress; ..."`:
  `192.168.1.0/24`, `192.168.2.0/24`, `192.168.3.0/24` each
  yield network `.0`, broadcast `.255`, 254 usable hosts
  (`.1`–`.254`); the DHCP pool `192.168.1.100`–`192.168.1.199`
  is 100 addresses by arithmetic.
- **Caddyfile** — validated with the official Caddy binary
  (v2.8.4): `caddy validate --adapter caddyfile --config
  Caddyfile` exited 0 (config adapted cleanly, no errors).
- **nginx server block** — validated with nginx 1.22.1
  (extracted from the Debian `.deb`, not installed):
  `nginx -t` on a minimal config wrapping the chapter's
  server block reported \"syntax is ok\" and \"test is
  successful.\"
- **dnsmasq snippet** — validated with dnsmasq 2.90
  (extracted from the Debian `.deb`): `dnsmasq --test`
  reported \"syntax check OK.\"
- **RFC 6598** — verified to exist and to reserve
  `100.64.0.0/10` as shared address space for CGNAT (April
  2012, Best Current Practice), checked against the published
  RFC text.
- **Docs build** — `mkdocs build --strict` passed after
  replicating the deploy workflow's staging step locally
  (`mkdir -p docs && cp *.md docs/ && rm -f docs/README.md
  && cp README.md docs/index.md`).

**Honestly labeled, not executed here:**

- Router web-UI steps are described as **typical** — every
  vendor's menus differ, and no router was available in this
  environment.
- Shell command outputs (`ipconfig`, `getent hosts`, `curl`
  checks) are marked as **examples / typical output** — your
  addresses will differ.
- Pi-hole and AdGuard Home install procedures are **from the
  projects' docs** — neither was installed here.
- Cross-guide links use absolute URLs to the published series
  sites; the target chapters were confirmed to exist in the
  sibling repos' `mkdocs.yml` navs, but the live published
  pages were not fetched.
- No speed, range, or performance numbers are claimed
  anywhere in the guide — Wi-Fi advice is qualitative
  (placement, bands, channels) by design.
