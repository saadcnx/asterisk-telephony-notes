# Time-Based Conditions with `GotoIfTime`

## The Scenario

We already have three main contexts:

```text
incoming
phones
outgoing
```

Now, when a call arrives from outside, we want Asterisk to check whether the call is received during business hours.

The required behavior is:

- During business hours: Ring the internal phone.
- Outside business hours: Play an announcement and hang up.

This is done using time-based conditional routing.

---

# The `GotoIfTime()` Application

`GotoIfTime()` is a conditional version of `Goto()`.

It jumps to another context, extension, and priority only when the specified time condition matches.

To view the built-in help, run:

```asterisk
core show application GotoIfTime
```

The Asterisk CLI displays the syntax supported by the installed Asterisk version.

A commonly used external reference is:

```text
voip-info.org
```

Search for:

```text
GotoIfTime
```

It provides examples of valid time ranges, weekday names, month days, and month names.

If a parameter should match every possible value, use:

```text
*
```

---

# Syntax

```asterisk
GotoIfTime(times,weekdays,monthdays,months?context,extension,priority)
```

| Parameter | Description | Example |
|---|---|---|
| `times` | Time range | `08:00-17:00` |
| `weekdays` | Days of the week | `mon-fri` |
| `monthdays` | Days of the month | `*` |
| `months` | Months of the year | `*` |
| `?` | Separates the condition from the destination | `?` |
| Destination | Context, extension, and priority to jump to | `phones,100,1` |

Weekday names use shortened forms such as:

```text
sun
mon
tue
wed
thu
fri
sat
```

Month names use shortened forms such as:

```text
jan
feb
mar
apr
may
jun
jul
aug
sep
oct
nov
dec
```

---

# What Happens When the Condition Is False?

If no false destination is provided, Asterisk does not jump anywhere.

Instead, it continues to the next dialplan priority.

This allows the next line to act like an `else` condition.

---

# Implementing Business Hours

Open:

```text
/etc/asterisk/extensions.conf
```

Add or update the incoming context:

```ini
[incoming]
exten => 991123123,1,GotoIfTime(08:00-17:00,mon-fri,*,*?phones,100,1)
exten => 991123123,2,Playback(tt-monkeys)
exten => 991123123,3,Hangup()
```

---

# How the Dialplan Executes

## Priority 1: Check the Time

```ini
exten => 991123123,1,GotoIfTime(08:00-17:00,mon-fri,*,*?phones,100,1)
```

The condition means:

```text
Time: Monday to Friday
Hours: 08:00 to 17:00
Month day: Any
Month: Any
```

### If the Condition Is True

Asterisk jumps to:

```text
Context: phones
Extension: 100
Priority: 1
```

Equivalent destination:

```asterisk
Goto(phones,100,1)
```

The normal internal phone-routing logic then runs.

---

### If the Condition Is False

Asterisk does not jump.

It continues to the next priority:

```ini
exten => 991123123,2,Playback(tt-monkeys)
```

---

# Priority 2: Outside-Hours Announcement

```ini
exten => 991123123,2,Playback(tt-monkeys)
```

This line is reached only when the business-hours condition does not match.

Asterisk plays:

```text
tt-monkeys
```

In a real setup, this would normally be replaced with a message such as:

```text
Our office is currently closed.
```

---

# Priority 3: End the Call

```ini
exten => 991123123,3,Hangup()
```

After the announcement finishes, Asterisk hangs up the call.

---

# Complete Call Flow

## During Business Hours

```text
Incoming caller dials 991123123
        ↓
Call enters [incoming]
        ↓
GotoIfTime condition matches
        ↓
Goto phones,100,1
        ↓
Internal phone rings
```

## Outside Business Hours

```text
Incoming caller dials 991123123
        ↓
Call enters [incoming]
        ↓
GotoIfTime condition does not match
        ↓
Continue to priority 2
        ↓
Playback announcement
        ↓
Hangup
```

---

# Testing During Business Hours

Set the time range so that it matches the current server time.

Example:

```ini
GotoIfTime(08:00-17:00,mon-fri,*,*?phones,100,1)
```

Reload the dialplan:

```asterisk
dialplan reload
```

Call the incoming number:

```text
991123123
```

Expected result:

- The time condition matches.
- Asterisk jumps to `phones,100,1`.
- The internal phone rings.
- The announcement does not play.

---

# Testing Outside Business Hours

Temporarily change the time range so it does not match the current server time.

Example:

```ini
GotoIfTime(16:00-17:00,mon-fri,*,*?phones,100,1)
```

Reload the dialplan:

```asterisk
dialplan reload
```

Call the incoming number again.

Expected result:

- The condition does not match.
- Asterisk continues to priority 2.
- The `tt-monkeys` prompt plays.
- Asterisk hangs up.
- The internal phone does not ring.

---

# Important Timezone Note

`GotoIfTime()` evaluates time according to the Asterisk server's clock and timezone.

Check the server time:

```bash
date
```

Check the timezone:

```bash
timedatectl
```

If the server timezone is wrong, business-hours routing may also happen at the wrong time.

---

# Why `GotoIfTime()` Is Useful

`GotoIfTime()` is commonly used for:

- Business-hours routing
- After-hours announcements
- Holiday routing
- Weekend call handling
- Lunch-break routing
- Time-based call forwarding
- Different day and night IVRs

It is one of the most commonly used conditional applications in real-world Asterisk dialplans.
