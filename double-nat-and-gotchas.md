# Double NAT and Gotchas

**The one-sentence version:** NAT works fine once; stack two
of them (your router behind your ISP's router, or your ISP
doing carrier-grade NAT) and inbound connections, port
forwards, and some games and calls break in confusing ways.
UPnP is a separate gotcha: convenience that lets any program
punch its own holes.

## Double NAT: two translators in a row

**Double NAT** means your traffic gets translated twice: once
by *your* router, once by another NAT box upstream — usually
your ISP's gateway/router combo unit. The chain:

```
Your devices → your router (NAT #1) → ISP gateway (NAT #2) → internet
   192.168.1.x      192.168.0.x (its WAN)      public address
```

How you get here: the ISP installs a modem+router combo, and
you plug *your* router into it instead of putting the combo in
**bridge mode** (\"just be a modem\"). Now you have two DHCP
servers, two firewalls, two NAT tables — and two admin pages
to check when anything breaks.

**What breaks:**

- **Port forwarding gets twice as hard.** A rule on *your*
  router isn't enough — the ISP gateway also needs a rule
  pointing at your router, and its idea of \"your router\"
  is a private address (`192.168.0.x`) that can change.
- **Some games, voice/video calls, and peer-to-peer apps**
  misbehave: they can usually still connect *out*, but
  incoming peer connections and some NAT-traversal tricks
  fail through two layers.
- **Confusing admin pages.** `192.168.1.1` is your router;
  `192.168.0.1` (or `10.0.0.1`) is the ISP box. Devices
  sometimes land on the wrong one (plugged into the ISP box
  directly), ending up on a different network from everything
  else.

**The fixes, in order of preference:**

1. **Bridge mode (best).** Put the ISP gateway in bridge /
   passthrough mode so it's just a modem; your router gets
   the public address and does the one NAT. One translator,
   one admin page.
2. **Use the ISP box as the only router.** If its Wi-Fi and
   features suffice, remove yours. One translator.
3. **DMZ / IP-passthrough to your router (acceptable).** Tell
   the ISP box to pass everything through to your router's
   WAN address. Still technically double NAT, but your router
   is the only one making decisions.

**Check for double NAT:** compare the WAN/internet address
shown on *your* router's status page with your public address
(search \"what is my ip\" on any device). If the router's WAN
is a *private* address (`10.x.x.x`, `172.16.x.x`–`172.31.x.x`,
`192.168.x.x`) while the search shows something else, there's
another NAT upstream.

## CGNAT: double NAT you can't fix

**CGNAT** (carrier-grade NAT) is double NAT where the second
translator lives *inside your ISP's network*, shared with
other customers. Your router's WAN address isn't a public
address at all — it's from the shared range **`100.64.0.0/10`**
(100.64.0.0 through 100.127.255.255, reserved for this purpose
by RFC 6598). The ISP translates you again onto a public
address shared with strangers.

**How to tell:** same check as above — if your router's WAN
address starts with `100.64` through `100.127`, you're behind
CGNAT. (A private-range WAN that *isn't* in that range usually
means an ISP box or second router on your premises — fixable
as above. The `100.64/10` range specifically means the ISP is
doing it.)

**What CGNAT breaks:** anything needing *inbound* IPv4 —
port forwarding (your router's \"public\" address isn't
public, so no rule you write can ever work), hosting game
servers, some peer-to-peer apps. Outbound browsing, streaming,
and anything that dials *out* (including Tailscale-style mesh
and cloud relays) works fine.

**Your options:** ask the ISP for a public IPv4 address (some
offer it free, some charge, some refuse); use IPv6 if both
ends support it (each device gets a real address — though you
still need firewall rules); or sidestep inbound entirely with
a mesh VPN or relay service (see
[Remote Access, Compared](remote-access-compared.md)). What
you *can't* do is configure around it from your side — the
missing mapping lives in the ISP's translator, not yours.

## UPnP: the automatic hole-puncher

**UPnP** (Universal Plug and Play, specifically its Internet
Gateway Device part) lets programs on your LAN *open their own
port-forwarding rules* without asking you. A game console
needs an inbound port? It asks the router, the router complies,
done. Convenient — and the reason many things \"just work.\"

The problem: **it can't tell your game from malware.** Any
program on any device — including a compromised IoT camera —
can punch arbitrary inbound holes, and the rules it creates
are invisible unless you go look. The cybersecurity guide's
[security basics](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/security-basics/)
flags it for the same reason.

The sane posture:

| Situation | UPnP setting |
|---|---|
| Everything works without it | **Off.** Fewer holes, fewer surprises. |
| A specific game/console needs it | On, *if* you trust every device on your LAN — or forward that one port manually and leave UPnP off. |
| IoT devices on the same LAN | Off, and consider segregating them ([Guest Networks and VLANs](guest-networks-and-vlans.md)). |

If you turn it off and something stops working, that's
*information*: you now know exactly which program wanted an
inbound port, and you can decide deliberately — manual forward
or leave it closed.

## Other classic gotchas

- **Two DHCP servers.** The ISP box *and* your router both
  handing out addresses on overlapping ranges → intermittent
  \"works, then doesn't\" chaos. Bridge mode or disabling one
  DHCP server fixes it.
- **Plugging into the wrong box.** A device cabled to the ISP
  gateway (when your real network is on your router) lands on
  the wrong subnet — it has internet but can't see your LAN.
  Every cable should ultimately hang off *one* router's LAN.
- **\"I replaced the router and everything broke.\"** New
  router, new DHCP range (`192.168.0.x` vs the old
  `192.168.1.x`), new Wi-Fi name, reservations gone, forwards
  gone. Migration checklist: same LAN subnet, same SSID and
  password, re-create reservations, re-create forwards, then
  test. (Full symptom → fix in [Troubleshooting](troubleshooting.md).)

**Next:** [Troubleshooting](troubleshooting.md)
