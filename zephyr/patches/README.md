# Patches to other west projects

Fixes this application needs in other projects of the workspace, applied
on top of the revisions in `west.yml` with
[`west patch`](https://docs.zephyrproject.org/latest/develop/west/zephyr-cmds.html).
They are listed in `../patches.yml`.

```sh
west update
west patch apply        # after every west update
west patch clean        # undo, e.g. before switching revisions
```

The build fails if the patches are missing (see `../../CMakeLists.txt`).

## zephyr

Both fix the nRF UARTE async driver on legacy targets (nRF91), where line
errors on the AT UART (overruns or framing errors, e.g. at 921600 baud without
flow control) could stop RX for good while TX kept working:

* `0001` resyncs the TIMER-based RX byte count when RX starts. After errors the
  count of reported bytes could get ahead of the hardware count, and the end
  of every later transfer was no longer reported.
* `0002` triggers STOPRX again when RXTO does not follow it. A STOPRX that
  reaches the receiver before RX has started is lost, and the driver waited for
  RXTO forever, so UART_RX_DISABLED was never reported.

To update a patch, change the commit in a Zephyr checkout, regenerate the file
with `git format-patch` and update its `sha256sum` in `patches.yml`.
