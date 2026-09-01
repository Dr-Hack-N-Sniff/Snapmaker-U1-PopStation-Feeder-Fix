# Installation / Workaround Procedure

## Read This First

This procedure is based on the tested Snapmaker U1 firmware 1.6 workaround. It is unofficial and modifies firmware-side files. Back up the original files and configuration before making changes.

This repository intentionally does not invent shell commands that were not recorded during testing. Use the access method you normally use to safely edit files on your U1.

## 1. Back Up the Original

Before editing, preserve the original copy of:

`/home/lava/klipper/klippy/extras/filament_feed.py`

Also back up the active printer configuration containing the `[filament_feed left]` and `[filament_feed right]` sections.

See [BACKUP-RESTORE.md](BACKUP-RESTORE.md).

## 2. Bypass Wheel-RPM Validation During Preload

Relevant logic is in:

`/home/lava/klipper/klippy/extras/filament_feed.py`

The tested workaround prevents the wheel-speed comparison from entering its error path during preload. The modified condition begins:

```python
if False and wheel_speed_a * FEED_MOTOR_REDUCTION_R < motor_speed * (1 - FEED_MOTOR_SLIP_RATE) and \
        wheel_speed_b * FEED_MOTOR_REDUCTION_R < motor_speed * (1 - FEED_MOTOR_SLIP_RATE):
```

This intentionally bypasses only the wheel RPM/slip validation implicated in the false code-12 failure.

## 3. Add Motor-Count Fallback for Preload Distance

The tested workaround also uses motor tachometer counts so preload still has a distance-based stopping mechanism when wheel RPM/count feedback is unreliable.

The recorded logic includes:

```python
motor_cnt_2 = self.motor_tachometer.get_counts()
```

and a distance comparison based on:

```python
(motor_cnt_2 - motor_cnt_1) / self.motor_tachometer.ppr / FEED_MOTOR_REDUCTION_R > self._feed_preload_counts
```

The source technical report does not contain the complete surrounding function, indentation, or exact insertion location. For that reason this repository does **not** provide a blind copy/paste replacement for the complete function.

## 4. Set the Tested PopStation Preload Length

The tested configuration was:

```ini
[filament_feed left]
preload_length: 1900

[filament_feed right]
preload_length: 1900
```

The value `1900` worked on the tested PopStation filament path and allowed all four feeders to stage filament just before the toolhead.

## 5. Restart and Test Carefully

After applying the changes using your normal U1 service/restart procedure:

1. Test one feeder first.
2. Watch the filament during the complete preload operation.
3. Confirm it stops just before the toolhead rather than driving hard into it.
4. Check the logs for code 11 or code 12 preload errors.
5. Repeat for all four feeder channels.
6. Run multiple preload tests on each channel before considering the workaround validated on your machine.

## Do Not Modify `pulse_counter.py` as Part of the Final Workaround

During investigation, `pulse_counter.py` was examined, but the final tested state restored it to stock.
