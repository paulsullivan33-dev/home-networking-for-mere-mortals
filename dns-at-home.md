# DNS at Home

**The one-sentence version:** Your router answers name lookups
by forwarding them upstream and caching the answers; you can
take over that job with a **Pi-hole** or **AdGuard Home** to
block ads/trackers house-wide, and you can teach your DNS
different answers *inside* vs *outside* your network.

**Two-sentence refresher:** DNS is the phone book that turns
names into addresses.
[The fundamentals guide](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/dns/)
covers how a lookup travels from your computer to the root
servers. This chapter is about the lookup step you control: the
one that happens inside your house.

## What the router's DNS forwarder does

Recall [Job 4](what-your-router-does.md): your devices ask the
router for names, and the router asks upstream (usually your
ISP's DNS servers, or `1.1.1.1` / `8.8.8.8` if you changed the
setting) and caches the answer for the TTL (time to live) the
answer carries. One cache serves the whole house, so the second
device to ask for the same name gets an instant answer.

The knob that matters: most routers let you set the **upstream
DNS servers** in one place, and every device using DHCP picks
it up. Reasons to change it:

| Change upstream DNS to… | Why people do it |
|---|---|
| Your ISP's default (do nothing) | It works; the ISP sees every lookup |
| `1.1.1.1` / `1.0.0.1` (Cloudflare) or `8.8.8.8` / `8.8.4.4` (Google) | Often faster, and moves your lookup history from the ISP to that provider |
| A filtering DNS service | Blocks known-bad domains at lookup time |

None of these encrypt the lookup by default — plain DNS is
readable on the wire. **DNS-over-HTTPS** (DoH) and
**DNS-over-TLS** (DoT) encrypt it; some routers and most
browsers support them now, but they're a privacy upgrade, not a
security requirement for a normal home. Say it plainly: changing
upstream DNS changes *who* can see your lookups, not whether the
sites you visit are safe.

## Pi-hole and AdGuard Home: the concept

A **Pi-hole** (or **AdGuard Home** — same idea, different
project) is a small DNS server you run on your own network,
typically on a Raspberry Pi or any always-on computer. It sits
*between* your devices and the upstream DNS and does one extra
thing: it checks every requested name against **blocklists** —
published lists of ad, tracker, and malware domains — and
answers blocked names with \"that doesn't exist.\"

The ad never loads because the lookup fails. The browser shows a
blank box where the ad would be; the \"smart\" TV can't phone
home to its telemetry domain. One box, whole-house blocking,
including apps and devices that ignore browser extensions.

What to know before installing one:

- **It must be always on.** If the Pi is off, nobody's DNS
  works. Keep the router's upstream DNS as a *secondary* on
  devices? No — a secondary DNS gets used unpredictably, which
  silently bypasses the blocking. Better: keep a spare Pi, or
  accept the single point of failure.
- **It needs to be your network's DNS.** Either set the
  Pi-hole's address as the DNS server in the router's DHCP
  settings (best — every device gets it automatically), or set
  it per-device.
- **Blocklists need judgment.** Aggressive lists break things
  (banking apps, smart-home setup flows). When something stops
  working, the Pi-hole/AdGuard query log shows exactly which
  domain was blocked — unblock it (*whitelist* it) and move on.
- **It doesn't block everything.** YouTube ads, for example,
  come from the same domains as the videos — no DNS blocker can
  separate them. It's a strong filter, not a force field.
- **Install steps are per-project.** Both projects document
  their own installers well; this guide won't re-teach them
  (installation wasn't verified in this environment — follow
  the projects' docs).

## Split horizon: one name, two answers

**Split-horizon DNS** sounds fancy; the idea is ordinary. You
want `files.home.arpa` to mean your NAS. From *inside* your
house, it should resolve to the NAS's private address
(`192.168.1.10`). From the *internet*, that name shouldn't
resolve to anything (or should resolve to your public address,
if you've set up outside access).

So your local DNS gives **different answers depending on who's
asking**:

| Who asks | Answer for `files.home.arpa` |
|---|---|
| A device on your LAN | `192.168.1.10` (direct, fast, private) |
| Anyone on the internet | Nothing (or your public IP, if you expose it) |

Why bother? Two real payoffs:

1. **No hairpin weirdness.** Without it, your phone on home
   Wi-Fi asking for your public address has to go out to the
   internet and back in — which breaks on some routers (see
   [Double NAT and Gotchas](double-nat-and-gotchas.md)). The
   inside answer skips the trip entirely.
2. **TLS certificates keep working.** If you have a proper
   certificate for the name (see
   [Reverse Proxies](reverse-proxies.md)), it validates the
   same whether you're home or away.

How you implement it depends on your DNS: a Pi-hole/AdGuard
\"local DNS record,\" a dnsmasq `address=/` line (like the one
validated in [DHCP and Reservations](dhcp-and-reservations.md)),
or your router's static-DNS feature. The mechanism differs; the
idea is identical: *inside names resolve inside.*

**A note on suffixes:** use `.home.arpa` for your private
names — it's the suffix reserved for exactly this purpose.
Don't invent something like `.lan` or use a real public domain
you don't own; the former can collide with a future real domain,
the latter breaks real lookups.

**Check:** from a device on your LAN, look up your local name
and confirm you get the private address:

```bash
getent hosts files.home.arpa
```

```text
192.168.1.10      files.home.arpa
```

(Example output — yours will show your own addresses. `getent`
is Linux; on Windows use `nslookup files.home.arpa`, on a Mac
`dscacheutil -q host -a name files.home.arpa`.)

## What this chapter doesn't do

It doesn't pick your upstream DNS provider for you, and it
doesn't install Pi-hole — those are per-project docs. What you
have now is the model: the forwarder, the filter, and the
inside-vs-outside answer. With that, the next chapter — reaching
things *by name* — clicks into place.

**Next:** [Names on Your LAN](names-on-your-lan.md)
