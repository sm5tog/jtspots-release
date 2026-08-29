# JTSpots — User Manual

Version covered: 1.28.0 | Operator: SM5K / SM5TOG

This manual explains how to install, configure, and use JTSpots from an
operator's point of view. For internal architecture and data formats, see
`DOCUMENTATION.md` instead — that file is a developer reference, this one
is not.

---

## 1. Introduction

JTSpots is a bridge application that sits between your decoding/spotting
sources and Log4OM:

- **WSJT-X / JTAlert** (FT8/FT4 decodes, via UDP multicast)
- **DX cluster** (Telnet, e.g. VE7CC, AR-Cluster, or your own skimmer)
- **QO-100 (Es'Hail-2)** — a public WebSDR poller (IS0GRB), for satellite FT8/FT4

Every spot from these sources passes through a configurable rule engine
(new DXCC entity, new band-country, WAS, DX Marathon, wanted calls, and
more). Spots that pass are forwarded to **Log4OM** through a local Telnet
server, exactly as if JTSpots itself were a DX cluster feed.

You can also double-click any spot to have JTSpots tune your rig to the
right VFO/frequency/mode and show the call in Log4OM's own window (via
Log4OM's Remote Control interface) — without logging a QSO. This mimics
what JTAlert does when you click a call there. For WSJT-X-sourced spots,
double-click (or the "Call via WSJT-X" right-click option) can also tell
WSJT-X to reply to that decode directly, exactly as if you had
double-clicked it in WSJT-X's own Band Activity window — these calls are
shown underlined in the DX! tab.

---

## 2. Installation & First Start

### Installing

JTSpots is distributed as a folder containing `JTSpots.exe` plus a small
set of data files (bandplan, DXCC database, etc.). Unzip that folder
anywhere you like — for example `C:\JTSpots\` — and keep its contents
together in that one folder. No Python installation, no pip packages, no
separate runtime to set up.

Double-click `JTSpots.exe` to start the program. All settings, caches,
and logs are created automatically inside the same folder on first run —
nothing needs to be created by hand.

JTSpots ships with a ready-to-use starting point: a working rule set
(ATNO, new band-country, WAS, DX Marathon, and a few example wanted-call
rules — see §6) and one preconfigured DX cluster server (**SK4BW**,
`cluster.sk4bw.net:8000`). Fill in your own callsign in Config → DX
Cluster before connecting to it — the callsign field ships blank on
purpose, so you don't accidentally connect as someone else. Feel free to
edit, disable, or delete any of the starting rules to match your own
station and interests.

If you ever move JTSpots to a new folder, move the **entire folder**,
not just the .exe — the data files it depends on (bandplan, DXCC
database, your settings and caches) live alongside it and are not
bundled inside the executable itself.

The window can be resized like any other — drag any edge or corner. The
default size may cut off some log columns depending on your screen and
font settings; widening the window is the fix, not a sign of something
wrong with the data.

The status row at the top of the window also has a **"^" (always on top)**
toggle (its state is remembered between runs) and an **"About..."** button
with the version number and links to the setup guide and the project
donation page.

### Updating

When a newer version is available, a blue "Uppdatering: X.Y.Z" button
appears in the status row at the top (checked once an hour automatically,
no action needed to trigger the check). Click it to update — JTSpots
closes itself and hands off to a small updater program that downloads and
installs the new version, then restarts JTSpots for you. Your settings,
rules, and cluster list are always preserved; you only lose them if you
explicitly choose the "overwrite everything" recovery option the updater
offers (for a corrupted installation), which is not the default.

### Folder layout

Everything JTSpots creates or reads (settings, the DXCC database file,
caches, logs) lives in the same folder as `JTSpots.exe`. Nothing is
written outside that folder except the connection to your Log4OM
database itself (see below).

### Log4OM connection

JTSpots needs to reach your Log4OM installation in three ways:

1. **Telnet spot feed** — JTSpots runs its own small Telnet server
   (default port 7300). Point Log4OM's own DX cluster client at
   `127.0.0.1:7300` to receive the filtered spots.
2. **Remote Control** — used for click-to-tune (shows the call in
   Log4OM's UI). Default port 2241.
3. **Database read access** — needed for the DXCC/WAS/Marathon matrices
   and DXpedition tracking. Choose **MariaDB** (Log4OM2's default for
   multi-station contest setups) or **SQLite** (Log4OM2's own default for
   a normal single-station installation) in Config → Log4OM. Point it at
   your real Log4OM database — JTSpots only reads from it, it never
   writes.

---

## 3. Configuration (Config tab)

The Config tab is organized into collapsible sections. Station and Log4OM
are always expanded; the rest start collapsed so a first run shows a short
list of headings with a one-line explanation under each, instead of every
field at once. Click a heading to expand or collapse it — the state is
remembered between runs.

Most fields have a hover tooltip (marked with a small "ⓘ" after the
label). Where a field has a sensible default, the tooltip states it.

Each section is described below.

### Log4OM

Telnet port, Remote Control host/port, and the database connection
(MariaDB host/port/database/user/password, or a SQLite file path). A
green/red indicator next to "Database type" shows whether the configured
database is currently reachable.

**"Enable Log correction QO-100"** (off by default): watches the Log4OM
database for new QSOs on 13cm and automatically fills in `propmode=SAT`,
`satname=QO-100`, and `satelliteqso=1` — before Log4OM's own periodic
Clublog upload sends the row off incomplete. Polls every 5 seconds while
enabled.

### Radio interface

Choose how JTSpots controls your radio for click-to-tune: **TCI**,
**Omnirig**, or **None**.

> **Important — some features only work with TCI.** Waterfall spotting,
> panorama centering, and the RX1/RX2 dual-receiver split are all
> TCI-specific (they require a TCI-capable SDR such as an Expert
> SunSDR/ExpertSDR3). If you choose **Omnirig**, click-to-tune still
> works, but only as a simple "set frequency + mode" on one VFO pair —
> there is no waterfall spotting, no panorama, and no split. If you
> choose **None**, no radio control happens at all; the rest of the
> program (filtering, forwarding to Log4OM, statistics) works exactly the
> same either way.

Switching the radio interface requires restarting JTSpots before the
Spots tab's **Waterfall ▾** button updates to match (the actual tuning
target changes immediately, but that button's layout only refreshes at
startup). The Radio tab's own fields (including "Use RX2" below) update
immediately, no restart needed.

**RX2 (second receiver)** is optional, TCI-only — the "Use RX2" checkbox
and the "RX per mode" row are hidden entirely whenever Omnirig or None is
selected, since RX1/RX2 only means something with a TCI-capable radio.
Switch back to TCI to see them again. If your SDR only has one receiver,
uncheck "Use RX2" — everything then tunes to your single receiver, and the
RX1/RX2 choice disappears from the right-click menu and the rest of the
UI. If you do have a second receiver (e.g. a SunSDR2 DUO),
leave it checked and use the "RX per mode" row to choose which receiver
each mode category (CW/Phone/Digital) tunes to by default — the built-in
default sends CW/Phone to RX1 and Digital modes to RX2, matching common
dual-receiver setups, but you can reassign any of the three.

### WSJT-X / QO-100

Multicast IP/port for WSJT-X's UDP broadcast, and a checkbox to enable
the QO-100 poller (off by default). When enabled, JTSpots polls a public
QO-100 WebSDR every 15 seconds for FT8/FT4 decodes and manual spots.

> **These two fields must match WSJT-X's own settings exactly**, digit for
> digit — not just "look similar". Open WSJT-X's own File → Settings →
> Reporting tab and compare the "UDP Server" and "UDP Server port number"
> fields there against Config → WSJT-X/QO-100 here. A mismatch (even a
> single-digit port difference) means JTSpots simply never receives
> anything from WSJT-X — nothing appears in the Alla log for that source,
> and there is no error message, since nothing ever arrives to fail on.
> The default in both places is `224.0.0.1` / `2237`; if you changed
> WSJT-X's port for any reason (e.g. to share it with another program),
> change it here too. See the quick-start guide for the full multicast
> setup across WSJT-X, JTAlert, and Log4OM together.

### Rotor

Your station's QTH grid locator (Maidenhead, e.g. `JO89CS`) and the
UDP host/port of your rotor controller. Used by the "Send AZ to rotor"
button/menu item, which calculates the great-circle bearing to the
spotted station's country and sends it via the same UDP format used by
PstRotator and several logging programs (not something unique to this
program — a common interface).

### DX Cluster

Add one or more Telnet cluster servers here (name, host, port, optional
init commands). You can connect to several at once.

### cty.xml

A button to update the local DXCC database (Clublog's `cty.xml`), plus a
status line showing when it was last updated. No account or API key
needs to be entered — the download works automatically for every
installation.

### JTAlert

Optionally writes your combined wanted-calls list (both the auto-filled
DX News list and the manual list, across all rules) to JTAlert's own
wanted-list (`config.sqlite`, list 3), so JTAlert alerts on the same
calls JTSpots is tracking. Off by default. The path to JTAlert's
`config.sqlite` is auto-detected on first use (it lives under
`%LOCALAPPDATA%\HamApps\<profile>\config\JTAlertX\`) — use "Browse..." if
you have a non-standard install or multiple JTAlert profiles and the
wrong one was picked. "Export now" writes immediately; once enabled, it
also writes automatically whenever your wanted-calls list changes.

**"Restart JTAlert on export"** (on by default): JTAlert keeps its own
copy of the wanted list in memory and writes it back to `config.sqlite`
on its next save, which would overwrite what JTSpots just wrote. To make
the change stick, JTSpots closes JTAlert, writes the list, and starts
JTAlert again with the exact same command line. The program path is found
automatically from the running JTAlert process (or the "JTAlert program:"
field, or a scan of `%ProgramFiles%\HamApps\`). Turn this off only if you
prefer to close JTAlert yourself before every export.

### Alerts

A visible and audible alert when a new **DX Marathon** or **QO-100 DXCC**
spot appears in Filtered/DX! — the two spot types most worth reacting to
immediately.

- **Sound** and **taskbar flash** checkboxes (both on by default). The
  flash keeps blinking until the JTSpots window gets focus, and only
  fires if JTSpots is not already the active window.
- **Sound device** — pick a specific output so the alert plays there
  regardless of the Windows default device or per-app mixer. "Windows
  default device" keeps the old behaviour (follows whatever Windows has
  set as default).
- **"Test alert now"** runs sound + flash immediately, independent of
  rules or DXCC status, to check the mechanism itself.

**Pushover notifications** (off by default): sends a push notification to
the Pushover app on your phone when a spot matches an alert of at least
the urgency you choose. Requires an API token and user key from
pushover.net (stored in plain text in the settings file). The urgency
threshold ranges from "Operation-critical only" (alerts from a tracked
DXpedition) through "Operation + ATNO" up to "Everything (incl. manual
calls)". A repeat guard ("Don't notify about the same call again within N
minutes", default 60) stops the same call+band notifying again until the
time is up, but a band change gets you a fresh notification right away.
All notifications go out at normal priority (one chime, no repeat,
respects Do Not Disturb). "Test Pushover now" sends a test message with
the saved credentials — save settings first, and check the "Alla" log for
the result.

### Language

Choose the UI language (Swedish or English). Most text switches
immediately; a restart is recommended so that everything, including
any already-open dialogs, is consistent. As of this version every
user-visible string in the whole program goes through the language files
(`lang/sv.json`, `lang/en.json`), so nothing is left untranslated in
either language.

---

## 4. Main Views

The main window has five top-level tabs: **Spots**, **Timeline** (§5),
**Rules** (§6), **Statistics** (§9), and **Config** (§3). This section
covers the Spots tab.

The Spots tab has four sub-views, shown as tabs of their own:

- **Alla (All)** — the raw log: every spot from every source, before
  filtering, plus connection/status messages. Nothing is ever filtered
  out of this view (only exact duplicate spots are merged).
- **Filtrerat (Filtered)** — only spots that passed at least one active
  rule. This is also what gets forwarded to Log4OM.
- **DX!** — a compact view highlighting spots worth working right now
  (colored by DXCC/DXpedition/Marathon status). A row shows `[WKD yymmdd]`
  when it re-triggers on a "worked but not yet confirmed" basis and you
  already worked that exact call+band on that date — a reminder before
  you call it again.
- **Skräp (Junk)** — spots rejected by the callsign/junk filter (see
  §10), so you can review and correct false positives.

**Sorting the DX! view.** A **Sort ▾** button chooses the field and
direction (time, band, call, DXCC, spotter, mode, trigger, source). A
separate **"Opportunity sorting"** checkbox overrides that menu: when on,
the list is ranked by the built-in DXO Opportunity score instead —
priority tier first (an active DXpedition slot outranks an ATNO, which
outranks a new band), then quality of the opening (how reachable and how
fresh the spot is) within the same tier, with the most recently received
spot last on a tie. Hover a DX! row to see the score and the reason it
was ranked where it is.

**Identifying where a spot came from:** every row ends with a bracketed
source tag, e.g. `[WSJT-X]`, `[SK4BW]` (a DX cluster server name),
`[Hamalert]`, or `[QO100]`. This is the quickest way to confirm a
particular source is actually feeding spots in — if you've just set up
WSJT-X's multicast connection and want to check it's working, look for
`[WSJT-X]` rows appearing in Alla as new decodes come in, rather than
guessing from the rest of the line.

Right-clicking a spot row (in Alla, Filtrerat, or DX!) opens a context
menu with actions such as sending the bearing to your rotor, tuning your
rig via TCI, tracking the call as a DXpedition, adding/removing it from
your wanted-calls list, adding it to the junk filter, or removing/
blacklisting the row.

### Info panel

Click any spot row once to show its details: country, continent, DXCC
number, and two side-by-side matrices — a DXCC band×mode matrix (green
dot = confirmed, yellow dot = worked but not confirmed) and, if the
station is a US/Canadian call, a WAS matrix for its state right next to
it. If the spotted call is a tracked DXpedition, a black checkmark
appears over any cell it has already worked in that specific expedition,
independent of your overall DXCC confirmation status.

### Click-to-tune

- **Single click** on a spot row shows its info panel (see above).
- **Double click** tunes your radio (via TCI or Omnirig, per your Radio
  interface setting) to the right VFO/frequency/mode for that spot, and
  sends the call to Log4OM's Remote Control window.
- The **Waterfall ▾** and **Mode ▾** menu buttons above the log (visible
  when TCI is selected) let you jump the SDR panorama or filter by mode
  category directly.

---

## 5. Timeline Tab

A 30-day forward view of upcoming DXpeditions, drawn as a Gantt-style
chart similar to dx-world.net's own timeline page, plus a text list of
the same events below it. It is built from three of the four DX News
sources (425 DX Bulletin, NG3K ADXO, and the DX-World timeline feed).

- **Update** is manual only — there is no automatic fetch at startup.
  The same click also refreshes the "DX" rule's auto-filled wanted-calls
  condition (the same effect as the "DX News Union" button in the rule
  editor).
- **Hide special calls** filters commemorative/special-event calls out
  of the view.
- Each event row links to its source. Right-click a call in either the
  chart or the list to start tracking it as a DXpedition (§7).

Independently of this tab, JTSpots also refreshes the wanted-calls list
on its own every 12 hours, and the "Alla" log then shows one line per
source (425 DX / NG3K / DX-World / Timeline) so you can see which fetch,
if any, failed.

---

## 6. Rules (Rules tab)

The Rules tab holds your list of filter rules. Each rule is a named group
of conditions (all AND-ed together, except that multiple "wanted call"
conditions inside one rule are OR-ed against each other). A spot passes
if **any** enabled rule matches it, or if it is an active DXpedition not
yet worked on that band/mode (see §7) — DXpedition matches are automatic
and need no rule.

### Condition types

| Condition | Meaning |
|---|---|
| ATNO | All-Time-New-One — DXCC entity never worked at all |
| New band-country | This DXCC entity, on this specific band, never worked (or only worked, not confirmed, if "Trigger even on wkd" is checked) |
| Satellite DXCC | DXCC entity never worked via satellite (QO-100) |
| DX Marathon this year | New Marathon entity/zone this year, or new for a specific band |
| WAS | US state needed on this band/mode |
| Wanted callsign | Matches a comma-separated list of calls (auto-filled from DX News, or maintained manually — see below) |
| Exclude callsign | Blocks specific calls |
| Band / Mode | Simple band or mode match |
| Source | WSJT-X / Cluster / QO-100 decode / QO-100 log |
| Min SNR | Minimum decoded SNR in dB |
| Spotter continent / Spotter country | Restrict by where the *spotter* (not the DX station) is located |
| Has QRZ | Optional cross-check against QRZ.com to filter out likely skimmer decoding errors (a rare miss on stations without a QRZ page is possible — keep this off calls you're specifically chasing) |
| Popular call | Matches when 3+ (adjustable) distinct stations are calling the same target within a short window, and JTSpots has itself decoded that target's own transmission recently — a possible pileup/rare DX indicator. WSJT-X spots only, never triggered by cluster or QO-100 spots. When a rule with this condition matches, the call is shown with a "!" mark in the DX! tab. |

### Wanted calls

A rule's "Wanted callsign" condition can be filled automatically from
the "DX News Union" / "DX News Special" buttons (overwrites the whole
list each time), or maintained by hand — either by typing comma-separated
calls directly, or by right-clicking any spot and choosing "Add CALL to
Wanted calls" / "Remove CALL from Wanted calls". The two lists never
overwrite each other.

### Test panel

A small panel lets you feed a synthetic spot through the rule engine to
verify a rule behaves as expected, without waiting for a real spot.

### Re-run filtering

The "Re-run filtering" button next to Filtered's header re-evaluates
every already-received spot against the current rules — useful right
after changing a rule or updating cty.xml, since filtering normally only
happens when a spot first arrives.

---

## 7. DXpedition Tracking

Track a specific callsign to see every spot of it regardless of whether
the DXCC entity is otherwise already confirmed — useful for working a
DXpedition on every band/mode combination it offers, not just the ones
you still need for DXCC.

- **Filtering happens automatically, with no visible rule.** As soon as a
  call is tracked, any of its still-unworked band/mode spots are passed
  to Filtered (and forwarded to Log4OM) on their own — this runs
  completely in the background, separately from the Rules tab. You will
  never see a rule for it in your rule list, and you don't need to
  create one; nothing to configure.
- **Start tracking:** right-click any spot → "Create DXpedition (CALL)".
- **Manage:** the DXpedition panel (between the info panel and the spot
  log) has a dropdown of tracked calls, a button to remove one (with a
  confirmation prompt), a button to edit the tracking start date, and an
  "Update" button to force an immediate refresh from your Log4OM log.
- **★ marker:** a tracked call always shows a ★ next to it in the DX!
  view, regardless of worked status.
- **Priority coloring:** an unworked spot from a tracked DXpedition is
  shown in orange (brand new band-country), blue (worked but not
  confirmed), or dimmed gray (already confirmed) — these DXP colors take
  priority over ordinary ATNO/Marathon coloring.
- **History:** the "History" button (or right-click → "Show history for
  CALL") opens a popup with every spot ever seen of that call since you
  started tracking it — a chronological list plus a heatmap (Overview:
  one row per band, all days combined; Detailed: one row per date per
  band) to help you spot when that station tends to be active. A "Fetch
  from cluster" button asks all connected clusters for that call's own
  history (`SH/DX <call>`) to fill in gaps — requires at least one
  connected cluster, and only reaches as far back as that cluster's own
  memory.

---

## 8. DX Cluster

Connect to one or more Telnet cluster servers configured in Config → DX
Cluster. Spots received from a cluster go through the same filtering
pipeline as WSJT-X spots. You can search the "Alla" (All) log for a
specific callsign or text using the search field above it.

---

## 9. Statistics Tab

The Statistics tab shows your full confirmation status across DXCC, WAS,
QO100, and DX Marathon, pulled directly from your Log4OM log.

- Choose the statistic type (DXCC / WAS / QO100 / Marathon) at the top.
- **Overview** shows one row per band (or, for QO100, per mode): worked,
  confirmed, total possible, and percentage — plus a final "Combined" row
  showing your best status per entity across all shown columns (not a
  sum, so an entity worked on several bands is not double-counted).
- **Matrix** shows one row per DXCC entity (or US state), with a dot per
  band/mode. Colored green if at least one shown column is confirmed,
  yellow if only worked, red if never worked.
- **Mode ▾** lets you check any combination of CW/Phone/Digital (and, in
  this tab only, an extra "Mixed" option that adds a combined row per
  entity). Checking more than one mode gives you one matrix row *per
  mode* for each entity, grouped country-first — not a single merged row.
- A **year selector** restricts the matrix/overview to QSOs from a
  specific year; "Aktuella"/"Aktuella + historiska" (Current /
  Current & Deleted) toggles whether deleted historical DXCC entities are
  included (DXCC/QO100 only).
- A **search field** in Matrix view matches on the entity name, or on a
  callsign prefix — type `P5` to jump straight to North Korea without
  needing to know the country name.
- A **"Hide unworked"** checkbox (Matrix view only) hides an entity
  entirely unless at least one of its shown rows has a worked or
  confirmed status on some visible column.
- DX Marathon shows only a text summary (units/zones/total score) — no
  matrix, since Marathon has no per-band/mode concept.

---

## 10. Junk Filter

Rejected decodes (usually skimmer/RTTY misreads that happen to look like
an exciting DX call) are captured in the "Skräp" (Junk) tab instead of
silently disappearing, so you can check and correct them:

- **Whitelist** — right-click a junk entry → "Exclude CALL from junk
  filter" if it was wrongly flagged. That call will always pass from
  then on.
- **Manual junk list** — right-click any real spot (in Alla or Filtered)
  → "Add CALL to junk filter" to force all future spots of that call into
  the junk log, even if it would otherwise look valid.
- **Dump file** — every rejected decode is kept (capped at 500 entries)
  along with the original raw text, so you can recognize a recurring bad
  pattern.

---

## 11. Troubleshooting

- **Database indicator stuck or red:** check host/port/credentials (or
  SQLite file path) in Config → Log4OM. The indicator re-tests itself
  whenever you leave a field or press Enter in it.
- **A spot never appears in Filtered:** check whether it might be caught
  by the junk filter (look in the Skräp tab) or blocked by an active
  1-day blacklist entry.
- **A rule you expect to trigger doesn't:** use "Re-run filtering" after
  any rule or cty.xml change, and double-check condition order if you use
  "Has QRZ" (it's cheapest to put it last, since conditions are checked
  in order and QRZ lookups take a moment).
- **Click-to-tune does nothing:** confirm your Radio interface (Config)
  matches what's actually running (TCI needs the SDR's TCI server
  running and reachable at the configured host/port; Omnirig needs
  OmniRig itself running with a rig configured).
- **WAS/DXCC numbers look wrong compared to Log4OM's own statistics:**
  make sure the Statistics tab's Mode ▾ selection includes the same modes
  Log4OM's own report used, and check the "Aktuella"/"Aktuella +
  historiska" toggle for DXCC/QO100.
- **No WSJT-X spots ever appear, everything else works fine:** almost
  always a port mismatch — compare Config → WSJT-X/QO-100's UDP port
  field against WSJT-X's own Settings → Reporting → "UDP Server port
  number" digit for digit. A mismatch produces no error anywhere (see the
  warning box in §3), it just silently receives nothing.
- **"WSJT-X-paket-fel" appears in the All tab:** a single WSJT-X UDP packet
  couldn't be processed; it's logged and skipped, WSJT-X spot reception
  continues normally. If it repeats constantly, it's worth reporting (with
  the exact error text) so the underlying packet format can be fixed.
- **"TCI-spot-fel" repeats constantly in the All tab:** the Waterfall ▾
  checkboxes are on but Radio interface isn't set to TCI (or the TCI
  server isn't reachable) — switch Radio interface to match what's
  actually running, or turn the Waterfall ▾ checkboxes off.

---

## 12. Glossary

| Term | Meaning |
|---|---|
| ATNO | All-Time-New-One — a DXCC entity never worked before |
| wkd | Worked — a QSO exists, but not yet confirmed |
| cfm | Confirmed — the QSO has a confirmed credit (e.g. via LoTW) |
| DXCC | The Clublog/ARRL entity list used to track "countries" worked |
| WAS | Worked All States — the US state award |
| DX Marathon | An annual worked-entities/zones competition |
| ATNO/DXP1/DXP2/DXP3 | Priority tags shown on spots: brand-new band-country, worked-not-confirmed, and already-confirmed, in that priority order |
| TCI | Expert Electronics' text-based WebSocket protocol used by SunSDR/ExpertSDR3 radios |
| CAT | Computer-Aided Transceiver control — traditional serial/USB rig control, reached here via Omnirig |
| DXO Opportunity | JTSpots' built-in scoring of how worth-it a spot is right now (priority tier + reachability + freshness), used to rank the DX! view when "Opportunity sorting" is on |
| Pushover | A third-party push-notification service (pushover.net); JTSpots can send an alert to its phone app |
