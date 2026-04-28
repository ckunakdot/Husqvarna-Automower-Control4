# Husqvarna Automower Control4 Driver

<img src="https://img.shields.io/badge/Control4-OS%203.x-red" alt="Control4 OS 3.x"> <img src="https://img.shields.io/badge/Control4-OS%204.x-blue" alt="Control4 OS 4.x">

<a href="https://www.buymeacoffee.com/ckunakdot" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png" alt="Buy Me A Coffee" style="height: 41px !important;width: 174px !important;box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;-webkit-box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;" ></a>

A Control4 driver that integrates Husqvarna Automower robotic lawn mowers using the official Husqvarna Automower Connect API. Provides full control and monitoring capabilities through the Control4 ecosystem with **dynamic Navigator icons**, **automatic status updates**, and an embedded web interface for real-time mower tracking.

### Key Features

**Gateway Driver (v2.27)**
- OAuth2 authentication with automatic token refresh
- Multi-mower support with automatic discovery
- Real-time status monitoring and GPS tracking
- Embedded web server with interactive map interface
- **Automatic status broadcasting to child devices every 3 minutes**
- Rate limit management for API compliance
- Configurable polling intervals (120-600 seconds)
- Cached and live data modes for web interface

**Child Device Driver (v2.26)**
- **Dynamic Navigator icons that change based on mower activity**
- Individual mower control and status display
- 8 icon states: idle, mowing, charging, parked, paused, leaving, stopped, error
- Cycling button behavior for quick command access
- Event triggers for programming automation
- Battery level and GPS location display

## System Requirements

- Control4 3.3.0+ 
- Husqvarna Automower with Connect module
- Active Husqvarna developer account with API credentials
- Network connectivity for API access

## Quick Start

### 1. Get API Credentials

1. Go to https://developer.husqvarnagroup.cloud/
2. Create account → Create application
3. Note your **Application Key** and **Application Secret**
4. Set redirect URI: `https://localhost/callback`
5. Request scopes: `iam:read` and `amc:api`

### 2. Install & Configure Gateway

1. Install `husqvarna_gateway_v2.27.c4z`
2. Set Application Key and Secret in properties
3. Run "Start Authentication" → copy URL
4. Authorize in browser → copy code
5. Run "Complete Authentication" → paste code
6. Gateway auto-discovers mowers

### 3. Install Child Drivers

1. Check gateway "Discovered Mowers" property
2. Add child driver for each mower
3. Set Mower ID (copy from gateway)
4. Connect binding 999 to gateway binding 1
5. **Refresh Navigator** (File → Refresh Navigator)
6. Force close & reopen Navigator apps

Done! Icons update automatically every 3 minutes.

## Dynamic Navigator Icons

The child driver displays different icons based on real-time mower activity:

| Icon | Activity | Description |
|------|----------|-------------|
| 🟢 Mowing | MOWING | Actively cutting grass |
| 🟠 Charging | CHARGING | Charging in station |
| ⚫ Parked | PARKED_IN_CS | Parked in charging station |
| 🔵 Leaving | GOING_HOME, LEAVING | Traveling to/from station |
| 🟡 Paused | PAUSED | Temporarily stopped |
| 🔴 Stopped | STOPPED_IN_GARDEN | Stopped outside station |
| ⚫ Idle | NOT_APPLICABLE | Idle, no activity |
| 🔴 Error | Error Code Present | Mower has error |

**Update Frequency:** Automatic every 3 minutes

## Web Map Interface

Access: `http://[controller-ip]:1122`

**Features:**
-  Interactive map (OpenStreetMap/Satellite)
-  Real-time mower positions
-  Battery levels and activity status
-  Mower selector dropdown
-  Cache age display

**Individual Mower URLs:**
- `?mower=0` (first mower)
- `?mower=1` (second mower)
- `?mower=2` (third mower)

## API Rate Limits

| Mowers | Interval | Calls/Week | % Limit |
|--------|----------|------------|---------|
| 1 | 180s | ~3,360 | 16% |
| 2 | 180s | ~6,720 | 32% |
| 3 | 180s | ~10,080 | 48% |
| 4 | 180s | ~13,440 | 64% |
| 5+ | 240s | Varies | Adjust |

**Limit:** 21,000 requests/week

**Recommended:** 180+ seconds polling interval

## Troubleshooting

### Icons Not Changing

1. File → Refresh Navigator
2. Force close Navigator apps
3. Verify binding 999 → gateway binding 1
4. Enable Debug Mode, check logs

### No Status Updates

- Check binding 999 connected
- Verify Mower ID matches exactly
- Gateway must be polling (check "Last Update")
- Look for "STATUS_UPDATE" in logs

### Rate Limited

- Increase polling interval (240s or 300s)
- Set Web Data Mode to "Cached"
- Wait 5 minutes for auto-recovery

Provided as-is. Husqvarna and Automower are trademarks of Husqvarna Group. Not affiliated with or endorsed by Husqvarna Group.

Use at your own risk. Always supervise robotic lawn mowers and follow the manufacturer’s safety guidelines. We are not responsible for any injury or damage.
