# simple_gps Design

## Overview

**simple_gps** provides location services for Eiffel applications using Windows Location API.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_GPS | Main facade |
| GPS_POSITION | Location data |
| GPS_DEVICE | GPS device info |

## Key Features

```eiffel
-- Get current position
if attached gps.current_position as pos then
    print ("Lat: " + pos.latitude.out)
    print ("Lon: " + pos.longitude.out)
    print ("Accuracy: " + pos.accuracy.out + " meters")
end

-- Continuous tracking
gps.on_position_changed (agent handle_position)
gps.start_tracking

-- later
gps.stop_tracking
```

## GPS Position

| Property | Type | Description |
|----------|------|-------------|
| latitude | REAL_64 | Degrees (-90 to 90) |
| longitude | REAL_64 | Degrees (-180 to 180) |
| altitude | REAL_64 | Meters above sea level |
| accuracy | REAL_64 | Horizontal accuracy (meters) |
| speed | REAL_64 | Speed (m/s) |
| heading | REAL_64 | Heading (0-360 degrees) |
| timestamp | DATE_TIME | Time of fix |

## Windows API

- Location API: `ILocation`, `ILocationReport`
- Geolocation: `Windows.Devices.Geolocation`

## Phase 1 Scope

1. Get current position
2. Position accuracy
3. Continuous tracking
4. Position change events

## Dependencies

- locationapi.lib
