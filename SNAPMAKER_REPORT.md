# Snapmaker U1 Firmware 1.6 — False Feeder Speed Abnormality During Filament Preload

## Technical Bug Report

Reproducible across all four feeder channels.

## Summary

After updating to firmware 1.6, feeder wheel RPM can report `0` while pulse counts continue increasing. The stock preload logic then raises code 12 (`wheel_speed` / Speed Abnormality). The same behavior was reproduced on multiple channels and when feeders were connected directly to the U1, not only through the BIQU/BIGTREETECH PopStation Mini.

## Environment

- Printer: Snapmaker U1
- Firmware: 1.6
- Accessory: BIQU/BIGTREETECH PopStation Mini
- Channels tested: Extruder 0, 1, 2, and 3
- Result before workaround: Repeatable Speed Abnormality during preload
- Direct-U1 test: Same behavior observed after firmware 1.6, removing PopStation as the sole cause

## Problem Description

After upgrading the U1 to firmware 1.6, filament preload began failing with `Speed Abnormality`. The feeder motor runs and filament physically moves, but the preload routine can reject wheel-speed telemetry and raise exception code 12.

## Log Evidence

### Wheel RPM zero while counts increase

```text
[feed_preload] extruder[0], start, wheel_cnt_a: 0, wheel_cnt_b: 2, motor_cnt: 0
[feed_preload] extruder[0], wheel speed error, wheel_speed_a:0, wheel_speed_b:0, motor_speed:13134
[feed_preloading] extruder[0], wheel, cnt_a_1:0, cnt_b_1:2, cnt_a_2:20, cnt_b_2:22,
wheel_speed_a:0, wheel_speed_b:0, motor, motor_cnt_1:0, motor_cnt_2:978, motor_speed:13567
Raising exception: id:525 index:0 code:12 ... message: preload fail: wheel_speed
```

Wheel pulse counters increased while both calculated wheel RPM values remained zero.

### Motor RPM zero while counts increase

```text
[feed_preload] extruder[0], motor speed error, motor_speed:0
[feed_preloading] extruder[0], wheel, cnt_a_1:2, cnt_b_1:2, cnt_a_2:87, cnt_b_2:87,
wheel_speed_a:0, wheel_speed_b:0, motor, motor_cnt_1:0, motor_cnt_2:16718, motor_speed:0
Raising exception: id:525 index:0 code:11 ... message: preload fail: motor_speed
```

Motor pulse count increased from 0 to 16,718 while calculated motor RPM was zero.

## Relevant Firmware Logic

File:

`/home/lava/klipper/klippy/extras/filament_feed.py`

Relevant constants recorded during investigation:

```python
FEED_MOTOR_SLIP_RATE = 0.7
FEED_MOTOR_REDUCTION_R = 33.0
FEED_WHEEL_CIRCUMFERENCE = 31.4159
FEED_PRELOAD_WHEEL_ERR_CNT_MAX = 3
FEED_PRELOAD_MOTOR_ERR_CNT_MAX = 2
```

The wheel-speed validation compares both wheel RPM readings with motor RPM. Repeated low/zero wheel RPM readings lead to `FEED_ERR_WHEEL_SPEED` and exception code 12.

Also investigated:

`/home/lava/klipper/klippy/extras/pulse_counter.py`

The observed frequency calculation included a path where a non-positive `delta_time` results in `_freq = 0.`. `FeedTachometer.get_rpm()` converts frequency to RPM, so zero frequency becomes zero RPM.

## Tested Workaround

### A. Bypass faulty wheel-RPM comparison during preload

```python
if False and wheel_speed_a * FEED_MOTOR_REDUCTION_R < motor_speed * (1 - FEED_MOTOR_SLIP_RATE) and \
        wheel_speed_b * FEED_MOTOR_REDUCTION_R < motor_speed * (1 - FEED_MOTOR_SLIP_RATE):
```

### B. Add motor-count fallback for preload distance

Recorded elements of the tested fallback:

```python
motor_cnt_2 = self.motor_tachometer.get_counts()
```

```python
(motor_cnt_2 - motor_cnt_1) / self.motor_tachometer.ppr / FEED_MOTOR_REDUCTION_R > self._feed_preload_counts
```

This preserves a distance-based stop when wheel RPM/count feedback is unreliable.

### C. Calibrated PopStation preload length

```ini
[filament_feed left]
preload_length: 1900

[filament_feed right]
preload_length: 1900
```

## Result

- All four feeder channels tested multiple times.
- All four feeders stage filament consistently just before the toolhead.
- False `Speed Abnormality` / `wheel_speed` failures no longer occur during normal preload.
- Filament does not overfeed or slam into the toolhead.
- `pulse_counter.py` is restored to stock.
- Motor-speed monitoring, filament detection, timeout, and preload-distance protections remain active.
- Only wheel RPM/slip validation during preload is intentionally bypassed.

## Requested Investigation

Please investigate firmware 1.6 changes around feeder tachometer sampling and preload validation, particularly:

1. `FrequencyCounter` timing and `count_time` handling.
2. Cases where `delta_time <= 0` forces `_freq = 0`.
3. Interaction between preload polling cadence and tachometer sample/report intervals.
4. Why wheel and motor pulse counts can increase while calculated RPM remains zero.
5. Whether firmware 1.6 preload wheel-speed validation introduced or exposed a sampling regression.

## Suggested Ticket Subject

**U1 Firmware 1.6: false feeder Speed Abnormality — tach counts increase while RPM reports 0**
