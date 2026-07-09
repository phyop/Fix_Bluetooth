# Final Fix

## Working Recovery Sequence

The working sequence was:

1. Fully shut down the laptop.
2. Wait more than 15 seconds.
3. Power the laptop on.
4. Check the Intel Bluetooth adapter state.
5. Enable the adapter from an elevated PowerShell session.
6. Confirm that the adapter status is `Started`.

## Command Used

The adapter was enabled with:

```powershell
pnputil /enable-device "USB\VID_8087&PID_0033\..."
```

The published command intentionally truncates the full device instance ID.

## Verified Result

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Started
```

The Bluetooth support service was also running:

```text
Name: bthserv
Status: Running
StartType: Manual
```

## Why It Worked

Before the full shutdown, Windows had unloaded the Bluetooth driver after the adapter stopped responding.

The full shutdown allowed the wireless module to reset. After reboot, Windows saw the adapter as `Disabled` instead of `Stopped`. That state could be repaired by enabling the adapter again.

## When To Try Driver Reinstall

If the adapter returns to `Stopped`, repeatedly logs `BTHUSB` timeout events, or disappears again after reboot, the next likely step is to reinstall or update the Intel Bluetooth driver from the laptop vendor or Intel.

