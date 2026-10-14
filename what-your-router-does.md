# What Your Router Does

**The one-sentence version:** Your router is one box doing four
jobs — translating addresses (NAT), handing out addresses
(DHCP), blocking uninvited traffic (firewall), and passing along
name lookups (DNS). Everything in this guide hangs on this
mental model.

## The picture to keep in your head

Most homes have a chain that looks like this:

```
Your devices          Your router              The internet
(phone, laptop, TV,   one public address       (billions of
printer, server)      from your ISP            devices)
        \                 |                     /
         \_____ LAN _____/_____________________/
         192.168.1.x
```

Inside your house, devices talk to each other on **private
addresses** — usually in the `192.168.1.0/24` range (more on what
that means in
[Subnetting in 10 Minutes](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/subnetting/)).
Your ISP gives your router *one* public address (or sometimes a
few) that the whole house shares. The router sits at the
boundary, translating between the two worlds. That's the job
everything else follows from.

## Job 1: NAT — the translator

**NAT** (Network Address Translation) lets many devices share
one public address. When your laptop visits a website, the router
rewrites the outgoing packet's source address from the laptop's
private address (`192.168.1.5`) to the router's public address,
and writes the change down in a table. When the reply comes
back, the router consults the table and rewrites the address
back, handing the reply to the laptop.

Think of it like a hotel front desk: guests (your devices) place
calls through the switchboard, and the switchboard routes
incoming calls to the right room by keeping track of who called
whom. Callers outside know only the hotel's number.

Two things follow from this:

- **Outbound is easy.** A device inside can reach out to
  anything, because the router creates a table entry for the
  connection and knows where replies go.
- **Inbound is hard.** Someone outside trying to reach your
  server cold has no table entry, so the router drops the
  packet — it has no idea which room the call is for. Reaching
  a home service from the internet requires an explicit rule
  (see [Port Forwarding](port-forwarding.md)) or a tunnel that
  dials *out* first (see [Remote Access, Compared](remote-access-compared.md)).

## Job 2: DHCP — the front desk clerk

**DHCP** (Dynamic Host Configuration Protocol) hands out network
settings to devices when they join. Plug in a laptop and it
asks: \"who am I? what network am I on? where's the gateway?
where's DNS?\" The router's DHCP server answers — usually with
an address from a pool like `192.168.1.100` through
`192.168.1.199` — and the address is a **lease**: borrowed, not
owned, typically for a day or so, renewed automatically.

That's why your printer's address \"changes sometimes\": the
lease expired while it was asleep and something else took its
old number. The fix is a **reservation** (covered next chapter),
not anger at the printer.

| Router setting | What it means |
|---|---|
| DHCP pool / range | The addresses the router may hand out, e.g. `192.168.1.100`–`192.168.1.199` (100 addresses) |
| Lease time | How long a device keeps its address before renewing, e.g. 24 hours |
| Gateway | The router's own LAN address, e.g. `192.168.1.1` — devices send non-local traffic here |
| DNS server | Usually the router's own address too — see Job 4 |

## Job 3: Firewall — the bouncer

The router's **firewall** is a simple rule by default: *outbound
connections are fine; unsolicited inbound connections are
dropped.* Your laptop can load a web page; a stranger on the
internet cannot start a connection to your laptop. This default
is the single most valuable security feature in your house, and
it works because of NAT's table: no table entry, no entry.

The firewall also *does* let in replies to connections you
started (that's how web pages load) and traffic to ports you
explicitly forward. The dangerous settings live here too —
UPnP, DMZ, port forwarding — all of which poke holes in the
bouncer's rule. They get their own chapters and warnings
([Port Forwarding](port-forwarding.md),
[Double NAT and Gotchas](double-nat-and-gotchas.md)).

## Job 4: DNS forwarder — the directory desk

Your devices need name lookups (`printer.local`, `example.com`),
but they don't do the full recursive lookup themselves.
Instead, they ask the router, and the router's **DNS forwarder**
passes the question up to your ISP's (or your configured) DNS
servers and caches the answer. One level of indirection, which
gives you two powers: you can change the upstream servers in
*one* place for the whole house, and you can add local names the
router answers itself (see [DNS at Home](dns-at-home.md) and
[Names on Your LAN](names-on-your-lan.md)).

## Why this model matters

Nearly every home-networking confusion is one of the four jobs
misbehaving:

| Symptom | Likely job at fault |
|---|---|
| New device gets no address | DHCP (pool exhausted, server off) |
| Server reachable inside, invisible outside | NAT (no inbound path) or firewall (rule missing) |
| \"I changed the router and everything broke\" | All four — new DHCP ranges, new DNS, new firewall rules |
| Name works on one device, not another | DNS forwarder / local names |

When something breaks, ask which job the symptom belongs to,
then open the router's settings page for *that* job. The rest
of this guide is about using each job on purpose instead of
fighting it.

**Next:** [DHCP and Reservations](dhcp-and-reservations.md)
