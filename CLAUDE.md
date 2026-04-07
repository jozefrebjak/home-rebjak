# Home Assistant Configuration

Instructions for Claude Code when working with Home Assistant.

## Repository Structure

This repo (`home-rebjak`) contains CLI tools and HA configuration for `home.rebjak.com`.
CLI tools (`ha-api`, `ha-ws`, `lovelace-sync`) connect remotely from Mac via REST/WebSocket API.
Commands using the `ha` CLI (like `ha core check`, `ha core restart`) require SSH to HA OS.

```
bin/ha-api              # REST API CLI (bash + curl/jq)
bin/ha-ws               # WebSocket CLI (python + websockets)
bin/lovelace-sync       # Dashboard sync (python + websockets)
homeassistant/          # HA YAML configs (mirrors /config/ on HA OS)
  configuration.yml     # Main config (sensors, templates, utility meters)
  energie_faktury.yaml  # Energy package (prices, invoices, cost estimates)
examples/               # Sample API responses (METAR, Open-Meteo)
install.sh              # Setup script for HA OS
```

## File Structure

YAML configurations are primarily stored in `/config/packages/*.yaml` on HA OS.
In this repo, the equivalent files are in `homeassistant/`.

When searching for configurations, check both `homeassistant/*.yaml` (this repo) and `/config/*.yaml` + `/config/packages/` (on HA OS).

Configurations and automations should be in self-contained files in the packages/ folder.

## Environment (.env)

Both `ha-api` and `ha-ws` search for `.env` in this order:
1. `$HA_ENV_FILE` (if set)
2. `/config/.env` (standard HA OS location)
3. `/homeassistant/.env`
4. Current directory (`./.env`)
5. `~/.ha-cli.env`

Required variables:
```bash
HA_URL=https://172.30.32.1:8128   # Internal HA OS URL (default)
HA_TOKEN=your_long_lived_access_token_here
```

## Getting Entity/Device Information

**Prefer `ha-ws` for detailed lookups** - it provides full context:

```bash
# Get comprehensive entity info (registry, state, related automations/scenes)
ha-ws entity get <entity_id>

# Get comprehensive device info (all entities, related automations/scenes)
ha-ws device get <device_id>

# Use --json for full raw data
ha-ws --json entity get sensor.temperature
```

For quick state checks, `ha-api` is faster:
```bash
ha-api state light.kitchen      # Just the state value
ha-api attr sensor.temperature  # Just attributes
```

## REST API CLI Tool

Use `ha-api` for REST API interactions:

```bash
ha-api <command> [options]
```

### Commands

| Command | Description |
|---------|-------------|
| `states [filter]` | List all entities (optional grep filter) |
| `state <entity_id>` | Get state of specific entity |
| `domains` | List entity counts by domain |
| `devices <device_class>` | Find entities by device_class (motion, door, temperature, etc.) |
| `search <pattern>` | Search entity IDs by pattern |
| `call <domain> <service>` | Call a service |
| `attr <entity_id>` | Show all attributes of an entity |
| `history <entity_id> [hrs]` | Get history for entity (default: 24 hours) |
| `get <endpoint>` | GET any API endpoint |
| `post <endpoint> [json]` | POST to any API endpoint with optional JSON body |

### Examples

```bash
ha-api devices motion           # Find all motion sensors
ha-api devices occupancy        # Find occupancy sensors
ha-api search kitchen           # Find entities containing "kitchen"
ha-api state light.living_room  # Get current state
ha-api attr sensor.temperature  # Show all attributes
ha-api call light turn_on       # Call a service
```

### Generic API Access

For any API endpoint not covered by the built-in commands:

```bash
ha-api get config                    # GET /api/config
ha-api get events                    # GET /api/events
ha-api get services                  # GET /api/services
ha-api get error_log                 # GET /api/error_log
ha-api post services/light/turn_on '{"entity_id":"light.living_room"}'
```

## Checking Configuration

Before reloading, always validate the configuration.
**Note:** `ha` is the HA OS CLI — requires SSH to HA OS (not available locally from Mac).

```bash
ha core check --no-progress --raw-json
```

Expected success response: `{"result":"ok","data":{}}`

## Reloading YAML Configuration

After making changes to YAML files, ask the user if they want to reload the configuration.

To reload all YAML-configured domains:

```bash
ha-api call homeassistant reload_all
```

Returns `[]` on success.

### Selective Reloads

| Domain | Command |
|--------|---------|
| Automations | `ha-api call automation reload` |
| Scripts | `ha-api call script reload` |
| Scenes | `ha-api call scene reload` |
| Groups | `ha-api call group reload` |
| Input booleans | `ha-api call input_boolean reload` |
| Input numbers | `ha-api call input_number reload` |
| Input selects | `ha-api call input_select reload` |
| Input texts | `ha-api call input_text reload` |
| Timers | `ha-api call timer reload` |
| Template entities | `ha-api call template reload` |

### HomeKit

HomeKit picks up entity changes (adds/removes) with just a YAML reload - no integration reload or restart needed.

## WebSocket CLI Tool

Use `ha-ws` for registry management and WebSocket API operations:

```bash
ha-ws [--json] [--quiet|-q] <command> [options]
```

### Options

| Flag | Description |
|------|-------------|
| `--json` | Output as raw JSON |
| `--quiet`, `-q` | Minimal output |

### Registry Commands

```bash
# Entity Registry
ha-ws entity list [filter]              # List entities (filter by ID, name, platform, area)
ha-ws entity get <entity_id>            # Get entity details + state + related items
ha-ws entity update <id> key=value...   # Update entity
ha-ws entity remove <entity_id>         # Remove entity

# Device Registry
ha-ws device list [filter]              # List devices (filter by name, manufacturer, model, area)
ha-ws device get <device_id>            # Get device details + all entities + related items
ha-ws device update <id> key=value...   # Update device
ha-ws device remove <device_id>         # Remove device

# Area Registry
ha-ws area list                         # List all areas
ha-ws area create <name>                # Create new area
ha-ws area update <id> key=value...     # Update area
ha-ws area delete <area_id>             # Delete area
```

### State & Service Commands

```bash
ha-ws state <entity_id>                 # Get current state + attributes
ha-ws states [domain]                   # List states (optional domain filter)
ha-ws call <domain>.<service> [data]    # Call a service
ha-ws services [domain]                 # List available services
```

### Search & System Commands

```bash
ha-ws search <entity_id>               # Find entities related to this one
ha-ws related <entity_id>              # Alias for search
ha-ws config                            # Get HA configuration (location, version, timezone)
ha-ws info                              # Get HA core info
```

### Batch Mode

Read commands from stdin (one per line, lines starting with `#` are skipped):

```bash
echo -e "state sensor.temperature\nstate sensor.humidity" | ha-ws batch
```

### Raw WebSocket

For any WebSocket message type:

```bash
ha-ws raw config/entity_registry/list
ha-ws raw config/core/info
```

### Examples

```bash
# Rename an entity
ha-ws entity update light.old_name new_entity_id=light.new_name

# Set entity icon and clear name (inherit from device)
ha-ws entity update light.lamp icon=mdi:floor-lamp name=none

# Set device area and name
ha-ws device update abc123 area_id=kitchen name_by_user="Kitchen Light"

# Call a service
ha-ws call light.turn_on entity_id=light.kitchen brightness=255

# Get state with JSON output
ha-ws --json state sensor.temperature
```

### Value Syntax

| Syntax | Result |
|--------|--------|
| `key=value` | String |
| `key=123` | Integer |
| `key=true` / `key=false` | Boolean |
| `key=null` / `key=none` | None (clears field) |
| `key="quoted string"` | String with spaces |

## Full Restart (if needed)

For changes that require a full restart (**requires SSH to HA OS**):

```bash
ha core restart --no-progress --raw-json
```

## Lovelace Dashboard Sync

The `.storage/lovelace` file is managed by HA in memory. Direct file edits won't take effect until synced.

**Workflow:**
```bash
# 1. Edit the lovelace file directly
# 2. Push to HA (no restart needed)
lovelace-sync
# 3. Refresh browser/app to see changes
```

Prefer direct JSON edits over writing Python scripts - simpler and faster.

## Template Entities (Modern Syntax)

**IMPORTANT:** Always use the modern `template:` syntax. The legacy `platform: template` syntax is deprecated.

### Modern Format

```yaml
template:
  - sensor:
      - name: "My Sensor"
        unique_id: my_sensor
        unit_of_measurement: "W"
        icon: mdi:gauge
        state: >
          {{ states('sensor.other')|float + 10 }}
    binary_sensor:
      - name: "My Binary Sensor"
        unique_id: my_binary_sensor
        delay_on:
          minutes: 5
        state: >
          {{ states('sensor.value')|int > 100 }}
```

### Key Differences from Legacy

| Legacy (Deprecated) | Modern |
|---------------------|--------|
| `sensor: platform: template` | `template:` with `sensor:` list |
| `value_template:` | `state:` |
| `icon_template:` | `icon:` |
| `friendly_name:` | `name:` |
| `delay_on: '01:30:00'` | `delay_on: { hours: 1, minutes: 30 }` |
