# Wi-Fi That Matters

**The one-sentence version:** Wi-Fi problems are almost always
about *physics* (distance, walls, interference) or *crowding*
(too many networks on the same channel) — not about settings
tweaks. Fix placement first, channels second, and buy new gear
last.

**Two-sentence refresher:** Wi-Fi is your devices talking to
the router over radio instead of cable; SSIDs, bands, and
passwords are the basics, covered in the
[fundamentals guide](https://paulsullivan33-dev.github.io/networking-for-mere-mortals/wifi/).
This chapter is the practical sequel: making it actually work
well.

## The bands, in practical terms

Your router probably broadcasts on up to three bands. They're
not \"good, better, best\" — they're trade-offs:

| Band | Range through walls | Speed potential | Crowding |
|---|---|---|---|
| **2.4 GHz** | Best — goes through walls and floors | Lowest of the three | Worst — everything uses it: neighbors, microwaves, baby monitors, Bluetooth |
| **5 GHz** | Decent — one or two walls OK | High | Better — more room, shorter range means less neighbor overlap |
| **6 GHz** | Similar to 5 GHz, slightly worse through walls | Highest (on new gear) | Emptiest — but only new devices have 6 GHz radios |

The practical rule: **let 2.4 GHz carry the far-away,
low-demand stuff** (smart plugs, the thermostat, the garage
camera) and **put your laptop and TV on 5 GHz** (or 6 GHz if
everything supports it). Most routers do this automatically
with one SSID and \"band steering\" — pushing each device to
the best band it supports. Separate SSIDs per band (`Home`
and `Home-5G`) give you manual control; one SSID is less to
manage. Either is fine.

**When to care:** a device keeps dropping, or speed is fine
next to the router and terrible one room over. **When not to
care:** everything works. Don't optimize a network that isn't
broken.

## Placement beats settings

Radio doesn't go through everything equally. The list that
matters, in order of how much they hurt:

1. **Distance** — signal fades with distance, full stop.
2. **Walls** — especially brick, concrete, plaster with metal
   lath, and mirrors. Drywall is nearly transparent by
   comparison.
3. **Metal** — filing cabinets, appliances, the router stuffed
   *behind* the TV. The router's antennas radiate outward;
   burying the box buries the signal.
4. **The floor** — routers on the floor in a closet serve the
   closet. Waist height or higher, out in the open, roughly
   central in the house.

**The one-room test:** stand next to the router with your
phone and run a speed test; walk to the bad room and run it
again. If it collapses, it's the *path*, not the internet —
moving the router (or adding an access point halfway) is the
fix. No settings page solves a wall.

## Channels: sharing the air politely

Wi-Fi bands are sliced into **channels**. Nearby networks on
the same channel take turns talking — yours waits for the
neighbor's, throughput drops. On **2.4 GHz**, only channels
**1, 6, and 11** don't overlap each other; everything else
partially overlaps *two* of them and is worse than just
sharing. On **5 GHz** there are many non-overlapping channels,
so crowding is rarer.

What to actually do:

- Leave channel selection on **Auto** unless you have a
  measured problem. Modern routers scan and pick reasonably.
- If 2.4 GHz is slow and a Wi-Fi analyzer app shows neighbors
  piled on your channel, try manually setting **1, 6, or 11**
  — whichever is least used around you. (Analyzer apps are
  free on every phone OS; they show a graph of nearby
  networks per channel.)
- Don't chase \"the empty channel\" on 2.4 GHz between the
  big three — overlapping two busy channels is worse than
  sharing one.

**A note on \"Wi-Fi 6/6E/7\":** these are generations of the
standard (6E adds the 6 GHz band; 7 is the newest). A new
standard helps most in *crowded* airspace and with many
devices; it doesn't punch through your brick wall any better.
Upgrade the standard when you have dozens of devices fighting,
not when one room is dead — that's a placement problem.

## Wired beats wireless, every time

The unglamorous truth behind every \"my Wi-Fi is slow\"
story: **anything that doesn't move should be on a cable.**
TV, desktop PC, game console, the NAS — a $15 cable gives each
of them full speed, zero interference, and *removes them from
the air*, leaving more room for the phones and laptops that
actually need wireless. Powerline adapters (networking over
your house wiring) are a distant second — they work, but
speeds vary wildly with your wiring. Mesh systems (multiple
units repeating the signal) fix dead rooms at the cost of
halving-ish throughput per hop; they're a placement solution
wearing a product box.

## The 80/20 checklist

In order, stop when it's good enough:

1. Router out in the open, central, off the floor.
2. Stationary devices on ethernet.
3. Phones/laptops on 5 GHz; IoT on 2.4 GHz.
4. 2.4 GHz channel set to the least-crowded of 1, 6, 11 —
   only if measured slow.
5. Dead room persists → access point or mesh node halfway
   between, wired back to the router if at all possible.

If you've done all five and one room is still bad, that's a
wiring-or-construction problem, not a Wi-Fi problem — and no
router upgrade fixes construction.

**Next:** [Remote Access, Compared](remote-access-compared.md)
