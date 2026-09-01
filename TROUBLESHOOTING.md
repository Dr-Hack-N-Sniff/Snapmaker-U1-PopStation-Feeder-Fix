# Troubleshooting

## Code 12 — `preload fail: wheel_speed`

Observed logs can show wheel pulse counts increasing while both calculated wheel RPM values remain zero. Repeated zero/low wheel RPM values can trigger the stock wheel-speed validation.

Example pattern:

```text
wheel_speed_a:0, wheel_speed_b:0
...
Raising exception ... code:12 ... preload fail: wheel_speed
```

## Code 11 — `preload fail: motor_speed`

A similar condition was observed for the motor: motor pulse counts increased while calculated motor RPM was zero.

Example pattern:

```text
motor_speed:0
...
Raising exception ... code:11 ... preload fail: motor_speed
```

## Filament Still Stops Short

The tested PopStation setup ultimately required:

```ini
preload_length: 1900
```

for both left and right feeds.

If your filament path differs, treat this as a tested reference rather than a universal value. Increase or decrease cautiously while observing where the filament stops.

## Filament Reaches or Hits the Toolhead Too Hard

Stop testing and reduce the preload distance. The objective is to stage filament just before the toolhead, not force it into the toolhead.

## Problem Returns After Firmware Update

A firmware update may have replaced `filament_feed.py` or configuration values. Follow [BACKUP-RESTORE.md](BACKUP-RESTORE.md) and compare the new firmware logic before reapplying an older workaround.

## `pulse_counter.py`

The final tested workaround used stock `pulse_counter.py`. If you changed that file during troubleshooting, compare it against the appropriate stock firmware version.
