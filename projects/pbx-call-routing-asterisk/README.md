# ☎️ Small Office PBX System — Asterisk Call Flow Project

A fully functional **PBX (Private Branch Exchange) system** built with Asterisk for a small company with three departments — **Sales, Support, and Admin**. The system implements time-based call routing, department-based call distribution, voicemail integration, and pattern-matched internal dialing.

---

## 📋 Overview

This project simulates a real-world small office telephony setup. When a call comes in, the system checks the current time and business hours to decide whether to present an interactive menu or route the caller straight to voicemail. During business hours, callers can select a department via DTMF input, and the call is distributed to the corresponding extension with proper fallback handling if no one answers.

**Core capabilities:**
- SIP peer registration and configuration for internal extensions
- Multi-context dialplan design
- Extension pattern matching (`_1XX`)
- Time-based conditional routing (`GotoIfTime`)
- Subroutines with argument passing (`GoSub`)
- Voicemail configuration and call-status-aware routing
- Custom audio prompts for a professional IVR experience

---

## 🏗️ System Architecture

```
Incoming Call
      │
      ▼
 Business Hours? ──No──► After-Hours Message ──► Voicemail
      │
     Yes
      │
      ▼
  Welcome Greeting / IVR Menu
      │
      ├── Press 1 ──► Sales Greeting ──► Ring 101 ──► Voicemail (if no answer)
      ├── Press 2 ──► Support Greeting ──► Ring 102 ──► Voicemail (if no answer)
      ├── Press 3 ──► Admin Greeting ──► Ring 103 ──► Voicemail (if no answer)
      └── Invalid Input ──► Invalid Option Prompt ──► Replay Menu
```

---

## 📁 Project Structure

```
project_repo/
├── sip.conf              # SIP peer/extension definitions
├── voicemail.conf         # Voicemail boxes per department/extension
├── extensions.conf        # Dialplan: contexts, IVR logic, routing
└── audio/
    ├── welcome.gsm
    ├── menu-option.gsm
    ├── sales.gsm
    ├── support.gsm
    ├── administration.gsm
    ├── after-hours.gsm
    └── invalid-option.gsm
```

---

## ⚙️ Configuration Files

### `sip.conf`
Defines SIP peers for each internal extension (101 – Sales, 102 – Support, 103 – Admin), including authentication, codecs, and NAT settings for local network testing.

### `extensions.conf`
The heart of the system. Contains:
- **`[incoming]`** context — entry point for external calls, checks business hours with `GotoIfTime`
- **`[ivr-menu]`** context — plays the welcome prompt and waits for DTMF input via `Background`/`WaitExten`
- **`[departments]`** context — uses `GoSub` with passed arguments to route calls to the correct extension and greeting
- **`[internal]`** context — pattern-matched internal dialing using `_1XX` for extensions 101–103
- Call status handling via `${DIALSTATUS}` to fall back to voicemail on `NOANSWER`, `BUSY`, or `CHANUNAVAIL`

### `voicemail.conf`
Defines voicemail mailboxes per extension, mapped to each department, with custom greetings and PIN-protected access.

---

## 🔀 Call Flow Logic

1. **Time-based routing** — `GotoIfTime` checks the current day/time against defined business hours before deciding the call path.
2. **IVR menu** — Caller hears the welcome prompt and selects a department by dialing 1, 2, or 3.
3. **Department routing** — A shared subroutine (`GoSub`) receives the target extension and department name as arguments, plays the relevant greeting, and dials the extension.
4. **Call distribution** — If the extension doesn't answer, the call falls through to that department's voicemail box based on `${DIALSTATUS}`.
5. **Pattern matching** — Internal extensions 101, 102, and 103 all match the `_1XX` pattern for simplified dialplan logic.
6. **Invalid input handling** — Any unrecognized DTMF input plays an "invalid option" prompt and re-presents the menu.

---

## ✅ Testing

### Time-Based Routing
| Scenario | Expected Result |
|---|---|
| Call during business hours | Caller hears the IVR menu |
| Call after business hours | Caller is routed directly to voicemail |

### Department Routing
| Input | Result |
|---|---|
| Press `1` | Sales greeting plays → rings extension **101** |
| Press `2` | Support greeting plays → rings extension **102** |
| Press `3` | Admin greeting plays → rings extension **103** |

### Pattern Matching
```bash
# Dialing 101, 102, or 103 internally all match the _1XX pattern
```

### Additional Behavior Verified
- Caller ID is captured and displayed correctly
- Time-based decisions correctly branch call flow
- Department routing correctly passes and uses arguments via `GoSub`
- `${DIALSTATUS}` correctly triggers voicemail fallback on no-answer/busy

---

## 🎓 Learning Outcomes

- ✅ Real-world SIP peer configuration
- ✅ Complex dialplan design across multiple contexts
- ✅ Extension pattern matching (`_1XX`)
- ✅ Variable manipulation and string operations in the dialplan
- ✅ Time-based routing with `GotoIfTime`
- ✅ Subroutines with `GoSub` and argument passing
- ✅ Voicemail integration across multiple scenarios
- ✅ Call flow control using `${DIALSTATUS}`
- ✅ Secure context mapping between inbound, IVR, and internal dialplans

---

## 🛠️ Requirements

- Asterisk (18.x or later recommended)
- A softphone client for testing (e.g., Zoiper, Linphone, MicroSIP)
- Basic familiarity with the Asterisk CLI (`asterisk -rvvv`)

---

## 🚀 Getting Started

1. Clone this repository.
2. Copy `sip.conf`, `voicemail.conf`, and `extensions.conf` into your Asterisk configuration directory (typically `/etc/asterisk/`).
3. Copy the contents of `audio/` into your Asterisk sounds directory.
4. Reload the dialplan and SIP configuration:
   ```bash
   asterisk -rx "dialplan reload"
   asterisk -rx "sip reload"
   ```
5. Register a softphone to one of the configured extensions (101, 102, or 103) and place a test call to the inbound context.

---

## 📸 Screenshots

> _Add your test screenshots here to show the working call flow._

- Business hours call → IVR menu
- After-hours call → straight to voicemail
- Department routing → Sales greeting → ring extension 101

---

## 📄 License

This project is open for educational and portfolio use. Feel free to fork and adapt it for your own learning.