# Husqvarna Automower Control4 Driver

<img src="https://img.shields.io/badge/Control4-OS%203.x-red" alt="Control4 OS 3.x"> <img src="https://img.shields.io/badge/Control4-OS%204.x-blue" alt="Control4 OS 4.x">

<a href="https://www.buymeacoffee.com/ckunakdot" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png" alt="Buy Me A Coffee" style="height: 41px !important;width: 174px !important;box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;-webkit-box-shadow: 0px 3px 2px 0px rgba(190, 190, 190, 0.5) !important;" ></a>

A Control4 driver that integrates Husqvarna Automower robotic lawn mowers using the official Husqvarna Automower Connect API. Provides full control and monitoring capabilities through the Control4 ecosystem with **dynamic Navigator icons**, **automatic status updates**, **weather-based rain protection**, and an embedded web interface for real-time mower tracking.

## Key Features

**Gateway Driver (v2.27)**
- OAuth2 authentication with automatic token refresh
- Multi-mower support with automatic discovery
- Real-time status monitoring and GPS tracking
- Embedded web server with interactive map interface
- Automatic status broadcasting to child devices every 3 minutes
- Rate limit management for API compliance
- Configurable polling intervals (120-600 seconds)
- Cached and live data modes for web interface

**Child Device Driver (v2.26)**
- **Dynamic Navigator icons that change based on mower activity**
- **Automatic weather-based rain protection** (NEW in v2.26)
- Individual mower control and status display
- 8 icon states: idle, mowing, charging, parked, paused, leaving, stopped, error
- Cycling button behavior for quick command access
- Event triggers for programming automation
- Battery level and GPS location display

## What's New in v2.26

**Weather-Based Automation**
- Automatically parks mower when rain is detected
- Automatically resumes when rain stops
- Uses free OpenWeatherMap API (1,000 calls/day)
- Checks weather every 5 minutes
- Parks indefinitely to prevent resuming in ongoing rain
- Manual "Check Weather Now" action for testing

## System Requirements

- Control4 3.3.0+ 
- Husqvarna Automower with Connect module
- Active Husqvarna developer account with API credentials
- Network connectivity for API access
- (Optional) Free OpenWeatherMap API account for weather features

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

### 4. Setup Weather Protection (Optional)

1. Get free API key from https://openweathermap.org/api
2. In child driver properties:
   - **Weather API Key** → Paste your API key
   - **Weather Latitude** → Your location (e.g., 40.7128)
   - **Weather Longitude** → Your location (e.g., -74.0060)
   - **Weather Control** → Set to "Enabled"
3. Run "Check Weather Now" action to test

## Weather-Based Automation

Automatically protects your mower from rain damage by parking when rain is detected and resuming when it's clear.

### How It Works

| Condition | Behavior |
|-----------|----------|
| Rain/Drizzle/Thunderstorm | Parks mower indefinitely |
| Clear/Clouds/Mist | Resumes normal schedule |
| Check Frequency | Every 5 minutes |

**Smart Parking Logic:**
- When rain detected → Parks until further notice (NOT until next schedule)
- When rain stops → Resumes schedule automatically
- Won't start mowing in ongoing rain events
- Perfect for multi-day rain periods

### API Usage

- **Free Tier:** 1,000 calls/day
- **Driver Usage:** ~288 calls/day per mower
- **Cost:** FREE (well within limits)
- **Multiple Mowers:** 4 mowers = ~1,152 calls/day (still free!)

## Dynamic Navigator Icons

The child driver displays different icons based on real-time mower activity:

| Icon | Activity | Description |
|------|----------|-------------|
| <img width="271" height="163" alt="Mowing Icon" src="https://github.com/user-attachments/assets/a07a6188-9c1f-4e90-b72f-9f34c2cfafe2" />| MOWING | Actively cutting grass |
| <img width="267" alt="Charging Icon" src="https://github.com/user-attachments/assets/ab45abe3-7f7a-4b0c-a9bb-556b82828046" /> | CHARGING | Charging in station |
| <img width="263" alt="Parked Icon" src="https://github.com/user-attachments/assets/24d15c08-15f1-4be6-99a2-14fdab514c67" /> | PARKED_IN_CS | Parked in charging station |
| <img width="271" height="163" alt="Leaving Icon" src="https://github.com/user-attachments/assets/58152dfc-39c9-4046-b8df-ec0dbb68162a" /> | GOING_HOME, LEAVING | Traveling to/from station |
| Paused | PAUSED | Temporarily stopped |
| Stopped | STOPPED_IN_GARDEN | Stopped outside station |
| <img width="270" alt="Idle Icon" src="https://github.com/user-attachments/assets/b86bc4bd-cf29-4680-9022-2f04df99a0cb" /> | NOT_APPLICABLE | Idle, no activity |
| <img width="272" alt="Error Icon" src="https://github.com/user-attachments/assets/93732d47-e238-4274-8d61-f98f5d788572" /> | Error Code Present | Mower has error |

**Update Frequency:** Automatic every 3 minutes

## Web Map Interface

<img width="878" alt="Map Screenshot" src="https://github.com/user-attachments/assets/1ebe1dbe-a6fe-4828-9ebf-7e247db2d830" />

Access: `http://[controller-ip]:1122`

**Features:**
- Interactive map (OpenStreetMap/Satellite)
- Real-time mower positions
- Battery levels and activity status
- Mower selector dropdown
- Cache age display

**Individual Mower URLs:**
- `?mower=0` (first mower)
- `?mower=1` (second mower)
- `?mower=2` (third mower)

## Properties

### Child Driver Properties

| Property | Description |
|----------|-------------|
| Mower ID | UUID from gateway (required) |
| Driver Version | Current version (2.26) |
| Status | Activity and battery display |
| Debug Mode | Enable verbose logging |
| **Weather Settings** | |
| Weather Control | Enable/Disable rain protection |
| Weather API Key | OpenWeatherMap API key |
| Weather Latitude | Location latitude |
| Weather Longitude | Location longitude |
| Weather Condition | Current weather (read-only) |

## Available Actions

**Control Commands**
- Start Mowing
- Pause
- Park
- Resume
- Park Until Next Schedule
- Park Until Further Notice
- Override Schedule 3h

**Utility Actions**
- Refresh Status
- Test Icon Cycle
- Check Weather Now (NEW)

## Programming Events

| Event | ID | When It Fires |
|-------|-----|---------------|
| Mowing Started | 100 | Mower begins cutting |
| Paused | 101 | Mower is paused |
| Resumed | 102 | Mower resumes |
| Parked | 103 | Returns to station |
| Charging | 104 | Charging in station |
| Error | 200 | Error reported |

**Example:** Turn on lights when mower parks at night
```
When: Automower → Parked
Conditions: Sunset to Sunrise
Actions: Outside Lights → On
```

## API Rate Limits

| Mowers | Interval | Calls/Week | % Limit |
|--------|----------|------------|---------|
| 1 | 180s | ~3,360 | 16% |
| 2 | 180s | ~6,720 | 32% |
| 3 | 180s | ~10,080 | 48% |
| 4 | 180s | ~13,440 | 64% ✅ |
| 5+ | 240s | Varies | Adjust |

**Limit:** 21,000 requests/week

**Recommended:** 180+ seconds polling interval

## Troubleshooting

### Icons Not Changing

1. File → Refresh Navigator
2. Force close Navigator apps
3. Verify binding 999 → gateway binding 1
4. Enable Debug Mode, check logs

### Weather Not Working

- Check API Key is valid (no spaces)
- Check Latitude/Longitude are numbers
- Enable Debug Mode
- Run "Check Weather Now"
- Look for "missing API key or location" or "bad URL format"

### No Status Updates

- Check binding 999 connected
- Verify Mower ID matches exactly
- Gateway must be polling (check "Last Update")
- Look for "STATUS_UPDATE" in logs

### Rate Limited

- Increase polling interval (240s or 300s)
- Set Web Data Mode to "Cached"
- Wait 5 minutes for auto-recovery

## Technical Details

**Communication:**
- Gateway → Husqvarna API every 180 seconds (default)
- Gateway → All children via STATUS_UPDATE broadcast
- Children update Navigator icons automatically
- Weather checks every 300 seconds (5 minutes)

**Weather API:**
- Provider: OpenWeatherMap
- Endpoint: `/data/2.5/weather`
- Format: JSON (simple pattern matching, no libraries)
- Triggers: Rain, Drizzle, Thunderstorm

**Bindings:**
- Child Binding 999 → Gateway Binding 1 (status updates)
- Child Binding 5001 → Internal (Navigator icons)

## Version History

**v2.26 (Current)**
- Added weather-based rain protection
- Automatic park on rain detection
- Automatic resume when rain stops
- Check Weather Now action
- Weather properties and status display

**v2.25**
- Fixed command mapping for device actions
- Added command name normalization
- Improved programming action compatibility

**v2.24**
- Fixed dynamic Navigator icons
- Added all 8 icon states with proper sizing
- Fixed XML structure for icon display

## License

Provided as-is. Husqvarna and Automower are trademarks of Husqvarna Group. Not affiliated with or endorsed by Husqvarna Group.

Use at your own risk. Always supervise robotic lawn mowers and follow the manufacturer's safety guidelines. We are not responsible for any injury or damage.

## Support

For issues, feature requests, or questions:
- Enable Debug Mode and check Lua Output
- Review embedded documentation in Composer Pro
- Check gateway and child logs for errors

Weather feature uses OpenWeatherMap API - see https://openweathermap.org/api for API documentation and support.
