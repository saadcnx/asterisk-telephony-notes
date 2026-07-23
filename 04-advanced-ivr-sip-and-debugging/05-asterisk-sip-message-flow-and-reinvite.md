# SIP Message Flow & Re-Invite

## Why SIP Is Easy to Learn

SIP is designed to be human-readable.

You can read a SIP trace and understand what is happening, which makes debugging easier.

---

# The Basic SIP Call Flow

In most cases, there are two call legs:

```text
Phone A → Asterisk
Asterisk → Phone B
```

Asterisk sits in the middle and manages both legs.

---

# Basic Message Flow

```text
Phone A          Asterisk/PBX          Phone B
  |                  |                    |
  |---- INVITE ----->|                    |
  |<--- 100 Trying --|                    |
  |                  |---- INVITE ------->|
  |                  |<--- 180 Ringing ---|
  |<-- 180 Ringing --|                    |
  |                  |<---- 200 OK -------|
  |<---- 200 OK -----|                    |
  |------ ACK ------>|                    |
  |                  |------- ACK ------->|
  |<========= RTP MEDIA FLOWS ===========>|
  |                  |                    |
  |------ BYE ------>|                    |
  |                  |------- BYE ------->|
  |<---- 200 OK -----|                    |
```

---

# Key Messages

| Message | Meaning |
|---|---|
| `INVITE` | Requests the start of a call and usually includes SDP |
| `100 Trying` | The request was received and is being processed |
| `180 Ringing` | The destination phone is ringing |
| `200 OK` | The call was answered or the request succeeded |
| `ACK` | Acknowledges the `200 OK` response |
| `BYE` | Ends the established call |
| `RTP` | Carries the actual media and is not SIP signaling |

---

# Default Behavior: Media Through Asterisk

In a standard Asterisk setup:

```text
Signaling → Through Asterisk
Media     → Through Asterisk
```

Asterisk relays the audio:

```text
Phone A ← RTP → Asterisk ← RTP → Phone B
```

This is useful for:

- Call recording
- DTMF detection
- IVR features
- Other audio-related features

---

# Re-Invite and Direct Media

After the initial call setup, Asterisk may allow the phones to send RTP directly to each other.

This is called:

```text
Re-Invite
```

or:

```text
Direct Media
```

The signaling still passes through Asterisk, but the media bypasses it.

```text
Phone A                  Asterisk                  Phone B
  |                         |                         |
  |   Initial SIP call setup through Asterisk        |
  |                         |                         |
  |<============= Direct RTP Media =================>|
```

---

# Advantages of Direct Media

## Reduced Server Load

RTP carries most of the network traffic.

If the media bypasses Asterisk, the server handles less traffic.

## Efficient Local Calls

If both phones are in the same office, they can exchange audio directly while signaling still passes through Asterisk.

---

# Disadvantages of Direct Media

## No Call Recording

Asterisk cannot record media that does not pass through it.

## Limited DTMF Handling

Asterisk may not receive DTMF tones when media bypasses the server.

## Network Problems

Direct media requires both phones to reach each other.

If they are on different networks and cannot communicate directly, audio may fail even though signaling works.

---

# Asterisk Configuration Option

For legacy `chan_sip`, the option is:

```ini
canreinvite=yes
```

or:

```ini
canreinvite=no
```

In newer configurations, the equivalent option is commonly:

```ini
directmedia=yes
```

or:

```ini
directmedia=no
```

A simple and safe legacy setting is:

```ini
canreinvite=no
```

This keeps RTP flowing through Asterisk.

---

# Critical Debugging Scenario

A common problem looks like this:

```text
The phone rings.
The call connects.
Audio is missing or works only one way.
```

One possible cause is direct media.

The phones may be trying to send RTP directly to each other, but a firewall, NAT device, or route blocks that path.

SIP signaling still works because both phones can reach Asterisk.

The media fails because the phones cannot reach each other directly.

---

# Signaling and Media Paths

## Default Mode

```text
Signaling:
Phone A ↔ Asterisk ↔ Phone B

Media:
Phone A ↔ Asterisk ↔ Phone B
```

## Direct Media Mode

```text
Signaling:
Phone A ↔ Asterisk ↔ Phone B

Media:
Phone A ↔ Phone B
```

---

# Comparison

| Mode | Signaling Path | Media Path | Typical Use |
|---|---|---|---|
| Default | Through Asterisk | Through Asterisk | Recording, IVR, contact centers, simpler networking |
| Direct Media | Through Asterisk | Direct between phones | Lower server load and local calls |

---

# Summary

The basic SIP call flow uses:

```text
INVITE
100 Trying
180 Ringing
200 OK
ACK
BYE
```

SIP controls the call.

RTP carries the media.

With re-invite or direct media, signaling remains through Asterisk while RTP flows directly between the phones.

Understanding this is important when debugging:

- No audio
- One-way audio
- Recording failures
- DTMF problems
- Firewall and NAT issues

---

# Next Step

The next tutorial will cover live SIP debugging and how to read SIP messages passing through Asterisk.
