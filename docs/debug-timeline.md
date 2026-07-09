# Debug Timeline

## 1. Initial Symptom

Bluetooth became unstable. The first visible symptom was a Bluetooth peripheral disconnecting repeatedly. After Bluetooth was turned off on the laptop, Windows no longer showed a Bluetooth icon or switch that could turn it back on.

## 2. Service Check

The Bluetooth support service was checked first:

```text
bthserv: Running
```

This showed that the service layer was not the primary failure.

## 3. Bluetooth Adapter Query

The Bluetooth Plug and Play device list showed the main adapter:

```text
Intel(R) Wireless Bluetooth(R)
Status: Stopped
```

Paired devices and Bluetooth profiles were visible, but the main adapter state was the important clue.

## 4. First Enable Attempt

An attempt was made to enable the Intel Bluetooth adapter with `pnputil`.

Windows reported that the device was already enabled, but the adapter still did not start:

```text
Device is already enabled.
Status: Stopped
```

This meant the configuration state and runtime state did not match.

## 5. Restart Attempt

An attempt was made to restart the adapter:

```text
Failed to restart device.
Access is denied.
```

Restarting the Bluetooth support service also did not solve the issue, because the adapter itself was stuck.

## 6. Windows Settings And Device Manager

The Windows Bluetooth settings page and Device Manager were opened to confirm the user-facing state and provide a manual recovery path.

The Bluetooth switch was not enough because Windows did not have a started Bluetooth adapter.

## 7. Event Log Evidence

The System event log showed `BTHUSB` warnings and errors:

```text
A command sent to the adapter has timed out. The adapter did not respond.
```

Then:

```text
The local Bluetooth adapter has failed in an undetermined manner and will not be used.
The driver has been unloaded.
```

This confirmed that the local Bluetooth adapter was the root failure.

## 8. Full Shutdown

The laptop was fully shut down and left powered off for more than 15 seconds.

The goal was to reset the wireless module at the hardware power-state level.

## 9. Post-Boot State

After boot, the adapter changed from `Stopped` to:

```text
Intel(R) Wireless Bluetooth(R)
Status: Disabled
```

This was progress because the adapter was no longer stuck in the previous failed runtime state.

## 10. Elevated Enable

The adapter was enabled from an elevated PowerShell session:

```powershell
pnputil /enable-device "USB\VID_8087&PID_0033\..."
```

## 11. Final Verification

Final state:

```text
Intel(R) Wireless Bluetooth(R): Started
bthserv: Running
```

No new Bluetooth `BTHUSB` errors appeared in the final event-log check.