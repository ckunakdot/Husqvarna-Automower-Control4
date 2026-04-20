# Husqvarna Automower Control4 Driver

Professional Control4 integration for Husqvarna Automower Connect robotic lawn mowers. Control your mowers through Control4 Navigator, Composer automation, and a beautiful web-based map interface.

## Features

### Core Functionality
- **OAuth2 Authentication** - Secure authentication through Husqvarna Developer Portal
- **Automatic Token Refresh** - Set-it-and-forget-it authentication (tokens auto-refresh every 24 hours)
- **Multi-Mower Support** - Control unlimited mowers from a single gateway
- **Real-time Status** - Live updates of mower activity, battery, position, and errors
- **Full Command Control** - Start, pause, park, resume, and override schedules
- **Navigator Integration** - Control mowers directly from Control4 touch panels and mobile apps
- **Composer Automation** - Trigger mowing schedules based on weather, time, or other conditions

### Web Map Interface
- **Interactive Map** - View all mowers on satellite or street map (port 1122)
- **Real-time Tracking** - Live position updates with GPS coordinates
- **Position History** - Visual trails showing mowing patterns (last 100 positions)
- **Multi-Mower Switching** - Quick links and URL parameters to switch between mowers
- **Color-Coded Markers** - Each mower gets a unique color (Green, Blue, Red, Orange, Purple)
- **Battery Monitoring** - Visual battery level with color-coded status
- **Activity Badges** - See at-a-glance mower status (Mowing, Charging, Parked, etc.)
- **Satellite/Street Toggle** - Switch between Esri satellite imagery and OpenStreetMap
- **Auto-Refresh** - Updates every 10 seconds

## What You Need

### Hardware
- Control4 controller (any model with network access)
- Husqvarna Automower with **Automower® Connect** or **Automower® Connect Module**
  - Models: 310, 315, 405, 415, 430X, 435X AWD, 450X, 535 AWD, 550, and NERA series
  - Must have cellular/WiFi connectivity to Husqvarna cloud

### Accounts
- **Husqvarna Account** - The same login you use for the Automower® Connect mobile app
- **Husqvarna Developer Portal Account** - Free account at https://developer.husqvarnagroup.cloud

### Software
- Control4 OS 3.3.0 or newer
- ComposerPro (for installation and configuration)

## Installation

### 1. Set Up Husqvarna Developer Portal

This is **critical** - most authentication issues come from incorrect portal setup.

**Step 1: Create Account**
1. Go to https://developer.husqvarnagroup.cloud
2. Sign in with your Husqvarna account (same as mobile app)
3. Authorize the Developer Portal when prompted

**Step 2: Create Application**
1. Click **"My Applications"** → **"Create New Application"**
2. Fill in:
   - **Application name**: `Control4` (or anything you want)
   - **Description**: Optional
   - **Redirect URLs**: `https://localhost/callback` (must be HTTPS!)
3. Click **"CREATE"**

**Step 3: Connect BOTH APIs** ⚠️ **CRITICAL STEP**
1. Click **"CONNECT NEW API"** button
2. Select **"Authentication API"** → Click **"Connect"**
3. Click **"CONNECT NEW API"** again
4. Select **"Automower Connect API"** → Click **"Connect"**

**You MUST have both APIs connected or commands will fail with HTTP 403 errors!**

**Step 4: Copy Credentials**
1. Copy **Application Key** (36 characters)
2. Copy **Application Secret** (36 characters)
3. Save both - you'll need them for driver configuration

### 2. Install Control4 Driver

**Gateway Driver (Required - Install First)**
1. Download `husqvarna_gateway_v2.25_FINAL.c4z`
2. In ComposerPro: **System Design** → **Drivers**
3. Right-click project → **Add Driver** → **Browse** → Select the .c4z file
4. Driver appears as **"Husqvarna Automower Gateway"**

**Child Drivers (One Per Mower)**
1. Download `husqvarna_child_v5.2_FixedActionNames.c4z`
2. Add to project same way as gateway
3. Add one child driver per mower you own
4. Children will auto-populate with mower data after authentication

### 3. Configure Gateway

**In ComposerPro Properties Tab:**
1. **Application Key**: Paste from developer portal
2. **Application Secret**: Paste from developer portal
3. **Polling Interval**: `60` (seconds between status updates)
4. **Debug Mode**: `false` (set to `true` only for troubleshooting)

**Network Configuration:**
- Driver runs a web server on **port 1122**
- Ensure this port is not blocked by firewall
- Access map at: `http://CONTROLLER_IP:1122`

### 4. Authenticate

**From Actions Tab in ComposerPro:**

1. **Click "Start Authentication"**
   - Logs will show a URL starting with `https://api.authentication.husqvarnagroup.dev...`
   - Copy this entire URL

2. **Paste URL in Browser**
   - Sign in to Husqvarna when prompted
   - You'll be redirected to `https://localhost/callback?code=XXXXX...`
   - Browser will show security warning (expected - ignore it)
   - Copy the **code** parameter from the URL (long hex string)

3. **Click "Complete Authentication"**
   - Paste the code when prompted
   - Logs should show: `★★★ AUTHENTICATION SUCCESSFUL! ★★★`
   - Mowers will be discovered automatically

**Expected Success Output:**
```
Authentication successful!
Token expires in: 86399 seconds
★ FOUND 4 MOWER(S)! ★
  1. Front Lawn
  2. Back Lawn
  3. Side Yard
  4. Echo Ridge
Status polling started
✓ Initial data for Front Lawn: 41.1380, -73.9656
```

## Using the Driver

### Control4 Navigator

**From Touch Panels/Mobile App:**
- Select mower from device list
- Tap **"Start Mowing"** - Resumes normal schedule
- Tap **"Pause"** - Pauses in current location
- Tap **"Park"** - Returns to charging station until next schedule
- Tap **"Park Indefinitely"** - Parks until manually resumed
- Tap **"Override 1hr/3hr"** - Mows for 1 or 3 hours regardless of schedule

**Status Display:**
- Battery level (0-100%)
- Current activity (Mowing, Charging, Parked, etc.)
- Current state (In Operation, Restricted, Error, etc.)
- Connection status (Online/Offline)

### Composer Automation

**Example: Start Mowing on Sunny Days**
```
WHEN: Weather changes to Sunny
  AND Time is 9:00 AM
  THEN: Front Lawn → Start Mowing
```

**Example: Park During Rain**
```
WHEN: Weather changes to Rain
  THEN: Front Lawn → Park Indefinitely
```

**Available Commands:**
- `START_MOWING` - Resume schedule
- `PAUSE` - Pause in place
- `PARK` - Park until next schedule
- `PARK_INDEFINITELY` - Park until manual resume
- `OVERRIDE_1H` - Mow for 1 hour
- `OVERRIDE_3H` - Mow for 3 hours
- `RESUME_SCHEDULE` - Resume normal schedule
- `REFRESH` - Force status update

### Web Map Interface

**Access Map:**
```
http://YOUR_CONTROLLER_IP:1122
```

**Multi-Mower URLs:**
```
http://192.168.1.100:1122?mower=0  → First mower (Green)
http://192.168.1.100:1122?mower=1  → Second mower (Blue)
http://192.168.1.100:1122?mower=2  → Third mower (Red)
http://192.168.1.100:1122?mower=3  → Fourth mower (Orange)
```

**Map Features:**
- Click mower icon to see popup with name and status
- Toggle between Satellite and Street view (top-right)
- Quick links at bottom to switch between mowers
- Position history shows as colored trail when mowing
- Auto-refreshes every 10 seconds

**Bookmark URLs for Quick Access:**
- Save each `?mower=N` URL as separate bookmark
- Open multiple browser tabs side-by-side to monitor all mowers
- Share URLs with family members

## Troubleshooting

### Authentication Fails (HTTP 403)

**Symptom:** `HTTP error 403` or `Discovery failed: code=403`

**Cause:** Missing Automower Connect API connection in developer portal

**Fix:**
1. Go to https://developer.husqvarnagroup.cloud
2. Click **"My Applications"** → Select your app
3. Check **"Connected APIs"** section
4. Should show:
   - ✅ Authentication API
   - ✅ Automower Connect API
5. If Automower Connect API is missing:
   - Click **"CONNECT NEW API"**
   - Select **"Automower Connect API"**
   - Click **"Connect"**
6. **Important:** After reconnecting APIs, regenerate Application Key/Secret and update driver properties
7. Re-run authentication process

### Commands Return HTTP 403

**Symptom:** Mowers discovered successfully, but commands fail with HTTP 403

**Cause:** OAuth2 token has `iam:read` scope but missing `amc:api` scope

**Fix:**
- This should be automatic in v2.25+
- If using older version, upgrade to v2.25
- Re-authenticate after upgrade
- Token should include both scopes: `["iam:read", "amc:api"]`

### Mowers Not Discovered

**Check:**
1. Developer portal has both APIs connected
2. Application Key and Secret are correct in driver properties
3. Authentication completed successfully (check logs)
4. Mowers are online in Husqvarna mobile app
5. Run **"Discover Mowers"** action manually

### Map Shows 404 Not Found

**Symptom:** Browser shows 404 when accessing `http://IP:1122?mower=1`

**Cause:** Gateway v2.19 and earlier had path matching bug with query parameters

**Fix:**
- Upgrade to v2.20 or later
- Query parameters now work correctly

### Map Icon Has Black Background

**Symptom:** Mower icon shows with black square background

**Fix:**
- Upgrade to v2.25 (transparent icon version)
- Icon background is now transparent

### Token Expires After 24 Hours

**Symptom:** Driver stops working after ~24 hours, requires re-authentication

**Cause:** Auto-refresh failing (should work automatically in v2.21+)

**Fix:**
1. Upgrade to v2.21 or later
2. Token refresh includes proper scope parameter
3. Should auto-refresh every 24 hours without intervention

### Web Server Port Conflict

**Symptom:** Map not accessible, logs show port binding error

**Fix:**
1. Port 1122 is in use by another application
2. Identify conflicting service: `netstat -an | grep 1122`
3. Stop conflicting service or change driver to use different port
4. No built-in port configuration currently - requires code modification

## Version History

### v2.25 (Current) - April 2026
- **Transparent mower icon** - Removed black background from map markers
- Clean look on satellite imagery
- All v2.24 features preserved

### v2.24 - April 2026
- **Embedded mower icon** - Fixed runtime image loading
- Professional Husqvarna Automower image on map
- Color-coded borders (Green/Blue/Red/Orange/Purple)

### v2.22 - April 2026
- **OAuth2 scope fixes** - CRITICAL UPDATE
- Added `amc:api` scope to both initial auth and token refresh
- Fixes HTTP 403 errors on commands
- Requires fresh authentication after update

### v2.20 - April 2026
- **URL parameter support** - Multi-mower map navigation
- Fixed query parameter path matching bug
- Access each mower via `?mower=0`, `?mower=1`, etc.

### v2.16 - April 2026
- **Satellite map view** - Esri World Imagery
- Street/Satellite layer toggle
- Improved map interface

### v2.8 - April 2026
- **Token auto-refresh** - 24-hour automatic renewal
- Navigator visibility fixes
- v2.8 Experience Button model

### v5.2 Child Driver - April 2026
- **Correct API action names** - All 9 commands working
- Fixed command format for Husqvarna API
- Navigator control integration

### v5.0 Child Driver - April 2026
- Icon cycling button
- Configurable polling intervals
- Multiple icon states

## API Reference

### Husqvarna Automower Connect API

**Base URLs:**
- Authentication: `https://api.authentication.husqvarnagroup.dev`
- Mower API: `https://api.amc.husqvarna.dev/v1`

**Authentication Flow:**
1. User authorization: `/v1/oauth2/authorize`
2. Token exchange: `/v1/oauth2/token` (grant_type: authorization_code)
3. Token refresh: `/v1/oauth2/token` (grant_type: refresh_token)

**Required Scopes:**
- `iam:read` - Account information access
- `amc:api` - Mower control and status

**Endpoints Used:**
- `GET /mowers` - List all mowers
- `GET /mowers/{id}` - Get mower details
- `POST /mowers/{id}/actions` - Send commands

**Action Types:**
- `Start` - Start mowing (with optional duration)
- `Pause` - Pause mower
- `ParkUntilNextSchedule` - Park until next scheduled start
- `ParkUntilFurtherNotice` - Park indefinitely
- `ResumeSchedule` - Resume normal schedule

**Rate Limits:**
- 10,000 API calls per month per application
- Recommended polling: 60 second intervals
- Batch status requests where possible

## Technical Details

### Driver Architecture

**Gateway Driver:**
- Handles OAuth2 authentication and token management
- Discovers and maintains list of all mowers
- Polls Husqvarna API for status updates (configurable interval)
- Runs HTTP server on port 1122 for web map interface
- Proxies commands from child drivers to Husqvarna API

**Child Drivers:**
- One instance per mower
- Receives status updates from gateway via proxy
- Sends commands to gateway for execution
- Integrates with Control4 Navigator and Composer
- Maintains local state for Navigator display

**Communication Flow:**
```
Child Driver → Gateway Driver → Husqvarna Cloud API
     ↓              ↓                    ↓
Navigator     Web Server           Mower (via cellular)
```

### Web Map Technology

**Stack:**
- Backend: Lua HTTP server (built-in C4:url() functions)
- Frontend: HTML5 + JavaScript (ES5 compatible)
- Maps: Leaflet.js 1.9.4
- Tiles: Esri World Imagery (satellite) + OpenStreetMap (street)

**API Endpoint:**
- `GET /` - Serves interactive map HTML
- `GET /api/mowers` - Returns JSON with all mower data
- `GET /?mower=N` - Same map HTML, JS selects mower N

**JSON Format:**
```json
{
  "mowers": [
    {
      "id": "a0c6103f-d6d3-4789-9c28-96b6e7d06e5b",
      "name": "Front Lawn",
      "latitude": "41.1380716",
      "longitude": "-73.9656333",
      "activity": "MOWING",
      "state": "IN_OPERATION",
      "battery": 99,
      "connected": true
    }
  ],
  "count": 1,
  "timestamp": "2026-04-20 14:30:00"
}
```

### Data Refresh Strategy

**Status Polling:**
- Configurable interval (default 60 seconds)
- Gateway polls Husqvarna API for all mowers
- Updates cached in STATE table
- Children receive updates via proxy notifications

**Web Map Refresh:**
- JavaScript polls `/api/mowers` every 10 seconds
- Uses cached data from gateway (no direct API calls)
- Position history maintained client-side (100 points max)

**Command Execution:**
- Immediate API call when command received
- Response returned to child driver
- Status update triggered 5 seconds after command
- Ensures Navigator shows updated state quickly

## Supported Mower Models

**Automower® Connect (Built-in Cellular)**
- 310, 315, 405, 415
- 430X, 435X AWD
- 450X, 535 AWD, 550
- 320 NERA, 430X NERA, 450X NERA

**Automower® Connect Module (Accessory)**
- Most older models can be upgraded with Connect Module
- Check Husqvarna website for compatibility

**NOT Supported:**
- Bluetooth-only models
- Models without cellular/WiFi connectivity
- Gardena robotic mowers (different API)

## Activity States

**Mower Activities:**
- `UNKNOWN` - Status not available
- `NOT_APPLICABLE` - Not in active state
- `MOWING` - Currently cutting grass
- `GOING_HOME` - Returning to charging station
- `CHARGING` - Docked and charging
- `LEAVING` - Leaving charging station
- `PARKED_IN_CS` - Parked in charging station
- `STOPPED_IN_GARDEN` - Stopped outside charging station

**Mower States:**
- `UNKNOWN` - Status not available
- `NOT_APPLICABLE` - Not in active state
- `PAUSED` - Paused by user
- `IN_OPERATION` - Operating normally
- `WAIT_UPDATING` - Firmware update in progress
- `WAIT_POWER_UP` - Powering up
- `RESTRICTED` - Restricted by schedule or zone
- `OFF` - Turned off
- `STOPPED` - Stopped (check error code)
- `ERROR` - Error state (check error code)
- `FATAL_ERROR` - Critical error
- `ERROR_AT_POWER_UP` - Error during power up

## FAQ

**Q: Can I control mowers from multiple Control4 systems?**
A: No. Only one Control4 system can authenticate with the API credentials at a time. Use a single gateway and share access via Navigator app.

**Q: How many mowers can I control?**
A: Unlimited. The driver supports as many mowers as your Husqvarna account has registered.

**Q: Do I need separate API applications for each mower?**
A: No. One application in the developer portal provides access to all mowers on your account.

**Q: Can I use this with Gardena mowers?**
A: No. Gardena uses a different API and is not compatible with this driver.

**Q: Will this work outside my home network?**
A: Web map requires access to your Control4 controller's IP. Navigator app works anywhere via Control4's cloud services.

**Q: Does the mower need to be online to see the map?**
A: Yes. The driver requires cellular/WiFi connectivity between the mower and Husqvarna cloud.

**Q: Can I change the map refresh interval?**
A: Web map is hardcoded to 10 seconds. Gateway polling is configurable in driver properties.

**Q: What happens if my token expires?**
A: v2.21+ automatically refreshes tokens every 24 hours. Manual re-authentication only needed if refresh fails.

**Q: Can I customize the map colors?**
A: Not via driver properties. Colors are defined in the HTML/JavaScript and require code modification.

**Q: Does this work with EPOS (GPS-guided) mowers?**
A: Yes. EPOS models work the same as standard models. GPS position data is available for both.

**Q: What's the API rate limit?**
A: 10,000 calls per month per application. At 60-second polling with 4 mowers, you use ~5,760 calls/month.

## Credits

Created for the Control4 community by passionate smart home enthusiasts.

**Developed with:**
- Husqvarna Automower Connect API
- Control4 DriverWorks SDK
- Leaflet.js mapping library
- Esri World Imagery tiles
- OpenStreetMap tiles

## Support

**For Issues:**
- Check Troubleshooting section above
- Enable Debug Mode in driver properties
- Check ComposerPro logs for detailed error messages
- Verify developer portal API connections

**Common Log Messages:**
- `★★★ AUTHENTICATION SUCCESSFUL! ★★★` - Auth worked
- `★ FOUND N MOWER(S)! ★` - Discovery succeeded
- `HTTP error 403` - Missing API connection in portal
- `attempt to index a nil value` - Driver code error, update to v2.25
- `Token refresh scheduled in X seconds` - Auto-refresh working

---

**Current Version: 2.25** | **Last Updated: April 2026**
