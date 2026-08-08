# Technical Issue & Fix: OBS Studio – Pause Recording Button Missing

### 🔴 ISSUE

**Application:** OBS Studio 32.2.1
**Operating System:** Windows
**Function:** Screen Recording

The **Pause Recording** button was not visible in OBS Studio while recording.

The OBS Control panel displayed:

> **Start Recording**

but after starting the recording, the expected **Pause** option was not available.

### 🔍 ROOT CAUSE

The OBS recording configuration was using a recording mode/encoder configuration that **does not support the Pause Recording function**, most commonly:

> **Recording Quality = Same as stream**

When OBS uses **Same as stream**, the recording shares the streaming encoder configuration, and OBS does not provide the pause functionality for that recording mode.

### 🛠️ FIX

Navigate to:

**OBS → Settings → Output**

If using **Simple Output Mode**:

```text
Output Mode
    ↓
Simple
    ↓
Recording
    ↓
Recording Quality
```

Change:

```text
❌ Same as stream
```

to:

```text
✅ High Quality, Medium File Size
```

Then:

**Apply → OK**

Start a new recording.

### ✅ RESULT

After changing the recording configuration, the **Pause Recording** option becomes available while recording:

```text
OBS Controls

┌───────────────────────────────┐
│  Pause ⏸   │   Stop Recording │
└───────────────────────────────┘
```

You can now:

**Start Recording → Pause → Resume → Stop Recording**

without having to create separate recording files.

<img width="1605" height="1011" alt="OBS Recording Fix" src="https://github.com/user-attachments/assets/ba63d771-4e4a-4fde-85c9-8460a293aa47" />


### 📌 Technical Summary

| Component               | Details                                       |
| ----------------------- | --------------------------------------------- |
| Application             | OBS Studio 32.2.1                             |
| Issue                   | Pause button unavailable                      |
| Impact                  | Unable to temporarily pause a recording       |
| Root Cause              | Recording configuration did not support pause |
| Configuration           | `Same as stream`                              |
| Resolution              | Use a dedicated recording quality/encoder     |
| Recommended Simple Mode | `High Quality, Medium File Size`              |
| Reinstallation Required | ❌ No                                          |
| Plugin Required         | ❌ No                                          |
| Result                  | ✅ Pause/Resume available                      |

**Reference:** OBS documentation explains that recording with **“Same as stream” does not support pausing**. [OBS Recording Encoder Presets Guide](https://obsproject.com/kb/recording-encoder-presets-guide?utm_source=chatgpt.com)
