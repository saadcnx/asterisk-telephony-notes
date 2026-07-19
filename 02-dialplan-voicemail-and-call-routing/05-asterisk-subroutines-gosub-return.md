# Subroutines: Cleaner Time Controls with `GoSub`

## The Problem with Copy-Pasting

Previously, we used `GotoIfTime()` to check business hours.

In a real environment, business hours can be more complex:

```text
Monday to Saturday: 09:00–17:00
Sunday: 10:00–16:00
Additional holiday rules
```

A complete time-checking section can easily contain 10–15 dialplan lines. If the same logic is copied for 100 users or departments, every future change must be repeated in all copies, which is slow and error-prone.

---

# The Solution: Subroutines

When the same dialplan logic is needed in multiple places, write it once as a subroutine.

The main dialplan:

1. Enters the subroutine.
2. Executes the shared logic.
3. Returns to the line immediately after the original subroutine call.

## Main Applications

| Application | Purpose |
|---|---|
| `GoSub()` | Jumps to another context, extension, and priority while remembering where execution came from |
| `Return()` | Returns to the priority immediately after the original `GoSub()` call |

Unlike a normal `Goto()`, `GoSub()` allows execution to return.

---

# Avoid the Legacy `Macro()` Application

Older Asterisk dialplans may use:

```asterisk
Macro()
```

`Macro()` is a legacy approach with nesting and design limitations.

Modern dialplans should use:

```asterisk
GoSub()
```

instead.

---

# Building the Subroutine

Create a dedicated context:

```ini
[timecheck]
exten => s,1,GotoIfTime(09:00-17:00,mon-fri,*,*?ok)
exten => s,2,GotoIfTime(10:00-16:00,sun,*,*?ok)
exten => s,3,Playback(tt-monkeys)
exten => s,4,Hangup()

exten => ok,1,Return()
```

---

# Understanding the `s` Extension

```ini
exten => s,1
```

The `s` extension means:

```text
start
```

It is commonly used as the entry point for a subroutine when no specific dialed number is required.

The subroutine starts at:

```text
Context: timecheck
Extension: s
Priority: 1
```

---

# Priority 1: Weekday Check

```ini
exten => s,1,GotoIfTime(09:00-17:00,mon-fri,*,*?ok)
```

This checks:

```text
Time: 09:00 to 17:00
Days: Monday to Friday
Month days: Any
Months: Any
```

If the condition matches, Asterisk jumps to:

```text
ok,1
```

If it does not match, execution continues to priority 2.

---

# Priority 2: Sunday Check

```ini
exten => s,2,GotoIfTime(10:00-16:00,sun,*,*?ok)
```

This checks:

```text
Time: 10:00 to 16:00
Day: Sunday
Month days: Any
Months: Any
```

If the condition matches, Asterisk jumps to `ok,1`.

If it does not match, execution continues to priority 3.

> The scenario mentions Monday to Saturday business hours, but the shown rule covers Monday to Friday. Add a Saturday rule if Saturday should also be open.

---

# Priorities 3–4: Closed Handling

```ini
exten => s,3,Playback(tt-monkeys)
exten => s,4,Hangup()
```

These lines run only if none of the time conditions matched.

The call:

1. Plays the closed-hours announcement.
2. Hangs up.

Because the call ends, the subroutine does not return.

---

# The `ok` Extension

```ini
exten => ok,1,Return()
```

If any business-hours condition matches, Asterisk jumps to `ok,1`.

`Return()` sends execution back to the line immediately after the original `GoSub()` call.

Using a named extension such as `ok` makes the dialplan easier to read.

---

# Using the Subroutine

```ini
[incoming]
exten => 991123123,1,GoSub(timecheck,s,1)
exten => 991123123,2,Goto(phones,200,1)
```

The call:

```asterisk
GoSub(timecheck,s,1)
```

means:

```text
Context: timecheck
Extension: s
Priority: 1
```

Asterisk remembers the original location before entering the subroutine.

---

# Execution During Business Hours

```text
Incoming call reaches 991123123
        ↓
GoSub(timecheck,s,1)
        ↓
A time condition matches
        ↓
Goto ok,1
        ↓
Return()
        ↓
Back to [incoming], priority 2
        ↓
Goto(phones,200,1)
        ↓
Internal phone rings
```

---

# Execution Outside Business Hours

```text
Incoming call reaches 991123123
        ↓
GoSub(timecheck,s,1)
        ↓
All time conditions fail
        ↓
Playback(tt-monkeys)
        ↓
Hangup()
        ↓
Call ends without returning
```

The internal phone does not ring.

---

# Full Working Example

```ini
[timecheck]
exten => s,1,GotoIfTime(09:00-17:00,mon-fri,*,*?ok)
exten => s,2,GotoIfTime(10:00-16:00,sun,*,*?ok)
exten => s,3,Playback(tt-monkeys)
exten => s,4,Hangup()

exten => ok,1,Return()

[incoming]
exten => 991123123,1,GoSub(timecheck,s,1)
exten => 991123123,2,Goto(phones,200,1)
```

---

# Testing During Business Hours

Set a time condition that matches the current server time.

Reload:

```asterisk
dialplan reload
```

Call:

```text
991123123
```

Expected flow:

```text
GoSub
→ Time condition matches
→ ok
→ Return
→ Goto(phones,200,1)
→ Phone rings
```

---

# Testing Outside Business Hours

Temporarily set conditions that do not match the current server time.

Reload:

```asterisk
dialplan reload
```

Call again.

Expected flow:

```text
GoSub
→ Conditions fail
→ Playback(tt-monkeys)
→ Hangup
```

---

# Why This Is Powerful

## Maintainability

If business hours change, edit only:

```ini
[timecheck]
```

## Reusability

Other incoming numbers, departments, or call flows can use:

```asterisk
GoSub(timecheck,s,1)
```

## Readability

The main incoming context stays short and easy to understand.
