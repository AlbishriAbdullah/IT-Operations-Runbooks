# Unexpected PC Power-On After Shutdown

## 1. Overview

This runbook explains how to troubleshoot a Windows computer that powers on or wakes unexpectedly after being shut down, placed in Sleep, or placed in Hibernate.

| Item | Details |
|---|---|
| Runbook Type | Manual |
| Category | Windows / Power / BIOS |
| Affected Systems | Windows desktops and laptops |
| Required Access | Local Administrator and BIOS/UEFI access |
| Status | Draft – Requires Field Testing |

## 2. Symptoms

The user may report one or more of the following:

- The computer powers on by itself after a normal shutdown.
- The computer turns on during the night or outside working hours.
- The computer starts after the network cable is connected.
- The computer starts after electricity is disconnected and restored.
- The computer wakes shortly after entering Sleep or Hibernate.
- No user pressed the power button.

## 3. Possible Causes

1. Wake-on-LAN is enabled.
2. A network adapter is allowed to wake the computer.
3. A scheduled task or wake timer is active.
4. RTC Alarm or Auto Power On is enabled in BIOS/UEFI.
5. Restore on AC Power Loss is configured to start the computer.
6. A USB device, keyboard, or mouse is allowed to wake the computer.
7. Windows Fast Startup is affecting shutdown behavior.
8. Intel AMT/vPro or another remote-management feature is configured.
9. The computer entered Sleep or Hibernate instead of shutting down completely.

## 4. Diagnostic Procedure

> Run Command Prompt as Administrator.

### Step 1: Check the Last Wake Source

```cmd
powercfg /lastwake
```

This reports the device or event that woke the computer from its last sleep transition.

> This command may not identify the cause if the computer started from a complete shutdown.

### Step 2: Check Active Wake Timers

```cmd
powercfg /waketimers
```

Review the output for scheduled tasks that are allowed to wake the computer.

### Step 3: Check Devices Allowed to Wake the Computer

```cmd
powercfg /devicequery wake_armed
```

Possible results may include:

- Network adapters
- Keyboards
- Mice
- USB devices

### Step 4: Perform a Full Shutdown

```cmd
shutdown /s /f /t 0
```

Monitor the computer to determine whether it powers on again after a complete shutdown.

### Step 5: Review Event Viewer

1. Open **Event Viewer**.
2. Go to **Windows Logs > System**.
3. Review events recorded around the startup time.
4. Look for these event sources:

```text
Power-Troubleshooter
Kernel-Power
Kernel-Boot
User32
```

## 5. Resolution

Apply only the solution related to the identified cause.

### Option A: Disable Network Adapter Wake Permission

1. Open **Device Manager**.
2. Expand **Network adapters**.
3. Right-click the active network adapter.
4. Select **Properties**.
5. Open the **Power Management** tab.
6. Clear:

```text
Allow this device to wake the computer
```

7. Open the **Advanced** tab.
8. Disable unused options such as:

```text
Wake on Magic Packet
Wake on Pattern Match
Shutdown Wake-On-Lan
```

> Do not disable Wake-on-LAN if it is required for remote support, patching, or device management.

### Option B: Disable Wake Permission Using PowerCFG

First, obtain the exact device name:

```cmd
powercfg /devicequery wake_armed
```

Then disable its wake permission:

```cmd
powercfg /devicedisablewake "Device Name"
```

Example:

```cmd
powercfg /devicedisablewake "HID-compliant mouse"
```

### Option C: Disable Wake Timers

1. Open **Control Panel**.
2. Go to **Power Options**.
3. Select **Change plan settings**.
4. Select **Change advanced power settings**.
5. Expand **Sleep**.
6. Expand **Allow wake timers**.
7. Set the option to **Disable**.

### Option D: Review BIOS/UEFI Settings

Restart the computer and enter BIOS/UEFI.

Disable unused options such as:

```text
Wake on LAN
Power On by PCI-E
RTC Wake / RTC Alarm
Auto Power On
Scheduled Power On
USB Wake Support
```

Configure the power recovery option as:

```text
Restore on AC Power Loss: Stay Off
```

> Setting names vary depending on the computer manufacturer.

### Option E: Disable Fast Startup for Testing

1. Open **Control Panel**.
2. Go to **Power Options**.
3. Select **Choose what the power buttons do**.
4. Select **Change settings that are currently unavailable**.
5. Clear **Turn on fast startup**.
6. Save the changes and test again.

## 6. Validation

1. Perform a complete shutdown:

```cmd
shutdown /s /f /t 0
```

2. Leave the computer connected to power and the network.
3. Monitor it during the period when the issue usually occurs.
4. Confirm that the computer remains powered off.
5. Check the remaining wake-enabled devices:

```cmd
powercfg /devicequery wake_armed
```

### Expected Result

The computer remains powered off until the power button is pressed or an approved remote-management function is used.

## 7. Rollback

If Wake-on-LAN is required again:

1. Re-enable Wake-on-LAN in BIOS/UEFI.
2. Re-enable **Wake on Magic Packet** in the network adapter settings.
3. Restore the device wake permission:

```cmd
powercfg /deviceenablewake "Device Name"
```

## 8. Escalation

Escalate the issue if:

- The computer continues powering on after all wake settings are disabled.
- BIOS/UEFI settings return automatically after being changed.
- Intel AMT/vPro or another management platform controls the device.
- Group Policy or endpoint management controls the power settings.
- A motherboard, power supply, or physical power-button fault is suspected.

Collect the following before escalation:

- Computer manufacturer and model
- Windows version
- BIOS/UEFI version
- Approximate startup time
- Relevant Event Viewer logs
- PowerCFG command output
- Screenshots of relevant BIOS/UEFI settings

## 9. Security Considerations

Wake-on-LAN is a legitimate administrative feature and is not a vulnerability by itself. However, unused remote-power and remote-management features should be disabled to reduce the device's attack surface.

Confirm that Wake-on-LAN, Intel AMT/vPro, and other remote-management features are not required by the organization before disabling them.

## 10. References

- [Microsoft Learn – PowerCFG command-line options](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/powercfg-command-line-options)

## 11. Document Control

| Field | Value |
|---|---|
| Created | 17 September 2026 |
| Last Updated | 17 September 2026 |
| Last Tested | Not yet field-tested |
| Owner | Abdullah Albishri |
| Status | Draft – Requires Field Testing |
