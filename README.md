# TVR MBETool Log Tools
A browser-based diagnostic log viewer and comparator for TVR Speed Six engines running the MBE 9A4 ECU, built around CSV logs exported from [MBETool](http://www.wiechens.de/mbetool/mbetool.zip) by EvoOlli.
All processing happens locally in the browser. No data leaves your machine or network.

---

## Tools

### Log Viewer (`viewer.html`)
Load a single MBETool CSV log and inspect all engine parameters interactively over time.

### Log Comparator (`compare.html`)
Load two logs side by side — before and after a repair, tune or adjustment. Hover, zoom and pan are synchronised across both panels so the same moment in time is always aligned.

### Landing page (`index.html`)
Start here — links to both tools.

---

## Features

### Charts
- **14 channels** — RPM, Adaptive 1 & 2, Lambda ECU 1 & 2, Throttle 1 & 2, Injection 1 & 2, Water temp, Air temp, Oil pressure, Ignition timing, Battery voltage
- Each channel has its own Y-axis scale so nothing gets squashed
- Toggle any channel on or off with the colour-coded buttons in the toolbar
- **Scroll to zoom** — centred on the mouse position
- **Click and drag to pan** when zoomed in
- Zoom buttons and Reset in the toolbar

### Trendlines
Click the **Trendlines** button to cycle through three modes:

| Mode | What you see |
|---|---|
| **Off** | Raw data only |
| **Both** | Raw data dimmed, trendline drawn on top |
| **Trend only** | Only the smooth trendline — clean and uncluttered |

Choose the rolling window size: 50, 100 or 200 samples, or full-range linear regression.

### Values strip
A colour-coded values bar sits permanently below each chart, updating as you hover. Shows all active channels at the cursor position, with adaptive values highlighted red when below −15%. Status flags (AFR123, AFR456, Lambda 1 & 2, Battery, Water Temp) show in a second row — green for OK, red for fault.

In the **viewer**, the hover tooltip can additionally be toggled on or off independently of the strip.

### Session Summary panel
A four-card summary panel computed from the full log (not just the visible zoom window):

**Session**
- Sample count and log duration
- Max and mean RPM
- Max water temperature
- Time taken to reach 80 °C operating temperature (flags red if thermostat is stuck open)

**Bank Balance**
- Mean absolute difference between Adaptive 1 and Adaptive 2 — ideal < 5 %
- Per-bank mean adaptive, worst (most negative) adaptive
- Percentage of the log spent beyond the −15 % threshold

**Lambda Health**
- Per-bank status (Healthy / Check)
- Switching frequency in crossings per second — healthy range 0.5–3 /s; a slow or non-switching sensor flags amber or red
- Symmetry: percentage of time reading rich — ideal 40–60 %; outside this range indicates a sensor bias or fuelling imbalance

**Fault Log**
- All six ECU status flags with either OK (green) or the percentage of the session they were faulting (red)

In the **comparator**, each panel gets its own summary row so you can read both logs' statistics without hovering.

---

## Files

```
index.html      Landing page — choose viewer or comparator
viewer.html     Single log viewer
compare.html    Two-log comparator
README.md       This file
```

---

## Log file format

MBETool exports semicolon-separated CSV files with comma as the decimal separator. The Time column contains wall-clock timestamps in `HH:MM:SS` format. Both formats are handled automatically.
Drop the CSV file directly onto the tool in the browser — no conversion needed.

---

## Self-hosting with plain Docker

If you have Docker installed but are not using Portainer, you can run the tool with a single command. This works on a Raspberry Pi, any Linux machine.

### Step 1 — Create the folder and copy the files

```bash
sudo mkdir -p /opt/tvr-viewer
sudo chown $USER:$USER /opt/tvr-viewer
```

Copy the three HTML files into that folder, then run:

```bash
docker run -d \
  --name tvr-viewer \
  --restart unless-stopped \
  -p 8080:80 \
  -v /opt/tvr-viewer:/usr/share/nginx/html:ro \
  nginx:alpine
```

The tool is now available at `http://localhost:8080` on the machine itself, or `http://YOUR-MACHINE-IP:8080` from any other device on the same network.

### Useful commands

Check the container is running:
```bash
docker ps | grep tvr-viewer
```

Stop the container:
```bash
docker stop tvr-viewer
```

Start it again:
```bash
docker start tvr-viewer
```

Remove it entirely:
```bash
docker stop tvr-viewer && docker rm tvr-viewer
```

### Updating files

Copy the new HTML file into `/opt/tvr-viewer/` — no container restart needed. Nginx reads directly from disk. Hard refresh the browser (`Ctrl+Shift+R`) to clear the cached version.

### Running without Docker at all

Because the tool is plain HTML with no server-side code, you can also open the files directly in any browser:

```
File → Open → viewer.html
```

This works on any system with no installation. If drag-and-drop does not work due to browser security restrictions, use the browse button, or serve the files locally with Python:

```bash
cd /path/to/files
python3 -m http.server 8080
```

Then open `http://localhost:8080` in the browser.
---


## ECU and software background

The TVR Speed Six uses an MBE 9A4 ECU in a TVR-specific configuration. MBETool (version 0.97, January 2019) is a community-developed Windows diagnostic tool written by EvoOlli that reverse-engineered the serial protocol. It connects via a simple USB-to-serial adapter on any COM port.

---

## Key diagnostic reference

| Parameter | Healthy range | Notes |
|---|---|---|
| Adaptive 1 & 2 | −10 % to +10 % | > ±20 % triggers AFR fault |
| Bank balance | < 5 % mean difference | Persistent gap points to TB imbalance or sensor wiring |
| Lambda switching | 0.5–3 /s | Below 0.3 /s suggests lazy or contaminated sensor |
| Lambda symmetry | 40–60 % rich | Bias toward rich = sensor earth noise or injector leak |
| Water temp (operating) | 85–90 °C | Consistently below 80 °C suggests stuck-open thermostat |
| Oil pressure at idle | 16–30 PSI | Low only at idle (< 1,000 RPM) is normal for the S6 |
| Oil pressure > 2,000 RPM | > 40 PSI | Low under load needs investigation |

### Common faults on the Speed Six

**Lambda earth wiring** — the heater earth and signal earth share a splice in the standard TVR loom. Heater current corrupts the signal voltage, causing the ECU to read rich and pull fuel via negative adaptives. Fix: disconnect the heater earth from the loom splice and run a short wire directly to a nearby engine block earth point. Both sensors can improve by this mod.
**Throttle body balance** — idle bypass screws with locknuts on a hot engine. Adjust in quarter-turn increments at fully warmed-up operating temperature, watching Adaptive 1 and Adaptive 2 converge toward zero in MBETool live mode.

---

## Acknowledgements

- **EvoOlli** — MBETool, the only reason any of this is possible
- **TVR PistonHeads community** — decades of accumulated Speed Six knowledge
- **Chart.js** — charting library used in the viewer

