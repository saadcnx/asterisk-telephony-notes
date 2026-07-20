# Voicemail: Differentiating Busy & Unavailable Greetings

## The Problem

Previously, all unanswered calls were sent to voicemail using one greeting.

However, a call may fail for different reasons:

- **Busy:** The person is already on another call.
- **Unavailable:** The phone may be off, unregistered, unreachable, or not answered before the timeout.

We want Asterisk to play a different voicemail greeting for each case.

This requires three things:

1. Record both greetings.
2. Check the `${DIALSTATUS}` variable.
3. Call `VoiceMail()` with the correct option.

---

# Step 1: Recording the Greetings

Dial the voicemail access extension:

```text
*100
```

Enter the mailbox PIN and press:

```text
#
```

From the voicemail menu, record:

- The unavailable greeting
- The busy greeting

Example messages:

```text
Unavailable: I am not available right now. Please leave a message.
```

```text
Busy: I am currently on another call. Please leave a message or try again soon.
```

---

# Where the Greeting Files Are Stored

For mailbox `100` in the `default` context:

```text
/var/spool/asterisk/voicemail/default/100/
```

Common greeting files include:

```text
busy.gsm
unavail.gsm
```

Depending on the configured formats, they may also exist as:

```text
busy.wav
unavail.wav
```

For a corporate mailbox, professionally recorded greetings can be copied directly into this folder using the expected filenames.

---

# Step 2: Understanding `${DIALSTATUS}`

The `Dial()` application stores the result of the call attempt in:

```asterisk
${DIALSTATUS}
```

View the installed documentation:

```asterisk
core show application Dial
```

## Common Values

| Value | Meaning |
|---|---|
| `BUSY` | The destination reported that it was busy |
| `NOANSWER` | The timeout expired without an answer |
| `CHANUNAVAIL` | The channel or phone was unavailable |
| `CONGESTION` | The call failed because of congestion |
| `CANCEL` | The caller cancelled before answer |

Our logic is:

```text
If DIALSTATUS is BUSY
    Play the busy greeting
Otherwise
    Play the unavailable greeting
```

---

# Step 3: `VoiceMail()` Options

| Option | Purpose |
|---|---|
| `b` | Play the busy greeting |
| `u` | Play the unavailable greeting |

Examples:

```asterisk
VoiceMail(100@default,b)
```

```asterisk
VoiceMail(100@default,u)
```

---

# Step 4: Implementing the Logic

```ini
[phones]
exten => 100,1,Dial(SIP/matthias,5)
exten => 100,n,GotoIf($["${DIALSTATUS}" = "BUSY"]?100-busy)
exten => 100,n,VoiceMail(${EXTEN}@default,u)
exten => 100,n,Hangup()

exten => 100-busy,1,VoiceMail(${EXTEN:0:3}@default,b)
exten => 100-busy,n,Hangup()
```

---

# Understanding the Condition

```ini
GotoIf($["${DIALSTATUS}" = "BUSY"]?100-busy)
```

The test is evaluated inside:

```asterisk
$[ ... ]
```

If `${DIALSTATUS}` equals `BUSY`, Asterisk jumps to:

```text
100-busy
```

If the condition is false, execution continues to the next priority.

---

# Unavailable Greeting

```ini
exten => 100,n,VoiceMail(${EXTEN}@default,u)
```

This line is reached when the call was not busy.

Examples include:

```text
NOANSWER
CHANUNAVAIL
CONGESTION
CANCEL
```

At this point:

```text
${EXTEN} = 100
```

so Asterisk sends the caller to:

```text
100@default
```

and plays the unavailable greeting.

---

# Busy Greeting

```ini
exten => 100-busy,1,VoiceMail(${EXTEN:0:3}@default,b)
```

After the jump:

```text
${EXTEN} = 100-busy
```

The expression:

```asterisk
${EXTEN:0:3}
```

extracts the first three characters:

```text
100
```

Asterisk then executes:

```asterisk
VoiceMail(100@default,b)
```

and plays the busy greeting.

---

# Complete Call Flow

## No Answer

```text
Caller dials 100
      ↓
Dial(SIP/matthias,5)
      ↓
Timeout expires
      ↓
DIALSTATUS = NOANSWER
      ↓
Busy condition is false
      ↓
VoiceMail(100@default,u)
      ↓
Unavailable greeting plays
```

## Busy

```text
Caller dials 100
      ↓
Dial(SIP/matthias,5)
      ↓
Destination reports busy
      ↓
DIALSTATUS = BUSY
      ↓
Goto 100-busy
      ↓
Extract mailbox number 100
      ↓
VoiceMail(100@default,b)
      ↓
Busy greeting plays
```

---

# Important Call-Waiting Note

If call waiting is enabled, a second call may ring instead of returning a busy response.

To test the busy greeting, disable call waiting in the softphone settings.

---

# Testing the Unavailable Greeting

1. Reload the dialplan:

```asterisk
dialplan reload
```

2. Call extension `100`.
3. Do not answer.
4. Wait for the five-second timeout.

Expected result:

```text
DIALSTATUS = NOANSWER
```

Asterisk plays:

```asterisk
VoiceMail(100@default,u)
```

---

# Testing the Busy Greeting

Create a temporary test extension:

```ini
exten => *500,1,Answer()
exten => *500,n,Wait(60)
exten => *500,n,Hangup()
```

From the target phone, dial:

```text
*500
```

While that call is active, call extension `100` from another phone.

If call waiting is disabled, the target phone should report busy.

Expected result:

```text
DIALSTATUS = BUSY
```

Asterisk jumps to:

```text
100-busy
```

and plays:

```asterisk
VoiceMail(100@default,b)
```

---

# Why Differentiate the Greetings?

A busy greeting tells the caller that the user is already on another call and may be available soon.

An unavailable greeting is more general because the system does not know whether the user is away, offline, unreachable, or simply not answering.

Using separate greetings provides a more professional caller experience.
