# JTSpots

JTSpots is a Windows app that bridges your DX-spot sources — **WSJT-X /
JTAlert** (FT8/FT4), a **DX cluster** (Telnet), and **QO-100** (satellite
FT8/FT4) — into **Log4OM**, filtering every spot through a configurable
rule engine (new DXCC entity, new band-country, WAS, DX Marathon, wanted
calls, and more) before forwarding it on. It also shows a full DXCC/WAS/
QO100/DX Marathon statistics view pulled directly from your Log4OM log,
and can tune your rig to a spotted station's frequency/mode with a
double-click.

## Download

Get the latest version from the **[Releases](../../releases/latest)**
page. Download `JTSpots-vX.Y.Z-win64.zip`, unzip it anywhere (e.g.
`C:\JTSpots\`), and double-click `JTSpots.exe` — no Python, no separate
installer, no admin rights needed.

## Requirements

- Windows 10/11 (64-bit)
- [Log4OM2](https://www.log4om.com/) already installed and set up — JTSpots
  reads your existing log, it doesn't replace Log4OM
- WSJT-X/JTAlert and/or a DX cluster connection, if you want to use those
  spot sources

## Quick start

New here? **[QUICKSTART.md](QUICKSTART.md)** gets a fresh install
receiving and forwarding spots in the shortest path, including the
WSJT-X/JTAlert/Log4OM multicast setup.

## Full manual

See **[MANUAL.md](MANUAL.md)** for installation details, every Config
tab explained, the rule engine, DXpedition tracking, the Statistics tab,
and troubleshooting.

---

This repository only carries the packaged app and its documentation —
source code is developed in a separate, private repository.
