# Typhur Bridge

Home Assistant app that connects your **Typhur Sync** thermometer (Sync Quad, Sync Dual, and other WT-series models) directly to Home Assistant using MQTT auto-discovery. No phone or extra tools required — just your Typhur account credentials.

## How it works

The app authenticates with the Typhur cloud API, subscribes to your device's real-time data stream via AWS IoT MQTT, and forwards temperature readings to your local Home Assistant MQTT broker. All sensors are created automatically via HA discovery.

```
Typhur probe  →  Typhur cloud (AWS IoT)  →  Typhur Bridge  →  Local MQTT  →  Home Assistant
```

## Installation

1. Go to **Settings → Apps → Install Apps**
2. Click **Repositories**
3. Add: `https://github.com/oleost/typhurHA`
4. Find **Typhur Bridge** and click **Install**

## Configuration

| Option | Description | Required |
|--------|-------------|----------|
| `typhur_email` | Your Typhur account email | Yes (or use token) |
| `typhur_password` | Your Typhur account password | Yes (or use token) |
| `typhur_token` | API token (advanced — overrides email/password) | No |
| `typhur_region` | Your account region: `eu` (Europe) or `us` (US, CA, AU, NZ) | No (default: `eu`) |
| `typhur_country` | ISO country code sent as the `x-region` header (e.g. `CA`, `AU`, `NZ`). Only needed if your account's country differs from the region default (`US` for `us`, `NO` for `eu`) — otherwise the device list comes back **empty** even though login succeeds. | No |
| `mqtt_host` | HA MQTT broker hostname | Yes (default: `core-mosquitto`) |
| `mqtt_port` | MQTT port | Yes (default: `1883`) |
| `mqtt_username` | MQTT username (if required) | No |
| `mqtt_password` | MQTT password (if required) | No |

**Recommended:** Fill in `typhur_email` and `typhur_password`. The app will log in automatically, cache the token locally, and renew it when it expires — no manual intervention needed.

## Sensors created per device

Probe sensors are created automatically for however many probes your device
reports (2 for the Sync Dual, 4 for the Sync Quad, etc.) — no model setting
needed. A probe's sensors appear the first time it sends a reading.

For each probe:
- **Temperature** (°C)
- **Ambient Temperature** (°C)
- **Target Temperature** (°C) — the cook setpoint; unknown when no target is set
- **Battery** (%)
- **State** (cooking / charging / idle)

For the device itself:
- **Battery** (%)
- **WiFi Signal** (dBm)

## Notes

- **Verified on the Typhur Sync Quad (WT08) and Sync One Pro (WT13).** The Sync
  Dual and other WT-series models should work through the same model-agnostic
  path but are unverified — feedback from other-model owners is very welcome
  (open an issue). The WT13 subscribes on `device/WT13/<id>/pub`, supplied
  directly by the API's `subTopics`.
- Data is routed via Typhur's cloud (AWS IoT). There is no local-only connection.
- The MQTT subscribe topic segment differs per model (`WT08` for the Sync Quad,
  `thermometer` for the Sync Dual). The bridge tries the candidates in order,
  advancing whenever AWS IoT rejects one, and caches the working topic in
  `/data/typhur_topics.json`.
- Certificates are fetched automatically from the Typhur API and cached in `/data/`. They are valid for several years.
- The token is cached in `/data/typhur_token.txt` and refreshed automatically when it expires.
- The MQTT broker endpoint is fetched dynamically from the Typhur API — no hardcoded server addresses.
