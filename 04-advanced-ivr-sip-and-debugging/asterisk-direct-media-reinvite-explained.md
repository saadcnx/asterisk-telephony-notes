# Direct Media / Re-Invite Explained

## The Normal Flow: Proxied Media

In the normal call flow, RTP media passes through Asterisk.

```text
Phone A → Asterisk → Phone B
```

Asterisk receives the voice packets from one phone and forwards them to the other phone.

This is called:

```text
Proxied Media
```

Asterisk stays in the middle of the media path.

---

# What Is Direct Media?

Direct media allows two phones to exchange RTP directly with each other.

Imagine Phone A and Phone B are in rooms next to each other and both are connected to the same local network.

## Normal Flow

```text
Phone A
   ↓
Asterisk server
   ↓
Phone B
```

Even though both phones are close to each other, the media travels through the Asterisk server.

This uses additional bandwidth and server resources.

## Direct Media Flow

Asterisk still handles SIP signaling, but RTP flows directly between the phones.

```text
Signaling:
Phone A ↔ Asterisk ↔ Phone B

Media:
Phone A ↔ Phone B
```

The main idea is:

```text
SIP signaling stays through Asterisk.
RTP media bypasses Asterisk.
```

---

# How Re-Invite Works

A re-INVITE changes the media path after the call has already been established.

The process is:

1. The call connects normally.
2. SIP uses `INVITE`, `200 OK`, and `ACK`.
3. Asterisk determines that both phones may communicate directly.
4. Asterisk sends a new `INVITE`.
5. Phone A is told to send RTP to Phone B.
6. Phone B is told to send RTP to Phone A.
7. Both phones begin direct RTP communication.

The new `INVITE` is called:

```text
Re-Invite
```

because it updates an already established session.

---

# Direct Media Flow

```text
Phone A                  Asterisk                  Phone B
  |                         |                         |
  |------ Initial SIP call setup through PBX ------>|
  |                         |                         |
  |<============= Direct RTP Media =================>|
```

Asterisk remains responsible for call signaling and control.

It is no longer in the RTP media path.

---

# Advantages of Direct Media

## Reduced Server CPU Usage

Asterisk does not have to relay large numbers of RTP packets.

This reduces server processing load.

## Lower Latency

The media follows a shorter path between the phones.

This can reduce voice delay.

## Reduced Bandwidth Usage

For local calls, media stays inside the local network.

The Asterisk server's external network bandwidth is not used for the RTP stream.

---

# Disadvantages of Direct Media

## No Call Recording

If RTP does not pass through Asterisk, applications such as:

```asterisk
MixMonitor()
```

cannot record the call.

Asterisk cannot record media that it does not receive.

## No Inband DTMF Detection

If DTMF tones are carried inside the audio stream, Asterisk may not be able to detect them when media bypasses the server.

This can affect IVR or feature-code behavior.

## NAT and Firewall Problems

Direct media requires both phones to communicate directly.

If one phone is inside an office and another is behind NAT at a remote location, they may not be able to reach each other.

Possible results include:

- No audio
- One-way audio
- Silent calls

---

# PJSIP Configuration

In `pjsip.conf`, direct media can be controlled with endpoint options.

Example:

```ini
[endpoint-template](!)
type=endpoint
direct_media=yes
direct_media_method=invite
```

---

# Parameter Explanation

| Parameter | Purpose |
|---|---|
| `direct_media=yes` | Allows RTP to flow directly between endpoints |
| `direct_media=no` | Forces RTP to remain through Asterisk |
| `direct_media_method=invite` | Uses a re-INVITE to change the media path |

---

# When to Enable Direct Media

Direct media may be useful for:

- Internal calls on the same LAN
- High call volumes
- Reducing Asterisk server load
- Calls that do not require recording
- Calls that do not require Asterisk to process media

---

# When to Disable Direct Media

Direct media should usually be disabled for:

- Contact centers requiring call recording
- IVR systems using in-call DTMF
- Remote workers behind NAT
- Mixed network environments
- Situations where endpoints cannot reach each other directly

---

# Comparison

| Mode | SIP Signaling | RTP Media | Main Benefit |
|---|---|---|---|
| Proxied Media | Through Asterisk | Through Asterisk | Recording, IVR, and simpler networking |
| Direct Media | Through Asterisk | Direct between phones | Lower server load and lower latency |

---

# Summary

```text
Proxied Media:
Phone A ↔ Asterisk ↔ Phone B

Direct Media:
Phone A ↔ Phone B
```

In both cases, SIP signaling remains under Asterisk's control.

Direct media changes only the RTP path.

It can improve performance, but it may break features that require Asterisk to receive the audio stream.
