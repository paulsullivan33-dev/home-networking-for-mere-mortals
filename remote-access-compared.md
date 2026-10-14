# Remote Access, Compared

**The one-sentence version:** Four ways to reach home from
away — port forwarding, VPN, Tailscale-style mesh, cloud relay
— compared honestly on what each costs you in risk, effort,
and fragility. For \"just me reaching my stuff,\" the mesh
wins; port forwarding is for serving the public.

## The four options

| | Port forwarding | VPN into home | Tailscale / mesh | Cloud relay |
|---|---|---|---|---|
| **What it is** | Router hole → your service, open to the internet | Encrypted tunnel to your home network; then you're \"on the LAN\" | Your devices form a private network over the internet, no open ports | The app vendor's server brokers the connection (or proxies it) |
| **Who can reach it** | Everyone on earth | Only devices with your VPN credentials | Only your logged-in devices | Anyone the vendor's sharing link allows |
| **Setup effort** | Low — one router rule | Medium — run a VPN server (often on the router), configure clients, manage keys/certs | Low — install, log in, done | Lowest — usually a toggle in the app |
| **Ongoing effort** | High — you own patching a public-facing service forever | Low-medium — keep the VPN server updated | Low — updates like any app | None — vendor's problem |
| **Risk if misconfigured** | Internet-facing breach | VPN credential leak exposes LAN | Account compromise exposes your devices | Vendor breach / vendor sees your traffic |
| **Survives CGNAT?** | No | Only with help (see below) | Yes | Yes |
| **Survives ISP/router change?** | Rule must be rebuilt | Server must be re-reachable | Yes — nothing to rebuild | Yes |
| **Speed** | Direct, full speed | Direct, full speed (your upload is the limit) | Direct when possible, relayed when not | Often relayed — slower, depends on vendor |

## Reading the table

**Port forwarding** is covered — with its warnings — in
[Port Forwarding](port-forwarding.md). It's the right tool
when you're *serving the public* (a website, a game server for
friends). It's the wrong tool for \"I want my files from the
hotel,\" because it publishes your private service to every
scanner on earth to solve a problem only *you* have.

**VPN into home** (WireGuard or OpenVPN on your router or a
home server) is the classic answer: one encrypted tunnel, and
your phone behaves as if it's on the couch. Strong and
private — no vendor in the middle. The costs are real though:
you run the server, you manage the keys, and if your home IP
changes or you're behind CGNAT, *you* fix the reachability.
The [Tailscale guide's VPN chapter](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/what-is-a-vpn/)
explains the tunnel concept if it's new.

**Tailscale / mesh** (Tailscale, NetBird, and similar) keeps
the VPN's privacy model — encrypted, your devices only — and
deletes the server-running: a coordination service handles
NAT traversal, so connections punch out through whatever
network you're on, CGNAT included. The trade: you're trusting
the vendor's coordination and identity service. For \"my
devices, reaching each other,\" this is the best effort-to-
safety ratio available, which is why this series has a whole
[guide](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/)
about it.

**Cloud relay** (Plex remote access, vendor camera apps, etc.)
is what most consumer apps actually do: your device dials
*out* to the vendor, your phone dials out to the vendor, the
vendor introduces them. Zero router configuration, works
behind CGNAT, and the ongoing effort is nil. The trade is
trust and control: the vendor can see metadata (sometimes
content), can change terms, can be breached, can discontinue
the product. Fine for a camera; think twice for anything
sensitive.

## The decision, as a flowchart in words

1. **Is it only for you (and your household)?** → Mesh
   (Tailscale-style) or VPN. Don't publish it to the world.
2. **Is it for the public or a rotating cast of other people?**
   → Port forwarding (or a hosted VPS), hardened per the
   [cybersecurity guide's exposure chapter](https://paulsullivan33-dev.github.io/cybersecurity-for-mere-mortals/public-services/).
3. **Does the app offer a relay toggle and the data isn't
   sensitive?** → Use the relay; save your effort for things
   that matter.
4. **Behind CGNAT?** → Cross port forwarding and classic VPN
   off the list; mesh or relay remain.

## Two honest footnotes

- **\"My upload speed is the limit.\"** Home internet is
  asymmetric: fast down, slow up. Reaching *into* your home
  uses your *upload* — every option above is capped by it.
  Pulling a big file from home over a 10 Mbps upload takes a
  while no matter how clever the tunnel.
- **Nobody sells \"secure.\"** Each option moves the trust
  somewhere: to your patching discipline (port forwarding),
  to your key management (VPN), to a vendor's infrastructure
  (mesh, relay). \"Which do I trust, and what happens when
  that trust breaks?\" is the whole question. Anyone selling
  one option as trust-free is selling something.

**Next:** [Double NAT and Gotchas](double-nat-and-gotchas.md)
