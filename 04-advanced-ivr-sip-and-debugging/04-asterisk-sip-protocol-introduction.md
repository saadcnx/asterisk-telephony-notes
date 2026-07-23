# SIP Protocol Introduction

## Why Learn SIP Now?

We have been using SIP throughout the tutorial series without fully understanding how it works.

Now we need to connect Asterisk to:

- Real SIP providers
- SIP trunks
- VoIP gateways
- External phone systems

Copying a configuration template is not enough.

Without understanding the basic protocol, troubleshooting becomes difficult when calls fail, registration does not work, or audio is missing.

---

# What Is SIP?

SIP stands for:

```text
Session Initiation Protocol
```

The most important point is that SIP is used to initiate and control a session.

The session itself may contain:

- Voice
- Video
- Data transfer
- Screen sharing

SIP starts, manages, and ends the session.

---

# SIP Call State

SIP manages call states such as:

```text
Call setup
Ringing
Connected
Hangup
```

It can maintain session state while using a transport protocol such as UDP.

---

# SIP Transport and Port 5060

## Default Transport

SIP commonly uses:

```text
UDP
```

UDP stands for:

```text
User Datagram Protocol
```

UDP itself does not maintain a connection state.

SIP handles the required call and transaction state at the application-protocol level.

## Default Port

The standard SIP port is commonly:

```text
5060
```

Example:

```text
SIP over UDP → Port 5060
```

---

# SIP over TCP and TLS

Asterisk can also use SIP over TCP.

Encrypted SIP signaling commonly uses:

```text
SIP over TLS
```

For the basic examples in this tutorial, SIP uses:

```text
UDP port 5060
```

---

# Session Description Protocol

SIP commonly carries another protocol called:

```text
SDP
```

SDP stands for:

```text
Session Description Protocol
```

SDP describes the media session that SIP is establishing.

---

# What SDP Describes

SDP includes information such as:

- Supported codecs
- Media IP addresses
- Media port numbers
- Audio or video capabilities

A codec determines how speech is converted into digital audio packets.

---

# SIP and SDP Negotiation

The basic process is:

1. SIP messages are exchanged over port `5060`.
2. SDP information is included inside the SIP messages.
3. Each side announces its supported codecs.
4. Each side announces the IP address and port where it expects media.
5. Both sides agree on compatible media settings.
6. The actual voice stream begins.

Example SDP meaning:

```text
I support these codecs.
Send audio to this IP address.
Send audio to this UDP port.
```

The actual voice is not normally sent over SIP port `5060`.

---

# Signaling vs Media

This is one of the most important SIP concepts.

```text
SIP Signaling → UDP Port 5060
RTP Media     → High UDP Ports
```

## SIP Signaling

SIP handles:

- Call setup
- Ringing
- Answering
- Call state
- Hangup

## RTP Media

RTP carries:

- Voice
- Audio
- Video media

Asterisk commonly uses an RTP range such as:

```text
UDP 10000–20000
```

---

# Separate Network Paths

The SIP signaling path and RTP media path are separate.

They can even travel through different network routes.

This separation is important when troubleshooting firewall and NAT problems.

---

# Common Firewall Mistake

A common mistake is opening only:

```text
UDP 5060
```

This may allow SIP signaling to work.

The result can be:

- The call starts
- The destination rings
- The call is answered
- No audio is heard

---

# Why One-Way Audio Happens

Suppose port `5060` is open, but the RTP ports are blocked.

SIP signaling works, so the call can be established.

However, the voice packets cannot pass through the firewall.

Another possibility is that RTP is allowed in only one direction.

Example:

```text
Audio from A to B works
Audio from B to A is blocked
```

This produces:

```text
One-way audio
```

One person can hear the other person, but the audio does not work in the opposite direction.

---

# Firewall Requirements

For the default Asterisk RTP range, allow:

```text
UDP 10000–20000
```

The firewall must allow the required RTP traffic in both directions.

---

# Protocol Summary

| Component | Protocol | Common Port | Purpose |
|---|---|---:|---|
| Signaling | SIP over UDP | `5060` | Call setup, state management, and teardown |
| Media Description | SDP inside SIP | — | Negotiates codecs, IP addresses, and media ports |
| Media | RTP over UDP | `10000–20000` | Carries the actual audio stream |

---

# Basic Call Setup

```text
Caller sends SIP signaling
        ↓
SIP negotiates the session
        ↓
SDP exchanges codec and media details
        ↓
Call is answered
        ↓
RTP carries the voice
        ↓
SIP ends the call
```

---

# Key Concept

```text
SIP does not normally carry the voice.
SIP controls the call.
RTP carries the voice.
```

Understanding the separation between signaling and media is essential when troubleshooting SIP calls.

---

# Next Step

The next tutorials will cover the SIP message flow, including:

```text
INVITE
200 OK
ACK
BYE
```

This knowledge will then be used to connect Asterisk to real SIP providers.
