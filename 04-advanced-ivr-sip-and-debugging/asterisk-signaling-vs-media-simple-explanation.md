# Signaling vs. Media

## The Analogy: Visiting a Friend

Imagine you are visiting a friend.

The process has two separate parts.

## Signaling

```text
Arriving at the house
Ringing the doorbell
Opening the door
Saying goodbye
Leaving
```

## Media

```text
Sitting inside the room
Having the actual conversation
```

The same principle applies in telecom and VoIP systems such as Asterisk.

Signaling and media are two separate things.

---

# What Is Signaling?

Signaling is the messaging process responsible for:

- Starting a call
- Managing a call
- Routing a call
- Ending a call

Without signaling, phones would not know:

- When to connect
- Which destination to call
- Which route to use
- When the call has ended

In VoIP, signaling is commonly handled by:

```text
SIP
```

SIP stands for:

```text
Session Initiation Protocol
```

---

# The Four Main Tasks of Signaling

## 1. User Location Check

The system checks where extensions are currently available.

Example:

```text
Extension 101 → Current IP address and status
Extension 102 → Current IP address and status
```

This allows Asterisk to know where registered phones can be reached.

---

## 2. Call Setup

Phone A sends a request to Asterisk.

Asterisk sends the call toward Phone B.

Important SIP messages include:

```text
INVITE
180 Ringing
```

The `INVITE` starts the call request.

The `180 Ringing` response indicates that the destination phone is ringing.

---

## 3. Media Negotiation

The devices must agree on how they will exchange audio.

For example:

```text
Phone A supports G.722.
Phone B also supports G.722.
```

This negotiation is described by:

```text
SDP
```

SDP stands for:

```text
Session Description Protocol
```

SDP is carried inside SIP messages.

It describes information such as:

- Supported codecs
- Media IP addresses
- RTP port numbers

---

## 4. Call Teardown

When a user hangs up, signaling ends the call and releases the call resources.

The main SIP message used is:

```text
BYE
```

The `BYE` message tells the other side that the established call should end.

---

# What Is Media?

Media is the actual voice data carried during the conversation.

In VoIP, media is commonly transported using:

```text
RTP
```

RTP stands for:

```text
Real-time Transport Protocol
```

RTP carries the voice packets between the devices.

---

# Signaling vs. Media

| Feature | Signaling | Media |
|---|---|---|
| Main job | Sets up, manages, and ends the call | Carries the actual voice |
| Protocol | SIP | RTP |
| Common ports | UDP `5060` or `5061` | UDP `10000–20000` |
| Asterisk's role | Processes the SIP messages | Can relay the audio or allow direct media |

---

# Asterisk's Role

## Signaling

Signaling normally goes through Asterisk.

Asterisk manages:

- Who calls whom
- Which phone should ring
- Call routing
- Call state
- Call teardown

Asterisk acts as the control point for the call.

## Media

Media can follow two possible paths.

### Through Asterisk

```text
Phone A ↔ Asterisk ↔ Phone B
```

This is useful for:

- Call recording
- IVR
- DTMF handling
- Audio processing

### Directly Between Phones

```text
Phone A ↔ Phone B
```

This is called:

```text
Direct media
```

or:

```text
Re-invite
```

It can reduce the media load on the Asterisk server.

---

# Why This Distinction Matters

Suppose:

```text
The destination phone rings.
The call connects.
There is no audio.
```

The signaling is working because SIP successfully established the call.

The problem is likely in the media path.

Possible causes include:

- RTP ports are blocked
- Firewall rules are incorrect
- NAT information is wrong
- Audio is allowed in only one direction
- Direct media cannot reach the other phone

---

# One-Way Audio

A common problem is:

```text
Phone A can hear Phone B
Phone B cannot hear Phone A
```

This usually means one direction of the RTP stream is blocked or incorrectly routed.

The SIP signaling may still work normally.

---

# Debugging Rule

```text
Phone does not ring → Check SIP signaling
Phone rings but no audio → Check RTP media
One-way audio → Check RTP, firewall, and NAT
```

Signaling and media should be debugged separately.

---

# Summary

```text
SIP controls the call.
RTP carries the voice.
```

Signaling handles:

- User location
- Call setup
- Codec and media negotiation
- Call teardown

Media carries:

- The actual conversation
- Voice packets

Understanding this separation is essential for:

- Technical interviews
- SIP troubleshooting
- Firewall configuration
- NAT debugging
- One-way audio problems
