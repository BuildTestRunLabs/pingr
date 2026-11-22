# Reading pingr results

A guide for people new to ping and path tracing: what the numbers mean, what is
normal, and when something is actually wrong. Examples assume a home connection
in Australia, but the reasoning applies anywhere.

## The one rule

**Only the last hop matters for loss.** If the destination shows 0% loss, your
traffic is getting through, no matter how bad the hops in the middle look.
Loss is only real when it starts at some hop and *carries through to every hop
after it, including the destination*.

## Why some hops show 100% loss or nothing at all

Tracing works by sending probes that are allowed to cross 1 router, then 2,
then 3, and so on. The router where a probe runs out is supposed to send back
an ICMP "time exceeded" message, which is how pingr learns that hop's address
and timing. A blank hop (`hop 4`, `100%`, `-`) is a router that forwarded your
traffic perfectly well but did not send that message, or sent one that was
dropped on the way back.

Common reasons:

- **Rate limiting and priority.** Routers forward traffic in hardware at full
  speed, but building an ICMP reply needs the router's much slower CPU. Vendors
  cap that by default and ISPs often turn it down further or off. Trace replies
  are the lowest priority job a router has.
- **Hiding the network.** Many ISPs filter these replies from their core
  routers so outsiders cannot map their internal layout or target the routers.
- **MPLS and tunnels.** ISP backbones often carry traffic in label-switched
  tunnels. Routers inside the tunnel may not answer, or answer with odd timings.
- **Private addresses.** Some routers answer from private or carrier-grade NAT
  addresses (`100.64.0.0/10`), and filters on the way back drop those replies.

The same rate limiting explains a hop that shows *some* loss (say 20 to 30%)
while every hop after it is clean: the router is answering as many probes as
its limit allows. Home routers do this too, so even hop 1 can show loss.

## A worked example

A household in Sydney on an NBN fibre (FTTP) plan, wired to the router, runs
`pingr -t 1.1.1.1`:

| Hop | Host | Loss | Avg | What it is |
|---|---|---|---|---|
| 1 | 192.168.1.1 | 25% | 1ms | Home router. The loss is rate limiting: hops after it are clean |
| 2 | hop 2 | 100% | - | ISP access network (NBN handover to the ISP), not answering |
| 3 | 203.0.113.10 | 0% | 4ms | ISP router that does answer |
| 4 to 6 | hop 4 to 6 | 100% | - | ISP core, filtered |
| 7 | 108.162.x.x | 0% | 5ms | Cloudflare edge in Sydney |
| 8 | 1.1.1.1 | 0% | 5ms | Destination |

Verdict: healthy. The destination has 0% loss and a low, steady latency. The
blank and lossy middle hops are routers declining to answer, nothing more.

What a real problem looks like, from the same house:

| Hop | Host | Loss | Avg |
|---|---|---|---|
| 1 | 192.168.1.1 | 0% | 1ms |
| 2 | hop 2 | 100% | - |
| 3 | 203.0.113.10 | 12% | 38ms |
| 4 | 203.0.113.25 | 11% | 41ms |
| 7 | 108.162.x.x | 12% | 44ms |
| 8 | 1.1.1.1 | 12% | 45ms |

Loss and latency both jump at hop 3 and stay there all the way to the
destination. Something at or just before hop 3 (the NBN connection or the
ISP's first router) is congested or faulty. This is the kind of output worth
sending to your ISP's support team.

### Typical Australian latencies

Rough round-trip times from a wired NBN fibre or HFC connection. FTTN/FTTC
copper adds a few milliseconds; satellite and fixed wireless are covered
further down.

| Destination | Typical round trip |
|---|---|
| Your router | under 1ms wired, 2 to 10ms over Wi-Fi |
| ISP's first router | 3 to 10ms |
| CDN in your capital city (Cloudflare, Google, Akamai) | 3 to 15ms |
| Another Australian capital (Sydney to Melbourne or Brisbane) | 10 to 25ms |
| Sydney to Perth | 45 to 60ms |
| New Zealand | 25 to 40ms |
| Singapore | 90 to 120ms |
| US west coast | 140 to 170ms |
| US east coast | 200 to 240ms |
| Europe | 260 to 320ms |

Overseas numbers are mostly the speed of light in undersea cable. No plan or
router upgrade will make Sydney to London much faster than about 260ms.

## How much is too much

Thresholds for the destination's results. Jitter is how much latency changes
from one reply to the next (pingr shows it in the stats bar).

| Metric | Good | Noticeable | Poor |
|---|---|---|---|
| Latency (to an Australian server) | under 30ms | 30 to 80ms | over 80ms |
| Jitter | under 5ms | 5 to 20ms | over 20ms |
| Loss | 0% | 0.5 to 2% | over 2% |

How the same numbers affect different traffic:

| Traffic | Latency | Jitter | Loss | What you notice |
|---|---|---|---|---|
| Web browsing | fine to about 150ms | rarely matters | 1 to 2% causes slow loads | Pages hang before loading |
| Streaming video (Netflix, YouTube) | barely matters | barely matters | buffering hides up to a few % | Lower quality or buffering when loss is sustained |
| Video and voice calls (Teams, Zoom, FaceTime) | under 150ms is fine | over 30ms causes choppy audio | over 1% causes robotic voice and frozen video | Talking over each other, dropouts |
| Online gaming | under 50ms competitive, under 100ms casual | over 10ms feels inconsistent | any loss causes rubber-banding | Lag, teleporting players |
| Remote desktop and SSH | under 100ms feels local | over 20ms makes typing feel uneven | over 1% causes stalls | Delayed keystrokes, frozen screen |
| Large downloads and backups | barely matters | barely matters | over 1% can noticeably cut speed | Slow transfers |

Notes:

- **Loss hurts more than latency.** A steady 60ms is better for almost
  everything than 20ms with 3% loss.
- **Judge over minutes, not seconds.** A single lost reply is normal. Let pingr
  run for a few minutes (or `--rounds 300`) before deciding anything.
- **Test at the right time.** Congestion usually shows up in the evening peak,
  roughly 7pm to 11pm. A test at 2pm can look perfect while the problem is real.

## Wired versus wireless

Where you test from changes what you see, especially in the first hop.

### Ethernet

- Router latency under 1ms and jitter close to 0.
- Any loss or jitter is almost certainly beyond your house.
- Best way to test your internet connection itself. If you are chasing a
  problem, plug in a cable first so Wi-Fi is ruled out.

### Wi-Fi

- Router latency of 2 to 10ms is normal, with regular spikes to 30 to 100ms or
  more. Jitter is higher everywhere along the path because Wi-Fi adds it at
  hop 1.
- Spikes come from devices taking turns on a shared channel, neighbouring
  networks on the same channel (very common in apartments), background scanning,
  and power saving on phones and laptops (devices doze between packets, so the
  first reply after a quiet period is slow).
- 5 GHz and 6 GHz are usually steadier than 2.4 GHz but reach less far through
  walls.
- Mesh systems add latency for each wireless hop between nodes; a node with a
  cabled (Ethernet) backhaul behaves close to the main router.
- Quick check: if hop 1 is jumpy, the problem is your Wi-Fi, not your ISP. Run
  pingr against your router alone (`pingr 192.168.1.1`, or whatever hop 1 shows)
  and watch for spikes.

### Mobile and fixed wireless

| Connection | Typical latency to a capital city server | Notes |
|---|---|---|
| 4G | 30 to 60ms | Varies with signal and tower load |
| 5G | 15 to 40ms | Falls back to 4G when signal is weak |
| NBN Fixed Wireless | 20 to 50ms | Evening congestion common in busy areas |
| Low Earth orbit satellite (e.g. Starlink) | 25 to 60ms | Brief loss or spikes every few minutes as satellites hand over; this is normal |
| NBN Sky Muster (geostationary satellite) | 600ms or more | Physics: the satellite is about 36,000km up. Fine for browsing and streaming, hard for calls and games |

On all of these, the first few hops are usually inside the carrier's network
and often stay blank, and jitter is higher than on fixed lines.

## Quick checklist

1. Look at the **destination's** loss first. 0% means traffic is getting
   through.
2. If there is loss, find the **first hop where it starts and stays**. That is
   where the problem is.
3. Ignore blank or lossy hops in the middle when later hops are clean.
4. Check **hop 1**: jumpy on Wi-Fi means fix the Wi-Fi before blaming the ISP.
5. Compare to the **latency table** for where the server is. High latency to
   Europe is normal; high latency to Sydney from Sydney is not.
6. Test during the **evening peak** and for **a few minutes** before drawing
   conclusions.
