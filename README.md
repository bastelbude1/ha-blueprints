# Home Assistant Blueprints

A collection of Home Assistant blueprints for automation.

## Traffic Monitoring with Telegram Notifications

**Version:** 0.1.0
**Blueprint:** `traffic_monitoring.yaml`

Monitor Waze Travel Time sensor and send smart Telegram notifications based on traffic conditions.

### Features

- **Configurable polling interval** - Set how often to check traffic (1-30 minutes)
- **Configurable failsafe threshold** - Define how many failed polls trigger failsafe (1-10 polls)
- **Automatic failsafe duration** - Calculated as polling interval × threshold
- **Flexible day selection** - Choose specific days (Mon-Sun) for automation to run
- **Configurable distance unit** - Choose km or mi to match your Waze sensor
- **Multi-map service support** - Google Maps, OpenStreetMap, Apple Maps, or custom
- **Smart location detection** - Auto-extracts origin/destination from Waze sensor or manual override
- **Integrated failsafe logic** - No separate automations needed
- **Counter-based failsafe** - Tracks consecutive sensor failures
- **Smart notifications** - Only ONE message per traffic state change
- **Three traffic levels** - Heavy, Normal, Light (customizable thresholds)
- **Comprehensive validation** - Runtime checks for entity configuration and thresholds
- **Retry logic** - 3 attempts with 15-second delays per poll
- **Customizable time windows** - Define monitoring start/end times
- **Message templates** - Full control over notification content with multiple placeholders
- **Debugging support** - Works with any numeric sensor for testing

### Installation

**Option 1: One-Click Import**

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fbastelbude1%2Fha-blueprints%2Fmain%2Ftraffic_monitoring.yaml)

**Option 2: Manual Import**

1. Go to **Settings → Automations & Scenes → Blueprints**
2. Click **"Import Blueprint"** button (bottom right)
3. Enter URL: `https://raw.githubusercontent.com/bastelbude1/ha-blueprints/main/traffic_monitoring.yaml`
4. Click **"Preview Blueprint"** then **"Import Blueprint"**

### Requirements

**Home Assistant:** 2021.3+ (for Blueprint support)

**Integrations:**
- Waze Travel Time (or any numeric sensor for debugging/testing)
- Telegram Bot

**Debugging Support:**
When using a non-Waze numeric sensor for testing:
- Disable "Use Waze Sensor Origin/Destination" toggle
- Provide manual origin/destination OR leave empty (map link defaults to https://www.google.com/maps)
- Distance and Route will show "N/A" in messages
- All functionality works normally with just the numeric sensor value

**Helpers (create before using blueprint):**

1. **Traffic State Helper** - `input_select` with these options:
   - none
   - heavy
   - normal
   - light

2. **Failed Attempts Counter** - `input_number`:
   - Min: 0
   - Max: 10
   - Step: 1
   - Initial: 0

**Create helpers via UI:**
- Settings → Devices & Services → Helpers → Create Helper

### Configuration

After importing the blueprint, create a new automation and configure:

**Required:**
- Waze Travel Time Sensor (or any numeric sensor for debugging)
- Traffic State Helper (input_select)
- Failed Attempts Counter (input_number)
- Telegram Chat ID

**Optional (for map fallback link):**
- Use Waze Sensor Origin/Destination (default: true) - Automatically use locations from Waze sensor
- Origin Location (Manual) - Only used if Waze sensor option disabled or attributes unavailable
- Destination Location (Manual) - Only used if Waze sensor option disabled or attributes unavailable
- Map URL Template (default: Google Maps) - Supports Google Maps, OpenStreetMap, Apple Maps, or custom

**Smart Location Detection:**
- If "Use Waze Sensor" enabled: Automatically extracts origin/destination from Waze sensor attributes
- If Waze attributes unavailable: Falls back to manual origin/destination
- If using non-Waze sensor for debugging: Disable "Use Waze Sensor" and provide manual locations

**Optional (with defaults):**
- Monitoring Start Time (default: 07:30:00)
- Monitoring End Time (default: 09:00:00)
- Active Days (default: Mon-Fri) - Select which days automation should run
- Polling Interval (default: 5 minutes)
- Failsafe Threshold (default: 3 failed polls) - **Failsafe duration (polls × interval) must be < 50% of time window**
- Distance Unit (default: km) - Must match your Waze sensor configuration
- Light Traffic Threshold (default: 50 minutes)
- Heavy Traffic Threshold (default: 60 minutes) - **Must be greater than Light Traffic Threshold**
- Message titles and content (customizable)

**Important Validations:**

The blueprint validates configuration at runtime and will stop with a clear error message if:

1. **Entity Configuration:**
   - Waze sensor must exist and have a valid numeric state
   - Traffic State Helper must be an input_select with required options: `none`, `heavy`, `normal`, `light`
   - Failed Attempts Counter must be an input_number

2. **Threshold Configuration:**
   - Heavy Traffic Threshold must be greater than Light Traffic Threshold

3. **Time Window Configuration:**
   - Failsafe duration (threshold × polling interval) must be < 50% of monitoring time window
   - Example: 90-minute window (07:30-09:00) → failsafe duration must be < 45 minutes
   - Valid: 3 polls × 5 min = 15 min ✓
   - Invalid: 10 polls × 5 min = 50 min ✗

### How It Works

**Polling:**
- Triggers at configurable interval (default: every 5 minutes) during monitoring window
- Updates Waze sensor with 3 retry attempts (15s delays per poll)

**Failsafe Logic:**
- If sensor unavailable after 3 retry attempts, increments failure counter
- After reaching failsafe threshold (default: 3 consecutive failed polls), sends failsafe notification
- Only sends failsafe if NO traffic message was sent yet
- Prevents duplicate failsafe messages
- Example: With 5-minute polling and 3-poll threshold = failsafe after 15 minutes

**Traffic Notifications:**
- **Heavy Traffic** - Travel time > Heavy Threshold (default: >60 min)
- **Normal Traffic** - Travel time between Light and Heavy Threshold (default: 50-60 min)
- **Light Traffic** - Travel time < Light Threshold (default: <50 min)
- Only sends when state changes (no duplicates)
- **Important:** Heavy Threshold must be greater than Light Threshold

**State Management:**
- Uses input_select to track current traffic state
- Prevents duplicate notifications
- Daily reset allows fresh notifications

### Message Templates

Messages support placeholders:
- `{travel_time}` - Current travel time in minutes
- `{distance}` - Route distance (numeric value from Waze sensor)
- `{distance_unit}` - Distance unit (km or mi, based on configuration)
- `{route}` - Route description from Waze
- `{failsafe_threshold}` - Number of failed polls configured (failsafe message only)
- `{failsafe_duration}` - Duration in minutes until failsafe triggers (threshold × interval) (failsafe message only)
- `{map_url}` - Map directions link using configured map service and locations (failsafe message only)

**Map Directions Link:**
The `{map_url}` placeholder builds a map directions URL using your configured template, origin, and destination.

**Supported Map Services:**

- **Google Maps** (default):
  ```
  https://www.google.com/maps/dir/?api=1&origin={origin}&destination={destination}
  ```

- **OpenStreetMap**:
  ```
  https://www.openstreetmap.org/directions?from={origin}&to={destination}
  ```

- **Apple Maps**:
  ```
  https://maps.apple.com/?saddr={origin}&daddr={destination}
  ```

- **Custom Map Service**: Use any URL with `{origin}` and `{destination}` placeholders

**Examples:**
- Origin: `123 Main St, City` or `47.4722,7.8753`
- Destination: `456 Oak Ave, Town` or `47.3833,8.4942`
- Result: Opens configured map service with directions

### License

MIT License

### Credits

Educational implementation demonstrating Home Assistant automation patterns.
