# When the Bluetooth Icon Disappears: Debugging a Windows Intel Bluetooth Adapter That Stopped Responding

A Bluetooth mouse started disconnecting repeatedly. That was annoying, but still easy to mistake for an ordinary mouse problem: maybe low battery, maybe interference, maybe a flaky pairing.

Then the real symptom appeared.

After Bluetooth was turned off on the laptop, the Bluetooth icon and switch disappeared. There was no obvious way to turn Bluetooth back on from the usual Windows UI.

That changed the shape of the problem. This was no longer just "my mouse disconnects." It became:

> Windows no longer has a working Bluetooth radio to expose in the UI.

This article walks through the debugging path, the false starts, the useful evidence, and the final fix.

## Start With The Boundary

The first question was simple: is Bluetooth broken at the service layer, the device layer, or the mouse layer?

The Bluetooth support service was running:

```text
bthserv: Running
```

That mattered. If the service had been stopped, the fix might have been as simple as starting it again. But the service was already healthy.

Next, Windows Plug and Play data showed the main adapter:

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Stopped
Driver Name: oem239.inf
```

That shifted the suspicion away from the mouse. The paired devices were mostly disconnected, but the important failure was the local adapter itself.

## The UI Was Telling The Truth

When the Bluetooth icon disappears, it can feel like Windows is hiding a setting.

In this case, the UI was not the root problem. The UI disappeared because Windows did not have a started Bluetooth adapter available.

Opening Bluetooth settings was still useful because it confirmed the user-facing symptom. But the real evidence came from Plug and Play state and the System event log.

## The Event Log Gave The Actual Failure

The Windows System log contained the key `BTHUSB` error:

```text
The local Bluetooth adapter has failed in an undetermined manner and will not be used.
The driver has been unloaded.
```

Immediately before that, Windows logged adapter timeouts:

```text
A command sent to the adapter has timed out. The adapter did not respond.
```

This explained the whole pattern:

1. The mouse was disconnecting because the Bluetooth adapter was unstable.
2. Bluetooth was turned off manually.
3. The adapter did not recover cleanly.
4. Windows unloaded the Bluetooth driver.
5. The Bluetooth UI disappeared because the local radio was no longer started.

That is a very different problem from replacing a mouse battery or pairing the device again.

## What Did Not Fix It

Several reasonable repair attempts did not work at first.

Restarting the Bluetooth support service was not enough, because the service was not the broken layer.

Trying to restart the device with `pnputil` failed:

```text
Failed to restart device
Access is denied.
```

Trying to enable the device also initially failed or reported that the device was already enabled, while the adapter still remained unusable.

That contradiction was important. A device can be "enabled" in configuration while still not started at runtime. In other words, Windows may allow the device in principle, but the driver and hardware can still be stuck.

## The Turning Point: Full Power-Off

The most useful physical step was a full shutdown, waiting more than 15 seconds, and then powering the laptop on again.

This was not just a normal restart. The goal was to let the wireless module fully reset instead of keeping part of the hardware state alive across a fast boot or warm reboot.

After the full power-off cycle, the adapter state changed:

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Disabled
```

That was progress.

Before shutdown, the adapter was stuck as `Stopped`. After shutdown, it was visible as `Disabled`. A disabled device is still not working, but it is a cleaner state: Windows can often enable it again.

## The Final Fix

From an elevated PowerShell session, the adapter was enabled:

```powershell
pnputil /enable-device "USB\VID_8087&PID_0033\..."
```

After that, Plug and Play showed:

```text
Device Description: Intel(R) Wireless Bluetooth(R)
Status: Started
```

The Bluetooth service was still running:

```text
bthserv: Running
```

A final event-log check showed no new Bluetooth `BTHUSB` errors in the last few minutes.

That was the validation point. The goal was not simply to make the icon appear once. The goal was to confirm that Windows had a started adapter, a running service, and no immediate new driver unload error.

## The Debugging Lesson

The important lesson is to separate Bluetooth into layers:

- the Bluetooth device, such as a mouse or keyboard
- the Bluetooth adapter, such as Intel Wireless Bluetooth
- the Windows Bluetooth service
- the Plug and Play driver state
- the hardware power state

If only one mouse disconnects, start with the mouse: battery, pairing, interference, distance, and device firmware.

But if the Bluetooth icon disappears and Windows has no switch to turn Bluetooth on, look lower:

1. Is `bthserv` running?
2. Does Windows list a Bluetooth adapter?
3. Is the adapter `Started`, `Stopped`, or `Disabled`?
4. Do System logs show `BTHUSB` timeout or driver unload events?
5. Does a full power-off reset the adapter state?
6. Can the adapter be enabled after the hardware reset?

In this case, the mouse was only the first visible symptom. The real failure was the local Intel Bluetooth adapter timing out and being unloaded by Windows.

## Takeaway

Connected peripherals are often blamed first because they are the part we touch.

But when the operating system loses the Bluetooth switch itself, the problem has moved below the peripheral. The adapter, driver, and power state become the useful places to look.

The final fix was not dramatic:

```text
Full shutdown -> wait >15 seconds -> boot -> enable Intel Bluetooth -> verify Started
```

Simple fixes are most satisfying when the evidence explains why they worked.

