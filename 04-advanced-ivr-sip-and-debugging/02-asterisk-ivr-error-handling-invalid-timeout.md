# IVR Advanced: Error Handling for Invalid Input & Timeout

## The Problem with the Basic IVR

Previously, we built a simple IVR menu.

It worked, but it had two critical problems:

- If the caller pressed an invalid option, such as `3`, the call failed and hung up.
- If the caller did not press anything before the `WaitExten()` timeout expired, the call also failed.

This behavior is not acceptable for a production system.

The IVR needs proper error handling.

---

# Asterisk Special Extensions

When an invalid extension or timeout occurs, Asterisk looks for special extensions inside the current context.

| Extension | Meaning |
|---|---|
| `i` | Invalid input |
| `t` | Timeout |
| `e` | General error fallback |

If these extensions are not defined, Asterisk may hang up the call.

Defining them allows the dialplan to handle errors gracefully.

---

# Handling Invalid Input with `i`

The `i` extension handles a key press that does not match any defined extension.

For example, if the IVR only defines:

```text
1
2
```

and the caller presses:

```text
3
```

Asterisk routes the call to:

```text
i
```

## Invalid Input Handling

```ini
[ivr1]
exten => i,1,NoOp(Invalid Key Pressed)
exten => i,n,Playback(invalid-key)
exten => i,n,Goto(start,1)
```

This logic:

1. Logs the invalid key press.
2. Plays an error message.
3. Returns to the beginning of the IVR.
4. Gives the caller another chance.

The `Playback()` line is optional if no custom error prompt is available.

---

# Handling Timeout with `t`

The `t` extension handles the case where the caller does not press anything before `WaitExten()` expires.

Example:

```ini
[ivr1]
exten => t,1,NoOp(Timeout)
exten => t,n,Queue(sales)
exten => t,n,Hangup()
```

This logic:

1. Logs the timeout.
2. Sends the caller to a queue or operator.
3. Hangs up after the queue application finishes.

---

# Why Handle Invalid Input and Timeout Differently?

## Invalid Input

The caller is actively interacting with the IVR but pressed the wrong key.

Recommended action:

```text
Play an error message and replay the menu.
```

## Timeout

The caller is not interacting with the menu.

They may:

- Need human assistance
- Be unable to send DTMF tones
- Be unsure which option to choose

Recommended action:

```text
Send the caller to a queue or operator.
```

---

# Full Enhanced IVR Dialplan

```ini
[ivr1]
exten => start,1,NoOp(IVR 1)
exten => start,n,Answer()
exten => start,n,Playback(ivr_q1)
exten => start,n,WaitExten(5)

; Valid Options
exten => 1,1,NoOp(Pressed 1)
exten => 1,n,Queue(test)
exten => 1,n,Hangup()

exten => 2,1,NoOp(Pressed 2)
exten => 2,n,Queue(blah)
exten => 2,n,Hangup()

; Invalid Input
exten => i,1,NoOp(Invalid Key)
exten => i,n,Playback(invalid-key)
exten => i,n,Goto(start,1)

; Timeout
exten => t,1,NoOp(Timeout)
exten => t,n,Queue(test)
exten => t,n,Hangup()
```

---

# Valid Option 1

```ini
exten => 1,1,NoOp(Pressed 1)
exten => 1,n,Queue(test)
exten => 1,n,Hangup()
```

When the caller presses:

```text
1
```

Asterisk sends the call to:

```text
test
```

queue.

---

# Valid Option 2

```ini
exten => 2,1,NoOp(Pressed 2)
exten => 2,n,Queue(blah)
exten => 2,n,Hangup()
```

When the caller presses:

```text
2
```

Asterisk sends the call to:

```text
blah
```

queue.

---

# Invalid Key Flow

```text
Caller presses an undefined key
        ↓
Asterisk enters extension i
        ↓
Log invalid input
        ↓
Play invalid-key prompt
        ↓
Goto(start,1)
        ↓
Replay the IVR menu
```

---

# Timeout Flow

```text
Caller does not press anything
        ↓
WaitExten(5) expires
        ↓
Asterisk enters extension t
        ↓
Log timeout
        ↓
Send caller to fallback queue
```

---

# Testing the Enhanced IVR

Reload the dialplan:

```asterisk
dialplan reload
```

Dial the IVR extension:

```text
800
```

---

# Test Invalid Input

Press:

```text
3
```

Expected result:

1. The CLI shows `Invalid Key`.
2. Asterisk plays the `invalid-key` prompt.
3. The IVR starts again.

---

# Test Timeout

Do not press any key.

Wait longer than:

```text
5 seconds
```

Expected result:

1. The CLI shows `Timeout`.
2. Asterisk sends the caller to the fallback queue.

---

# DTMF Problems

Some callers may be unable to send DTMF tones correctly because of phone or provider issues.

In that situation, they cannot select any IVR option.

For this reason, the timeout path should lead to a real person, queue, or operator.

Otherwise, the caller may have no way to reach anyone.

---

# Summary

| Scenario | Special Extension | Recommended Action |
|---|---|---|
| Invalid key pressed | `i` | Play an error message and replay the menu |
| Nothing pressed | `t` | Transfer to a queue or operator |
| Other error | `e` | Use as a general fallback handler |

With invalid-input and timeout handling, the IVR can recover from common caller problems instead of ending the call unexpectedly.
