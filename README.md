# Bluetooth Mouse Disconnect Debug Case Study

This repository documents a real Windows Bluetooth troubleshooting case: a Bluetooth mouse became unreliable, then the Bluetooth icon and switch disappeared after Bluetooth was turned off.

The final working state was restored by fully powering off the laptop, waiting more than 15 seconds, then enabling the Intel Bluetooth adapter again from an elevated PowerShell session.

## Problem

The initial symptom was a Bluetooth mouse that frequently disconnected. After Bluetooth was manually turned off on the laptop, Windows no longer showed a usable Bluetooth icon or switch to turn it back on.

This looked like a UI problem at first, but the system evidence pointed deeper:

- the Bluetooth support service was running
- paired Bluetooth devices still existed in Windows history
- the main Intel Bluetooth adapter was not started
- Windows event logs showed the Bluetooth USB driver had unloaded the adapter after it stopped responding

## Environment

```text
OS family: Windows
Bluetooth adapter: Intel(R) Wireless Bluetooth(R)
Adapter hardware ID: USB\VID_8087&PID_0033
Debug date: 2026-07-09
Timezone: Asia/Taipei
```

The full device instance ID is intentionally not published in this README.

## Key Finding

The decisive event log entry was from `BTHUSB`:

```text
The local Bluetooth adapter has failed in an undetermined manner and will not be used.
The driver has been unloaded.
```

Immediately before that, Windows also logged:

```text
A command sent to the adapter has timed out. The adapter did not respond.
```

That means the Bluetooth mouse was not the root problem. The local Bluetooth adapter itself had stopped responding, so Windows unloaded the driver and the Bluetooth UI disappeared.

## Debugging Path

The investigation followed this path:

1. Checked Bluetooth services.
2. Queried Plug and Play Bluetooth devices.
3. Found `Intel(R) Wireless Bluetooth(R)` in a non-working state.
4. Tried enabling and restarting the adapter with `pnputil`.
5. Opened Windows Bluetooth settings and Device Manager.
6. Checked Windows System event logs.
7. Found `BTHUSB` adapter timeout and driver unload events.
8. Performed a full shutdown and waited more than 15 seconds.
9. Rechecked adapter state after boot.
10. Found the adapter had changed from `Stopped` to `Disabled`.
11. Enabled the adapter from an elevated PowerShell session.
12. Verified the adapter reached `Started`.

## Final Fix

After the full power-off cycle, the adapter was visible but disabled:

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Disabled
```

The adapter was then enabled from an elevated PowerShell session:

```powershell
pnputil /enable-device "USB\VID_8087&PID_0033\..."
```

After enabling, the adapter state became:

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Started
```

The Bluetooth support service was also healthy:

```text
bthserv: Running
```

No new `BTHUSB` Bluetooth errors appeared in the final five-minute event-log check.

## Why Full Shutdown Helped

The first repair attempts failed because the adapter was still in a bad hardware or firmware state. Windows could see the driver history, but the adapter did not respond correctly.

A full shutdown gave the wireless module a chance to lose power and reset. After boot, the adapter was no longer stuck as `Stopped`; it came back as `Disabled`, which Windows could recover from by enabling it again.

## Public Files

- [Medium article draft](medium-bluetooth-mouse-disconnect-debug-article.md)
- [Debug timeline](docs/debug-timeline.md)
- [Final fix](docs/final-fix.md)
- [Privacy notes](docs/privacy.md)

## Privacy Notes

This repository intentionally avoids publishing:

- the full Bluetooth device instance ID
- personal machine names
- full paired-device identifiers
- screenshots containing private device names
- account or user identifiers

## Disclaimer

This is a personal troubleshooting case study. Bluetooth failures can come from drivers, firmware, USB power management, Windows settings, radio interference, or device battery issues. The steps here are useful for reasoning through a similar failure, but they are not a universal repair script.

