# Home Assistant Blueprints

A collection of Home Assistant blueprints for automation.

## Traffic Monitoring with Telegram Notifications

**Version:** 0.1.0
**Blueprint:** `traffic_monitoring.yaml`

Monitor Waze Travel Time sensor and send smart Telegram notifications based on traffic conditions.

### Features

- **Configurable polling interval** - Set how often to check traffic (1-30 minutes)
- **Configurable failsafe threshold** - Define how many failed polls trigger failsafe (1-10 polls)
- **Flexible day selection** - Choose specific days (Mon-Sun) for automation to run
- **Integrated failsafe logic** - No separate automations needed
- **Counter-based failsafe** - Tracks consecutive sensor failures
- **Smart notifications** - Only ONE message per traffic state change
- **Three traffic levels** - Heavy, Normal, Light (customizable thresholds)
- **Retry logic** - 3 attempts with 15-second delays per poll
- **Customizable time windows** - Define monitoring start/end times
- **Message templates** - Full control over notification content

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
- Waze Travel Time Sensor
- Traffic State Helper (input_select)
- Failed Attempts Counter (input_number)
- Telegram Chat ID

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
- `{google_maps_link}` - Fallback link (failsafe message only)

### License

MIT License

### Credits

Educational implementation demonstrating Home Assistant automation patterns.
