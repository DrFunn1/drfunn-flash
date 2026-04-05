# Adding New Firmware to the Flash Tool

## What lives where

```
firmware/
  catalog.json                          ← dropdown menu source of truth
  <project>-<YYYYMMDD>.bin              ← merged binary (bootloader + app, offset 0)
  <project>-<YYYYMMDD>.manifest.json    ← ESP Web Tools flash descriptor
```

---

## Step-by-step: publish a new build

### 1. Get the merged binary

From an `arduino-cli` build, the file you want is `MushroomDriver.ino.merged.bin`
(or equivalent for other projects). The `.merged.bin` contains bootloader +
partitions + app at the correct flash offsets — no separate address arguments needed.

Copy it here with a versioned name:
```
firmware/<project>-<YYYYMMDD>.bin
```
Example: `firmware/serene-shrooms-20260601.bin`

### 2. Create the ESP Web Tools manifest

Copy an existing `.manifest.json` and update the name, version, and bin filename:

```json
{
  "name": "SereneShrooms MushroomDriver",
  "version": "2026-06-01",
  "builds": [
    {
      "chipFamily": "ESP32-S3",
      "parts": [
        {
          "path": "serene-shrooms-20260601.bin",
          "offset": 0
        }
      ]
    }
  ]
}
```

`chipFamily` values: `ESP32-S3`, `ESP32`, `ESP32-S2`, `ESP32-C3`

Save as `firmware/<project>-<YYYYMMDD>.manifest.json`.

### 3. Add an entry to catalog.json

Append to the array in `firmware/catalog.json`:

```json
{
  "display": "SereneShrooms — MushroomDriver (2026-06-01)",
  "chip": "ESP32-S3",
  "description": "One-line description shown below the dropdown when selected.",
  "manifest": "firmware/serene-shrooms-20260601.manifest.json"
}
```

Entries appear in the dropdown in the order they appear in this file.
Put the newest build at the top if you want it selected first.

### 4. Push

```bash
git add firmware/
git commit -m "Add SereneShrooms MushroomDriver 2026-06-01"
git push
```

Cloudflare Pages auto-deploys on push to `main`. Live in ~30 seconds.

---

## Adding a new project (first build)

Same steps as above. No other files need editing — `index.html` reads
`catalog.json` dynamically. Just use a distinct project prefix in the filename
so builds sort cleanly, e.g. `dryer-controller-20260601.bin`.

---

## Cloudflare Access — managing the whitelist

Zero Trust → Access → Applications → flash.drfunn.com → Edit Policy → Add email addresses.
New users receive a one-time PIN to their email on first visit.

---

## ESP Web Tools reference

Library: https://github.com/esphome/esp-web-tools  
Requires Chrome or Edge (Web Serial API). Not supported on iOS or Firefox.
Works on Chromebook Linux is NOT needed — flash directly from the Chrome browser.
