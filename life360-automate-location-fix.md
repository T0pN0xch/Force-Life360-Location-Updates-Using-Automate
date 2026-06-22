# Force Life360 Location Updates Using Automate (Android)

## Problem

Life360 occasionally fails to update a device's location in real time. This happens due to Android's background process restrictions, battery optimization killing the app's location services, or the app simply not polling frequently enough when idle.

## Solution

Use the **Automate** app (by LlamaLab) to periodically launch Life360 in the foreground, forcing it to refresh its location.

## Requirements

- Android device
- [Automate by LlamaLab](https://play.google.com/store/apps/details?id=com.llamalab.automate) (free tier is sufficient — this flow uses ~3 blocks)
- Life360 installed (`com.life360.android.safetymapd`)

---

## Setting Up the Flow

### Step 1: Create a new flow

Open Automate → tap **+** → name it (e.g. `life360 helper`).

### Step 2: Add blocks

Add the following three blocks in order:

#### Block 1 — Flow beginning
- Drag in: `Flow beginning`
- No configuration needed.

#### Block 2 — Delay
- Drag in: `Delay`
- **Proceed:** `Exact`
- **Duration:** `0h 30m 0s` (set to 30s for testing)
- **Wake up:** `Awake device` ✓

#### Block 3 — App start
- Drag in: `App start`
- **Package:** `com.life360.android.safetymapd`
- **Activity class:** *(leave empty)*
- **Action:** `Main` ← **critical, must be `Main` not `Run`**
- Leave all other fields empty.

### Step 3: Connect the blocks

```
Flow beginning (GO) → Delay (IN)
Delay (OK) → App start (IN)
App start (OK) → Delay (IN)   ← loop back
```

The loop from `App start OK` back to `Delay IN` is what makes this repeat indefinitely.

### Step 4: Configure permissions

On the flow's run screen, ensure the following privileges are enabled:

- ✅ Appear on top of other apps
- ✅ Show notifications
- ✅ Ignore app hibernation
- ✅ Ignore battery optimizations

Also go to **Android Settings → Apps → Automate → Battery → Unrestricted** to prevent the OS from killing it.

---

## How It Works

```
[Start] → Wait 30 min → Launch Life360 → Wait 30 min → Launch Life360 → ...
```

Every 30 minutes, Automate brings Life360 to the foreground. This triggers the app's active location polling, which updates the device's position on the map for other circle members.

---

## Known Limitations

| Limitation | Detail |
|---|---|
| Screen-off blocking | Android 10+ restricts background apps from launching activities when the screen is locked. Life360 may not open if the device is locked. |
| Interval accuracy | `Exact` mode improves precision but Android may still defer execution slightly under heavy load. |
| Battery impact | Launching Life360 every 30 minutes increases battery usage. Adjust interval as needed. |

---

## Troubleshooting

### `android.content.ActivityNotFoundException`
**Cause:** Action field is set to `Run` instead of `Main`.  
**Fix:** Open the App start block → change Action to `Main` → save.

### Flow runs once and stops
**Cause:** Missing loop — `App start OK` is not connected back to `Delay IN`.  
**Fix:** Long-press the OK connector on App start → drag arrow to the IN connector on Delay.

### Flow stops after some time
**Cause:** Battery optimization is killing Automate.  
**Fix:** Set Automate to **Unrestricted** in battery settings and enable **Ignore battery optimizations** in flow privileges.

---

## Tested Environment

- Device: Android (Samsung, MIUI-based devices)
- Automate version: Free tier
- Life360 package: `com.life360.android.safetymapd`
- Android version: 10+
