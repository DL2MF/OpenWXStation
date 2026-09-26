# OpenWXStation - A Weather Station for LilyGO T-Display S3

_Based on ["Cheap Internet Weather Station using LilyGo T-Display S3" by Volos Projects.](https://github.com/VolosR/tDisplayS3WeatherStation)_

## This is an ESP32 Internet Weather station project using the LilyGo T-Display S3. 

It's fully integrated into the OpenWX Suite with:
[OpenWXSDR](https://github.com/DL2MF/OpenWXSDR), [OpenWXTTGO](https://github.com/DL2MF/OpenWXTTGO), [OpenWXDeck](https://github.com/DL2MF/OpenWXDeck) 
and [OpenWXFleetMonitor](https://github.com/DL2MF/OpenWX-Fleet-Monitor).

## Parts used:
LilyGO T-Display S3 [https://www.lilygo.cc/](https://lilygo.cc/products/t-display-s3-copy)   

<img width="342" height="189" alt="grafik" src="https://github.com/user-attachments/assets/792da0a4-045d-4128-adcb-0ce198115383" />

### ... but OpenWXStation is even more - it's a tiny and versatile Radiosonde Decoder Station Manager.

<img width="342" height="191" alt="grafik" src="https://github.com/user-attachments/assets/4407226a-7450-4003-acd5-29d978e589a0" />


<img width="1671" height="997" alt="grafik" src="https://github.com/user-attachments/assets/df6ef8b3-e0ec-40a2-beef-c5d602a895ab" />


<img width="1666" height="991" alt="grafik" src="https://github.com/user-attachments/assets/e65849ed-c1b1-4dd9-9b01-ce3e7b0291af" />


## Build with PlatformIO (VS Code)
1. Install the **PlatformIO IDE** extension in VS Code and open this folder (the one containing `platformio.ini`).
2. Build: PlatformIO toolbar ✓ (or `pio run`). Libraries and the ESP32 core (Arduino 2.0.x) download automatically.
3. Upload: → (or `pio run -t upload`), then monitor: 🔌 (or `pio device monitor`).
4. If upload does not start: hold **BOOT**, tap **RST**, release BOOT, upload again, then tap **RST** once after flashing.

The TFT_eSPI display configuration (Setup206, T-Display S3) is set in `platformio.ini` – no edits to the library needed.

## Arduino IDE settings (if you keep using it)
Board *ESP32S3 Dev Module*, ESP32 core **2.0.x** (not 3.x), USB CDC On Boot: Enabled, Flash Size: 16MB, PSRAM: **OPI PSRAM**, Partition: 16M Flash (3MB APP/9.9MB FATFS) or Huge APP.
In `Arduino/libraries/TFT_eSPI/User_Setup_Select.h` comment out `#include <User_Setup.h>` and enable `#include <User_Setups/Setup206_LilyGo_T_Display_S3.h>`.

## Configuration (`config.yaml` on SPIFFS)
All settings live in `OpenWXStation/data/config.yaml`: location (name, latitude, longitude), Open-Meteo API URL,
units, pressure type, update/retry intervals, graph length, time zone (POSIX TZ with automatic DST), NTP server,
display rotation/brightness/night dimming, and WiFi portal settings.

1. Edit `OpenWXStation/data/config.yaml`
2. PlatformIO → *Platform* → **Upload Filesystem Image** (`pio run -t uploadfs`) – only needed when the config changes
3. Restart the board. The serial log prints every loaded key (`[CFG] ...`).

If SPIFFS is empty or the file is missing, built-in defaults are used and the start screen says so.
Partition table: `partitions.csv` (2 × 6.25 MB app, 3.4 MB SPIFFS, core dump).

## Buttons
| Button | Short press | Long press |
|---|---|---|
| BOOT (GPIO0, button 1) | Next page: weather → radiosonde → system info | 3 s: open WiFi setup portal |
| KEY (GPIO14, button 2) | Refresh weather + radiosondes now | 1 s: next brightness level |

Hold **BOOT** while the board starts to erase the stored WiFi credentials.
The info page shows WiFi/RSSI, IP, last update, errors, uptime, battery voltage, memory and config status.

## Web configuration page (v1.2)
After WiFi is connected, open **http://&lt;ip&gt;/** (the IP is shown for a few seconds after boot and on the info page)
or **http://openwx-display.local/**.
- Live status (weather, WiFi, uptime, battery, memory)
- Place search (Open-Meteo geocoding, runs in your browser) → fills name, latitude, longitude
- All config.yaml settings as a form; *Save settings* writes config.yaml to SPIFFS and applies it immediately
- Advanced: edit the raw config.yaml; buttons for *Refresh weather* and *Reboot*
- Optional login: set `web.password` (HTTP basic auth, user `web.user`)

**Initial setup:** the WiFi setup portal (AP `OpenWXS3`, http://192.168.4.1) also asks for location name, latitude and longitude.

## Radiosondes from up to 10 receivers (v1.4)
Receivers are configured in the web panel (System → Settings → *Radiosonde receiver*: Add / Edit / Test)
or in `config.yaml` as sections `rx1:` … `rx10:` (`enabled`, `source`, `host`, `port`, `name`, `user`, `password`).
A v1.3 config with a single `sonde.host` is migrated to `rx1` automatically.

| Receiver | Sondes (every poll) | Info + health (every 60 s) | Default port |
|---|---|---|---|
| OpenWXSDR | `/api/sondes` | `/api/gateway`, `/api/status`, `/api/health` | 5000 |
| OpenWXTTGO | `/api/sondes` | `/live.json`, `/api/status`, `/api/health` | 80 |
| OpenWXDeck | `/api/status` | `/api/config`, `/api/health` | 80 |

All sondes are merged by serial (newest frame wins, "heard by" lists every receiver) and kept 24 h.
The device page shows the newest sonde; *active* = younger than `sonde.max_age_s`, older ones are *recently received*.
Unreachable receivers are retried every 60 s.

## Web panel (v1.4)
- **Menu → Weather**: large weather view (icon, temperature, condition, local clock, min/max, humidity, pressure, wind, 24 h chart)
- **Menu → Radiosonde**: monitor in OpenWXDeck style – map (Leaflet/OSM, needs internet in the browser) with receivers,
  sondes and tracks; sidebar with System Statistics, Active Radiosondes, Recently Received, System Health, Receiver Information
- **System → Settings** (tiles: Now · Location · Weather / Radiosonde · Radiosonde receiver · Display / WiFi · Web access · Time / System),
  **Refresh**, **config.yaml** (editor + download), **Reboot**
- **About**

The page source is `web/index.html`. PlatformIO embeds it (gzip) into `OpenWXStation/webpage.h` before every build
via `tools/embed_web.py`; with the Arduino IDE run `python tools/embed_web.py` after editing.

**Note:** *Upload Filesystem Image* replaces `config.yaml` on the device, including everything saved in the web panel.
Use it for a fresh setup only; otherwise download the current file via System → config.yaml first.
