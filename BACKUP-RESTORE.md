# Backup, Restore, and Firmware-Update Protection

## Why Backups Matter

The workaround changes a firmware-side Python file and printer configuration. Firmware upgrades may overwrite either change.

## Files to Preserve

At minimum, keep copies of:

- Original `/home/lava/klipper/klippy/extras/filament_feed.py`
- Working modified `filament_feed.py`
- Original printer configuration
- Working printer configuration containing the tested preload settings
- A note recording the firmware version on which the files were captured

Keep these backups somewhere other than the printer.

## Before a Firmware Update

1. Record the currently installed firmware version.
2. Save fresh copies of the working modified files/configuration.
3. Record that the tested PopStation setting is `preload_length: 1900` on both left and right feeds.
4. Assume the update may restore Snapmaker's stock `filament_feed.py`.

## After a Firmware Update

Do **not** automatically overwrite a newer Snapmaker file with an older modified file.

First test stock behavior. Snapmaker may have corrected the underlying issue.

If the problem returns:

1. Compare the new `filament_feed.py` with the version on which the workaround was developed.
2. Determine whether Snapmaker changed the preload/tachometer logic.
3. Reapply only the minimum necessary workaround if it is still applicable.
4. Verify the preload length and all four feeders again.

## Restoring Stock Behavior

To undo the workaround, restore the original firmware-version-matched `filament_feed.py` backup and restore the appropriate stock/configured preload settings.

Avoid restoring a backup from one firmware version over a different firmware version without first comparing the files.
