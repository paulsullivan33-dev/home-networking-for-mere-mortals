# Port Forwarding

**The one-sentence version:** Port forwarding punches a hole
through your router's firewall so the internet can reach one
service inside — it's the obvious answer to remote access, and
usually the wrong one. Understand it, then prefer the safer
alternatives.

## What it is

Recall the hotel switchboard from
[What Your Router Does](what-your-router-does.md): inbound
packets with no table entry get dropped, because the router
doesn't know which room they're for. A **port-forwarding rule**
is a standing instruction: \"anything arriving on *my public
address*, port X, goes to *this inside machine*, port Y —
no questions asked.\"

Example: you run a web service on `192.168.1.10:8000` and add
this rule:

| Router setting | Value |
|---|---|
| External port | 8000 |
| Internal address | 192.168.1.10 |
| Internal port | 8000 |
| Protocol | TCP |

Now `http://<your-public-ip>:8000` from anywhere on the
internet reaches your service. The bouncer has been told to wave
through everyone asking for that door.

**How to think about it:** every forwarded port is a door you
installed in your firewall and left unlocked for the whole
internet to try. The service behind it is now exposed to every
bot on earth that scans for open ports — and they scan
*constantly*, within minutes of the port opening.

## Setting one up (typical router UI)

Router menus vary; the shape is always the same:

1. Give the inside machine a **DHCP reservation** first
   ([DHCP and Reservations](dhcp-and-reservations.md)) — a
   forwarding rule pointing at an address that later changes
   points at a stranger's device.
2. Find **Port Forwarding** / **Virtual Server** / **NAT** in
   the router settings.
3. Add the rule: external port, internal address, internal port,
   TCP or UDP (or both — most services say which in their
   docs).
4. Save. Test from *outside* your network (phone on cellular
   with Wi-Fi off) — testing from inside can lie to you (see
   [Double NAT and Gotchas](double-nat-and-gotchas.md)).

**Check:** from outside, the service answers; from a port-scan
perspective, only the ports you forwarded are open. Verify the
rule points at the right machine — a typo'd internal address
forwards the internet to the wrong device.

## Why to avoid it

This is the chapter's real point. Port forwarding works, but
each rule buys you permanent exposure:

- **The service must be hardened.** Whatever you expose will
  be probed by automated scanners within minutes, then
  attacked with password guessing and known exploits for as
  long as the port is open. \"It's just a small personal
  thing\" is not a defense — bots don't care.
- **You own the patching.** A forwarded service you forget to
  update is a forwarded vulnerability. Every exposed service
  is a maintenance commitment.
- **Some things should never be forwarded.** Remote-desktop
  protocols, databases, and anything with a default password
  are the classic breach stories. If the service wasn't
  designed to face the internet, don't make it.
- **It doesn't travel well.** Change ISPs, get put behind
  CGNAT, or add a second router, and the rule silently stops
  working (see [Double NAT and Gotchas](double-nat-and-gotchas.md)).

If you expose *anything*, read
[Exposing a Service Safely](https://paulsullivan33-dev.github.io/cybersecurity-for-mere-mortals/public-services/)
in the cybersecurity guide first — it's the companion piece to
this chapter: strong unique passwords, updates, and knowing
what \"probed within minutes\" really means.

## The better answers

For nearly every \"I want to reach my home stuff from away\"
need, one of these beats port forwarding. They're compared
properly in [Remote Access, Compared](remote-access-compared.md);
the short version:

| Instead of forwarding… | Consider |
|---|---|
| A port for *your own* remote access | **Tailscale** — your devices join a private mesh; nothing is exposed to the internet at all. The [Tailscale guide](https://paulsullivan33-dev.github.io/tailscale-for-mere-mortals/) teaches it end to end. |
| A port for a *service others use* | A **VPN** into your home, then reach the service as if you were on the LAN. |
| A port because an app demanded it | Check whether the app offers a **cloud relay** instead — many do, and the relay means no inbound hole. |

The rule of thumb: **forward ports for services you serve to
the public; use a private tunnel for everything that's just
for you.** Most home \"remote access\" needs are the second
kind wearing the first kind's clothes.

## What port forwarding can't fix

- It can't help if your ISP puts you behind **CGNAT** (your
  router's \"public\" address isn't public — the ISP's router
  would need the rule, and it won't take one from you).
- It can't reach a device on a network the rule doesn't point
  at (wrong subnet, guest network).
- It doesn't add encryption, authentication, or logging. It's
  a hole, not a service.

Forward with both eyes open, or don't forward at all.

**Next:** [Guest Networks and VLANs](guest-networks-and-vlans.md)
