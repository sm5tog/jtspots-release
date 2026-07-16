# JTSpots — Quick Start

This gets a fresh installation receiving and forwarding spots in the
shortest path. For anything not covered here, see the full
[MANUAL.md](MANUAL.md).

## 1. Install

Download the latest zip from [Releases](../../releases/latest), unzip it
anywhere (e.g. `C:\JTSpots\`), and double-click `JTSpots.exe`. Nothing
else to install — no Python, no separate runtime.

## 2. Point WSJT-X, JTAlert, and Log4OM at the same multicast group

JTSpots listens for WSJT-X's FT8/FT4 decodes over UDP multicast. All three
programs need to agree on the same IP and port — JTSpots' own default
(Config → WSJT-X/QO-100) is `224.0.0.1` / `2237`.

**WSJT-X:**
1. File → Settings → Reporting.
2. Under "UDP Server": set the address to `224.0.0.1`, the port to `2237`.
3. Tick the checkbox that enables multicast (labelled "Enable multicast"
   or similar, depending on your WSJT-X version).
4. Make sure "Accept UDP requests" is ticked.
5. Leave "Outgoing interfaces" and "Multicast TTL" on their defaults
   unless you have a specific reason to change them.

**JTAlert:** usually needs nothing — it reads the same settings directly
from WSJT-X on startup. Restart JTAlert after changing WSJT-X's settings,
then double-check its own UDP Server fields match `224.0.0.1:2237` too.

**Log4OM:** requires **version 2.39.0.0 or later** for multicast (check
Help → About; upgrade first if older — earlier versions don't support it
at all). Under Settings → Configurations → Software → Connections, change
the inbound UDP connection that receives WSJT-X's `JT_MESSAGE` (port
`2237`) from `127.0.0.1` to `224.0.0.1`. From 2.39.0.0 onward, Log4OM
detects the multicast address automatically — no separate checkbox.

**Verify it's working:** in JTSpots' Alla tab, watch for rows ending in
`[WSJT-X]` as WSJT-X decodes new stations. If none ever appear, the two
most common causes are (1) the port doesn't match everywhere — compare
digit for digit, a silent mismatch produces no error anywhere, spots just
never arrive — or (2) Log4OM is older than 2.39.0.0.

## 3. Connect Log4OM to JTSpots' own outputs

- Point Log4OM's DX cluster client at `127.0.0.1:7300` (JTSpots' built-in
  Telnet server) to receive the *filtered* spots.
- Point Log4OM's Remote Control connection at the port shown in Config →
  Log4OM (default `2241`) for click-to-tune call display.
- Set your database connection (MariaDB or SQLite, matching your real
  Log4OM setup) in Config → Log4OM — needed for the DXCC/WAS/Marathon
  matrices and DXpedition tracking. The indicator next to it turns green
  once it can reach your log.

## 4. Fill in your station details

- Config → Station: your callsign, QTH grid locator.
- Config → DX Cluster: fill in your callsign on the preconfigured SK4BW
  entry (ships blank on purpose) before connecting, or add your own
  cluster server.
- Config → Radio interface: TCI, Omnirig, or None, depending on what your
  rig/SDR actually supports — see MANUAL.md §3 for the tradeoffs between
  them (Omnirig has no waterfall spotting or RX1/RX2 split, for example).

## 5. You're set

New spots that pass an active rule show up in Filtrerat and get forwarded
to Log4OM automatically. The shipped rule set (ATNO, new band-country,
WAS, DX Marathon, a few wanted-call examples) is a starting point — edit,
disable, or delete any rule in the Regler tab to match your own station.

The window can be resized by dragging any edge, if some columns look cut
off. A blue "Uppdatering" button appears in the status row automatically
whenever a newer version is available — no need to check the releases
page by hand.
