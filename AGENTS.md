# Project preferences

## Git

- **NEVER push to GitHub** (or any remote). Commits stay local unless the user explicitly asks to push.
- **NEVER add the `Co-Authored-By: Devin` trailer** to commit messages. Write plain commit messages with no co-author line.

## Documentation Mandate

- **Before any modification** to code, configs, or documentation, the agent MUST create a timestamped backup in `/home/pi/backups/YYYYMMDD_HHMMSS_<description>/`.
- **After any modification**, the agent MUST document the changes in `CHANGELOG.md` with:
  - Date of change
  - Description of what was added/changed/fixed
  - Files affected
  - Testing results (if applicable)
- **Commit messages** must clearly describe the change (e.g., "Add dashboard password change feature").

## Hardware

- **Board:** ESP32-C3 (XMC flash, 4MB, revision v0.4)
- **MAC:** `0c:4e:a0:68:ee:88`
- **Port:** `/dev/ttyACM0`

## Build & Flash (from source — REQUIRED)

The prebuilt `full-flash.bin` from GitHub Releases **does not work** on this board.
It causes an immediate `TG0WDT_SYS_RST` (watchdog crash loop), likely due to
flash mode/frequency mismatch with the XMC flash chip. Always build from source.

### Steps

1. **Create `secrets.h`** (one-time):
   ```bash
   cp src/secrets.example.h src/secrets.h
   ```

2. **Build the blocklist** (~140k domains):
   ```bash
   mkdir -p data
   python3 tools/build_blocklist.py data/blocklist.bin
   ```

3. **Build firmware:**
   ```bash
   pio run -e c3
   ```

4. **Upload firmware + filesystem:**
   ```bash
   pio run -e c3 -t upload
   pio run -e c3 -t uploadfs
   ```

5. **Monitor serial** (optional):
   ```bash
   screen /dev/ttyACM0 115200
   ```
   The dashboard password is printed on boot: `[cfg] dashboard password: <pw>`

### Notes

- WiFi credentials are configured via the **captive portal AP** (`C3AdBlock-Setup`)
  on first boot, not in `secrets.h` (which can be left as defaults).
- Dashboard auth: **Basic Auth** — any username, password from serial output.
- DNS: point devices to the C3's IP (or set in router DHCP).
- OTA updates available via dashboard after initial flash.
