# Husqvarna Automower Control4 Gateway Driver

A Control4 driver that integrates Husqvarna Automower robotic lawn mowers using the official Husqvarna Automower Connect API. Provides full control and monitoring capabilities through the Control4 ecosystem with an embedded web interface for real-time mower tracking.

## Overview

This gateway driver connects to the Husqvarna Automower Connect API and manages multiple robotic mowers within a Control4 system. It handles OAuth2 authentication, automatic token refresh, status polling, and provides both Control4 native controls and a standalone web-based map interface.

### Key Features

- OAuth2 authentication with automatic token refresh
- Multi-mower support with individual device drivers
- Real-time status monitoring and GPS tracking
- Embedded web server with interactive map interface
- Rate limit management for API compliance
- Configurable polling intervals
- Debug mode for troubleshooting
- Cached and live data modes for web interface

## System Requirements

- Control4 OS 3.3.0 or later
- Husqvarna Automower with Connect module
- Active Husqvarna developer account with API credentials
- Network connectivity for API access

## Architecture

The driver consists of two components:

1. **Gateway Driver** (this driver) - Handles API communication, authentication, and data aggregation
2. **Child Device Drivers** - One per mower, provides Control4 UI and controls

```
Control4 Controller
    |
    +-- Husqvarna Gateway Driver (API communication)
    |       |
    |       +-- Embedded Web Server (port 1122)
    |       +-- OAuth2 Token Management
    |       +-- Rate Limit Control
    |       |
    |       +-- Child: Mower 1 Device Driver
    |       +-- Child: Mower 2 Device Driver
    |       +-- Child: Mower N Device Driver
```

## Installation

### 1. Obtain API Credentials

Register for Husqvarna developer access:

1. Go to https://developer.husqvarnagroup.cloud/
2. Create an account and login
3. Create a new application
4. Note your Application Key (client_id) and Application Secret (client_secret)
5. Set redirect URI to: `https://localhost/callback`
6. Request scopes: `iam:read` and `amc:api`

### 2. Install Driver

1. Open Composer Pro
2. Navigate to Drivers section
3. Click "Add Driver" > "Browse"
4. Select `husqvarna_gateway_v2.26.c4z`
5. Driver will appear in system tree

### 3. Configure Authentication

In driver properties:
1. Set **Application Key** (your client_id)
2. Set **Application Secret** (your client_secret)
3. Save properties

### 4. Authenticate

1. In Actions menu, run **Start Authentication**
2. Copy the displayed URL
3. Open URL in web browser
4. Authorize the application
5. Copy the authorization code from redirect URL
6. In Actions menu, run **Complete Authentication**
7. Paste the authorization code
8. Execute action

Driver will authenticate and automatically discover mowers.

### 5. Add Child Drivers

After discovery:
1. Note discovered mower IDs in logs
2. Add child device drivers for each mower
3. Configure each with corresponding Mower ID
4. Child drivers will now display status and accept commands

## Configuration

### Properties

**Application Key**
- Description: OAuth2 client ID from Husqvarna developer portal

**Application Secret**
- Description: OAuth2 client secret from Husqvarna developer portal

**Polling Interval**
- Default: 180 seconds
- Range: 120-600 seconds
- Note: Must comply with API rate limits

**Web Data Mode**
- Default: Cached (Recommended)

**Enable Polling**
- Default: Enabled

**HTTP Port**
- Default: 1122

### Actions

**Start Authentication**
- Generates OAuth2 authorization URL
- Displays URL

**Complete Authentication**
- Exchanges authorization code for access token

**Discover Mowers**
- Populates mower list

**Refresh Token**

**Test Connection**
- Verifies API connectivity

## API Rate Limits

Husqvarna enforces strict rate limits:

- **Weekly Limit**: 21,000 requests per week
- **Average**: ~2 requests per minute
- **Burst**: 120 requests per minute

### Rate Limit Compliance

With 4 mowers, recommended polling intervals:

| Interval | Calls/Hour | Calls/Week | Limit Usage |
|----------|------------|------------|-------------|
| 120s     | 120        | 20,160     | 96% |
| 180s     | 80         | 13,440     | 64% (recommended) |
| 240s     | 60         | 10,080     | 48% |
| 300s     | 48         | 8,064      | 38% |

Formula: `(3600 / interval) * mower_count * 24 * 7`

### Rate Limit Management

The driver includes automatic rate limit handling:

1. **Detection**: Identifies 403 errors from rate limiting
2. **Backoff**: Stops API calls for 5 minutes
3. **Warning**: Logs recommendations for interval adjustment
4. **Recovery**: Automatically resumes after backoff period

### Web Data Modes

**Cached Mode (Recommended)**
- Web interface serves from cached data
- No API calls triggered by web requests
- Ideal for multiple users or frequent refreshes
- Data freshness: polling interval

**Live Mode**
- Triggers API refresh if cache older than 30 seconds
- May contribute to rate limit exhaustion
- Best for single user with infrequent access
- Automatically backs off if rate limited

## Web Interface

Embedded web server provides real-time map view of all mowers.

### Access

Open browser to: `http://[controller-ip]:1122`

Default port: 1122 (configurable in properties)

### API Endpoint

**GET /api/mowers**

Returns JSON with current mower status:

```json
{
  "mowers": [
    {
      "id": "mower-uuid",
      "name": "Front Lawn",
      "latitude": "40.987720",
      "longitude": "-74.070023",
      "activity": "MOWING",
      "state": "IN_OPERATION",
      "battery": 85,
      "connected": true
    }
  ],
  "count": 4,
  "timestamp": "2026-04-27 14:30:00",
  "cacheAge": "2 minutes ago",
  "cacheSeconds": 120
}
```

## Mower Commands

Available commands (sent through child device drivers):

- **Start** - Begin mowing
- **Pause** - Pause current operation
- **Park** - Return to charging station
- **Park Until Next Schedule** - Park and wait for next scheduled run
- **Park Until Further Notice** - Park indefinitely
- **Resume Schedule** - Resume automatic scheduling


## Token Management

### Authentication Flow

1. User initiates authentication
2. Driver generates OAuth2 authorization URL
3. User authorizes in browser
4. Husqvarna redirects with authorization code
5. Driver exchanges code for access token and refresh token
6. Tokens stored in driver properties

### Automatic Refresh

- Access tokens expire after 24 hours
- Driver automatically refreshes 5 minutes before expiry
- Refresh scheduled via Control4 timer
- Refresh token used for obtaining new access token
- On failure, manual re-authentication required

### Token Storage

Tokens are persisted in driver properties:
- **Access Token** (read-only property)
- **Refresh Token** (read-only property)
- **Token Expiry** (read-only property, human-readable timestamp)

## Troubleshooting

### 403 Forbidden Errors

**Cause**: Rate limit exceeded or authentication issue

**Solution**:
1. Check Polling Interval (should be 180+ seconds)
2. Verify Web Data Mode is "Cached"
3. Enable Debug Mode to see error details
4. Wait 5 minutes for rate limit backoff
5. If persistent, re-authenticate

### No Mowers Discovered

**Cause**: Authentication issue or API connectivity

**Solution**:
1. Run "Test Connection" action
2. Verify Application Key and Secret are correct
3. Check network connectivity
4. Re-run "Discover Mowers" action
5. Check Debug Mode logs for errors

### Token Refresh Failures

**Cause**: Refresh token expired or invalid

**Solution**:
1. Obtain new authorization code from Husqvarna portal
2. Run "Complete Authentication" with new code
3. Verify Application Secret is correct
4. Check that app has correct scopes (iam:read, amc:api)

### Web Map Not Updating

**Cached Mode**:
- Data updates every polling interval
- Verify polling is enabled
- Check that mowers were discovered successfully

**Live Mode**:
- May be in rate limit backoff
- Check Debug Mode for API call logs
- Consider switching to Cached mode

## Debug Mode

Enable Debug Mode in properties to see:

- HTTP request/response details
- Token values (first 20 characters)
- API call timing and results
- Rate limit detection and backoff
- Web server request logging
- Cache update timestamps

Disable Debug Mode for production use to reduce log verbosity.

## Version History

### 2.26 (Current)
- Added rate limit management and detection
- Implemented web data caching
- Added configurable web data modes (Cached/Live)
- Enforced safe polling intervals (120-600s)
- Added API usage estimation and warnings
- Added cache age tracking and display
- Improved startup diagnostics

### 2.25
- Enhanced 403 error debugging
- Added comprehensive token logging
- Fixed scope URL encoding
- Added header verification logging

### 2.24
- Fixed token refresh scope parameter issue
- Removed scope from refresh requests
- Enhanced debugging for token issues

### 2.23
- Added conditional debug logging
- Implemented debug mode toggle
- Reduced production log verbosity

### 2.22
- Fixed token refresh scheduling
- Improved error handling

## Technical Details

### API Endpoints

**OAuth2**
- Authorization: `https://api.authentication.husqvarnagroup.dev/v1/oauth2/authorize`
- Token: `https://api.authentication.husqvarnagroup.dev/v1/oauth2/token`

**Automower Connect API**
- Base URL: `https://api.amc.husqvarna.dev/v1`
- Mower List: `GET /mowers`
- Mower Status: `GET /mowers/{id}`
- Mower Actions: `POST /mowers/{id}/actions`

### Required Headers

All API requests include:
- `Authorization: Bearer {access_token}`
- `Authorization-Provider: husqvarna`
- `Content-Type: application/vnd.api+json`
- `X-Api-Key: {application_key}`

### Data Structures

**STATE Object** (Lua table):
```lua
{
  authenticated = boolean,
  accessToken = string,
  refreshToken = string,
  tokenExpiryTime = number (unix timestamp),
  mowerList = array of {id, name},
  mowerData = table[mower_id] = {
    latitude = string,
    longitude = string,
    activity = string,
    state = string,
    battery = number,
    connected = boolean
  },
  lastCacheUpdate = number (unix timestamp),
  rateLimitHit = boolean,
  rateLimitUntil = number (unix timestamp)
}
```

### Polling Mechanism

Status polling uses Control4's timer system:
1. `C4:AddTimer(interval, "SECONDS", true)` creates recurring timer
2. `OnTimerExpired(timer_id)` callback triggered
3. `FetchAllMowerData()` executes API calls
4. Cache timestamp updated on completion
5. Timer continues until disabled

### Web Server

Single-file embedded HTTP server:
- Implemented using Control4's `C4:CreateServer(port)` API
- Handles GET requests for HTML and JSON
- No external dependencies
- Stateless request handling
- Supports query parameters

## Support and Resources

**Husqvarna Developer Portal**
- https://developer.husqvarnagroup.cloud/

**Husqvarna API Documentation**
- https://developer.husqvarnagroup.cloud/apis/automower-connect-api


### Event Handlers

Required Control4 callbacks:
- `OnPropertyChanged(property_name)` - Property updates
- `OnTimerExpired(timer_id)` - Polling timer
- `OnServerConnectionStatusChanged(handle, port, status)` - Web server
- `OnServerDataIn(handle, data)` - Web requests
- `ExecuteCommand(command, params)` - Action execution

## Known Limitations

1. **Rate Limits**: Strict Husqvarna API limits require careful interval tuning
2. **OAuth2 Flow**: Requires manual browser interaction for initial auth
3. **Token Expiry**: Refresh token eventually expires, requiring re-authentication
4. **Polling Only**: No push notifications or webhooks available
5. **Web Server**: Basic HTTP only, no HTTPS support
6. **Concurrent Requests**: No request queuing or throttling beyond rate limit backoff

## Performance Considerations

- API latency typically 200-500ms per request
- Token refresh scheduled async, does not block polling
- Web requests served from cache (Cached mode) have <10ms response time
- Live mode web requests may take 200-500ms for API fetch
- Multiple simultaneous mower updates execute in parallel
