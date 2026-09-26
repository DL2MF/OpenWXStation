<img width="595" height="124" alt="grafik" src="https://github.com/user-attachments/assets/e022ffb9-4a60-47b1-bb65-20e34e792b1f" />


_Based on ["Cheap Internet Weather Station using LilyGo T-Display S3" by Volos Projects.](https://github.com/VolosR/tDisplayS3WeatherStation)_

## ESP32 Internet Weather station project using the LilyGo T-Display S3. 

It's fully integrated into the OpenWX Suite with:
[OpenWXSDR](https://github.com/DL2MF/OpenWXSDR), [OpenWXTTGO](https://github.com/DL2MF/OpenWXTTGO), [OpenWXDeck](https://github.com/DL2MF/OpenWXDeck) 
and [OpenWXFleetMonitor](https://github.com/DL2MF/OpenWX-Fleet-Monitor).

## Parts used:

<img width="638" height="337" alt="grafik" src="https://github.com/user-attachments/assets/0213c594-bdd3-4ea7-9fde-88a7d7996257" />

LilyGO T-Display S3 [https://www.lilygo.cc/](https://lilygo.cc/products/t-display-s3-copy)   


## Integrated Weatherstation configurable for different Weatherservices

<img width="1505" height="1186" alt="grafik" src="https://github.com/user-attachments/assets/6cea8db7-0986-4223-8446-ce85fb2f5b79" />

## Device UI displaying same detailed weather data

<img width="342" height="189" alt="grafik" src="https://github.com/user-attachments/assets/792da0a4-045d-4128-adcb-0ce198115383" />

<img width="340" height="189" alt="grafik" src="https://github.com/user-attachments/assets/c8d904b4-f8ef-4502-b0e1-e1c9d9f5dddc" />

<img width="342" height="191" alt="grafik" src="https://github.com/user-attachments/assets/2574d2a3-e467-4c35-b20a-98b3c63360a1" />

<img width="336" height="189" alt="grafik" src="https://github.com/user-attachments/assets/cbc4def8-d689-4c76-8e3a-af7994a2c310" />


## ... but OpenWXStation is even more - It's a tiny and versatile Station Manager for your Radiosonde Decoder Gateways.

<img width="342" height="191" alt="grafik" src="https://github.com/user-attachments/assets/4407226a-7450-4003-acd5-29d978e589a0" />

<img width="340" height="189" alt="grafik" src="https://github.com/user-attachments/assets/362b44bb-5077-4768-bf77-4b50fc4d0eea" />

https://github.com/user-attachments/assets/18e50a10-9580-46b6-88e5-e9233bd1aef0

##  Local Webserver hosted on T-Display S3 with OpenWXStation

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
or **http://openwx-station.local/**.
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
