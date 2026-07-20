# Voicemail Dialplan Integration

## The Goal

Previously, we configured voicemail mailboxes in:

```text
/etc/asterisk/voicemail.conf
```

Now we need to use those mailboxes inside the dialplan so:

- Callers can leave voicemail messages.
- Mailbox owners can listen to and manage their messages.

---

# The Two Main Voicemail Applications

| Application | Purpose | Used By |
|---|---|---|
| `VoiceMail()` | Allows a caller to leave a voicemail message | Internal or external callers |
| `VoiceMailMain()` | Allows a user to manage their mailbox | Mailbox owner |

Find them in the Asterisk CLI:

```asterisk
core show applications like voice
```

Display detailed help:

```asterisk
core show application VoiceMail
```

```asterisk
core show application VoiceMailMain
```

The exact options supported depend on the installed Asterisk version.

---

# Part 1: Leaving a Voicemail

## `VoiceMail()` Syntax

```asterisk
VoiceMail(mailbox[@context][,options])
```

If the mailbox is in the `default` context, the context may be omitted. Using the complete mailbox address is clearer:

```text
100@default
```

---

# Adding Voicemail After `Dial()`

```ini
[phones]
exten => 100,1,Dial(SIP/matthias,5)
exten => 100,2,VoiceMail(100@default)
exten => 100,3,Hangup()

exten => 200,1,Dial(SIP/james,10)
exten => 200,2,VoiceMail(200@default)
exten => 200,3,Hangup()
```

## How the Timeout Works

```ini
Dial(SIP/matthias,5)
```

rings Matthias for five seconds.

```ini
Dial(SIP/james,10)
```

rings James for ten seconds.

If the call is answered, execution remains inside `Dial()` until the call ends, so voicemail is not reached.

If the timeout expires or the channel cannot be created because the phone is offline, unregistered, or unreachable, `Dial()` finishes and execution continues to `VoiceMail()`.

---

# Using `${EXTEN}` for Efficiency

Instead of hardcoding the mailbox number, use:

```asterisk
${EXTEN}
```

Example:

```ini
exten => 100,1,Dial(SIP/matthias,5)
exten => 100,2,VoiceMail(${EXTEN}@default)
exten => 100,3,Hangup()
```

When someone dials `100`:

```text
${EXTEN} = 100
```

Asterisk uses:

```asterisk
VoiceMail(100@default)
```

The same line can be reused for extension `200`:

```ini
exten => 200,1,Dial(SIP/james,10)
exten => 200,2,VoiceMail(${EXTEN}@default)
exten => 200,3,Hangup()
```

When someone dials `200`, `${EXTEN}` automatically becomes `200`.

---

# Cleaner Syntax

```ini
[phones]
exten => 100,1,Dial(SIP/matthias,5)
same => n,VoiceMail(${EXTEN}@default)
same => n,Hangup()

exten => 200,1,Dial(SIP/james,10)
same => n,VoiceMail(${EXTEN}@default)
same => n,Hangup()
```

---

# What Happens When a Voicemail Is Left?

A typical voicemail flow is:

1. Asterisk plays the mailbox greeting.
2. The caller hears a message such as:

```text
The person at extension 100 is unavailable.
```

3. Asterisk plays a beep.
4. Recording begins.
5. The message is saved in the configured audio formats.

The caller can:

- Hang up — the recording is still saved after recording has started.
- Press `#` — the recording ends and Asterisk may play a confirmation message before hanging up.

---

# Part 2: Managing Voicemails

## `VoiceMailMain()` Syntax

```asterisk
VoiceMailMain(mailbox[@context][,options])
```

This application allows mailbox owners to:

- Listen to messages
- Delete messages
- Save messages
- Move messages
- Record greetings
- Change mailbox settings

---

# Creating a Mailbox Access Extension

```ini
exten => *100,1,VoiceMailMain(100@default)
exten => *100,2,Hangup()
```

When the user dials:

```text
*100
```

Asterisk opens:

```text
100@default
```

The user is prompted for the mailbox PIN, such as:

```text
1234
```

Cleaner syntax:

```ini
exten => *100,1,VoiceMailMain(100@default)
same => n,Hangup()
```

---

# The Voicemail Menu

Common actions include:

| Key | Action |
|---|---|
| `1` | Listen to new messages |
| `7` | Delete the current message |

Other options may allow users to:

- Save messages
- Move messages
- Replay messages
- Record greetings
- Change mailbox settings

The exact menu can vary by Asterisk version and installed sound files.

---

# Recording Greetings

Mailbox owners can record:

## Unavailable Greeting

Played when the phone does not answer or is unavailable.

## Busy Greeting

Played when the phone is busy.

## Name Recording

The user records only their name.

Asterisk can build a complete greeting such as:

```text
This is the mailbox of [recorded name]. Please leave a message after the beep.
```

---

# PIN Entry Tip

After entering the mailbox PIN, press:

```text
#
```

This submits the PIN immediately instead of waiting for the input timeout.

---

# Testing the Flow

## Reload the Dialplan

```asterisk
dialplan reload
```

## Test Leaving a Message

1. Call extension `100`.
2. Do not answer Matthias's phone.
3. Wait for the five-second timeout.
4. Listen to the greeting and beep.
5. Record a message.
6. Press `#` or hang up.

The message should be stored in:

```text
100@default
```

## Test Managing Messages

Dial:

```text
*100
```

Enter:

```text
1234
```

Then press:

```text
#
```

Use the menu to listen to, save, or delete the message.

---

# Full Working Example

```ini
[phones]
exten => 100,1,Dial(SIP/matthias,5)
same => n,VoiceMail(${EXTEN}@default)
same => n,Hangup()

exten => 200,1,Dial(SIP/james,10)
same => n,VoiceMail(${EXTEN}@default)
same => n,Hangup()

exten => *100,1,VoiceMailMain(100@default)
same => n,Hangup()

exten => *200,1,VoiceMailMain(200@default)
same => n,Hangup()
```

---

# Next Step

This setup uses the unavailable greeting for every unsuccessful call.

A future improvement is to check:

```asterisk
${DIALSTATUS}
```

after `Dial()` so Asterisk can:

- Play the busy greeting when the phone is busy.
- Play the unavailable greeting when the call is not answered.
