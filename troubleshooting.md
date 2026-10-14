# Troubleshooting

**The one-sentence version:** Most home-network failures are
one of five stories — no address, wrong network, name doesn't
resolve, blocked path, or the router changed underneath you.
Match your symptom, work the steps in order, and check which
of the [router's four jobs](what-your-router-does.md) is at
fault before changing anything.

## How to use this chapter

Each symptom lists the *most likely cause first*. Do the
checks in order; stop when it's fixed. Resist changing three
settings at once — if it starts working you won't know which
change did it, and you'll do it again next time.

A useful first command on any misbehaving machine (typical
output — your addresses will differ):

```bash
# Linux / Mac
ip addr show 2>/dev/null | grep "inet " || ifconfig | grep "inet "
```

```text
inet 192.168.1.42/24 brd 192.168.1.255 scope global dynamic
```

No `inet` line on the active interface = the device has no
address at all → start with \"new device gets no IP.\"
Windows equivalent: `ipconfig` and look for \"IPv4 Address.\"

---

## Can't reach the printer

1. **Is the printer awake and on the network?** Printers sleep
   hard. Wake it, wait 30 seconds, try again.
2. **Did its address change?** Print the network config page
   from the printer's panel and compare with the address your
   computer is trying. If it moved → give it a
   [DHCP reservation](dhcp-and-reservations.md) so it stops.
3. **Try the address, not the name.** If `http://192.168.1.25`
   (printer's web page) loads but `printer.local` doesn't,
   it's a *name* problem ([Names on Your LAN](names-on-your-lan.md)),
   not a printer problem.
4. **Guest network?** If your computer is on the guest SSID,
   it *can't* see the printer — by design. Move to the main
   network.

## Works on Wi-Fi but not ethernet (or vice versa)

The device is on **two different networks** without you
realizing it — e.g., Wi-Fi joined the ISP gateway's network
while the cable goes to your router, or vice versa. Compare
the addresses: Wi-Fi `192.168.0.x` vs ethernet `192.168.1.x`
means two routers are both active (see
[Double NAT and Gotchas](double-nat-and-gotchas.md)). Fix: one
router does routing; the other goes to bridge mode, and every
cable and SSID hangs off the same one.

## New device gets no IP

1. **DHCP pool exhausted?** Check the router's DHCP client
   list. A pool of 100 addresses with 100 clients means the
   new device gets nothing. (Rare in homes — but a stuck pool
   full of long-dead leases happens.)
2. **DHCP server off?** If you run DHCP from a Pi or second
   box and it's down, nothing new gets an address. Devices
   with old leases keep working — the classic \"everything
   old works, nothing new connects.\"
3. **Wrong network cable / VLAN port.** A device plugged into
   a port assigned to an empty VLAN gets silence. Move it to
   a known-good port.
4. **As a test only:** assign a temporary static IP in the
   right range. If that works, DHCP specifically is the fault —
   don't leave the static in place; fix DHCP.

## Slow Wi-Fi in one room

1. **Confirm it's Wi-Fi, not the internet:** speed test next
   to the router vs in the bad room. Collapses with distance
   → it's the path (see [Wi-Fi That Matters](wifi-that-matters.md)).
2. **Check the band:** the device may be clinging to 2.4 GHz.
   Forget the network and rejoin near the router, or split
   SSIDs so you can choose 5 GHz manually.
3. **Look for the new interference:** the dead room started
   when the baby monitor / new TV / mirror went in? 2.4 GHz
   shares airspace with a lot of household gadgets.
4. **One room, always, everything tried:** that's construction
   (brick, concrete, plaster-and-lath). The fix is a wired
   access point on the near side of the wall, not a stronger
   router.

## "I replaced the router and everything broke"

The new router brought a new LAN subnet, new Wi-Fi name, no
reservations, and no port forwards. Work the migration list:

1. Set the new router's LAN to the **old subnet**
   (e.g. `192.168.1.0/24`) so every static IP and bookmark
   still points somewhere real.
2. Reuse the **old SSID and password** so devices rejoin
   silently.
3. Re-create **DHCP reservations** for servers/printers.
4. Re-create **port forwards** (or better, revisit whether
   you still need them — [Port Forwarding](port-forwarding.md)).
5. If the old router was in bridge-behind-ISP-box harmony,
   put the new one there too — check for
   [double NAT](double-nat-and-gotchas.md).

## Can't reach a home service from outside

1. **Test from outside properly:** phone on cellular, Wi-Fi
   *off*. Testing from inside your own LAN can succeed via
   the internal address and tell you nothing about the
   outside path.
2. **Is the service listening where you think?** From inside,
   `curl http://<server-ip>:<port>` should answer. If not,
   the service itself is down — no router setting fixes that.
3. **Port forward pointing at the right machine?** The
   server's address must match its current
   [reservation](dhcp-and-reservations.md). A stale rule
   forwards the internet to whatever took the old address.
4. **CGNAT?** If the router's WAN is in `100.64.0.0/10`,
   inbound IPv4 cannot work — stop debugging the rule and
   read [Double NAT and Gotchas](double-nat-and-gotchas.md).
5. **ISP blocking?** Some ISPs block inbound port 80/443 (and
   sometimes 25) on residential plans. The rule is fine; the
   ISP drops it before it arrives. A non-standard external
   port, or a tunnel/mesh instead, sidesteps this.

## A name resolves to the wrong address

1. **Flush the cache:** the old answer is cached somewhere —
   device, browser, or router. `ipconfig /flushdns`
   (Windows); `sudo dscacheutil -flushcache` (Mac);
   restart the browser. Then check the TTL expired.
2. **Which DNS is the device using?** `ipconfig /all`
   (Windows) / check the network details. If it's not your
   local DNS, the [local record](names-on-your-lan.md) never
   gets asked. Fix the DHCP-advertised DNS server.
3. **Split-horizon confusion:** the name resolves to the
   *public* address while you're inside. That works only if
   hairpin NAT works on your router; the inside answer (see
   [DNS at Home](dns-at-home.md)) is the robust fix.

## The internet is down (whole house)

1. **One device or all?** One → it's that device (reboot it,
   check its Wi-Fi). All → it's the router or the ISP.
2. **Read the router's lights / status page.** No WAN
   address, or a `169.254.x.x` address on the WAN → the
   router isn't getting anything from the ISP. (`169.254.x.x`
   is the \"I asked for an address and nobody answered\"
   range.)
3. **Power-cycle in order:** ISP modem/gateway off 60
   seconds → on, wait for its lights to settle → router off
   30 seconds → on. Order matters: the router can't get an
   address from a modem that's still booting.
4. **Still down?** Plug a laptop directly into the modem (if
   separate). Works → your router is the fault. Doesn't →
   call the ISP; it's their side.

---

## When to stop and ask

If you've worked a symptom's list twice with no change, stop
— you're now in \"change random settings\" territory, which
creates second problems. Write down what's actually happening
(addresses, what's reachable, what's not, what changed
recently) and bring that note to whoever helps you. \"The
internet is broken\" gets shrugs; \"laptop is 192.168.1.42,
can ping the router, can't resolve names since Tuesday\" gets
answers.

**Next:** [Glossary](glossary.md)
