# Codecs Explained & SIP Debugging Introduction

## What Is a Codec?

Human voice begins as analog sound.

A VoIP system converts that sound into digital packets.

The method used to encode, compress, and carry that audio is called a codec.

Different codecs provide different levels of:

- Compression
- Audio quality
- Bandwidth usage

---

# Essential Codecs

## 1. G.711 — A-law and μ-law

G.711 is commonly associated with traditional ISDN-quality voice.

| Variant | Common Region |
|---|---|
| A-law | Europe |
| μ-law | North America |

### Bandwidth

```text
Payload bitrate: 64 Kbps
Total bandwidth with overhead: Approximately 87 Kbps
```

For simple planning, this can be rounded to roughly:

```text
100 Kbps per call
```

G.711 is available with Asterisk.

---

## 2. G.729 — Bandwidth-Saving Codec

G.729 uses less bandwidth than G.711.

```text
Payload bitrate: 8 Kbps
Total bandwidth with overhead: Approximately 31 Kbps
```

It provides voice quality close to traditional telephony while reducing bandwidth usage.

G.729 transcoding may require commercial licensing.

---

## 3. G.722 — HD Voice Codec

G.722 provides wideband or HD voice quality.

It carries a wider voice-frequency range, making speech sound clearer.

G.722 is available with Asterisk.

Many SIP carriers still use G.711 for external calls, so HD quality may only be available internally.

To hear the difference, the phone should have:

- G.722 support
- A suitable microphone
- A suitable speaker

---

## 4. Opus

Opus can provide high-quality audio at relatively low bitrates.

Asterisk supports Opus through its Opus codec module.

Before using it, confirm:

- The module is installed
- Both endpoints support Opus
- The SIP provider supports Opus for external calls

---

# Bandwidth Comparison

| Codec | Payload Bitrate | Approximate Total with Overhead |
|---|---:|---:|
| G.711 | 64 Kbps | 87 Kbps |
| G.729 | 8 Kbps | 31 Kbps |

---

# Bandwidth Is Not the Only Factor

Call quality also depends on:

- Latency
- Packet loss
- Jitter
- Connection reliability

A stable connection is more important than raw bandwidth alone.

---

# SIP Debugging Introduction

SIP debugging is useful for:

- Registration problems
- Failed calls
- Routing issues
- Authentication failures
- Missing audio
- Unexpected SIP responses

---

# 1. Asterisk CLI Debugging

For legacy `chan_sip`:

```asterisk
sip set debug on
```

Disable it with:

```asterisk
sip set debug off
```

For PJSIP:

```asterisk
pjsip set logger on
```

Disable it with:

```asterisk
pjsip set logger off
```

On a busy system, live debug output can scroll too quickly to analyze.

---

# 2. Capture SIP Traffic with `tcpdump`

```bash
tcpdump -i any -w sip-capture.pcap port 5060
```

This command:

- Listens on all interfaces
- Captures port `5060`
- Saves the traffic to `sip-capture.pcap`

## Capture Process

1. Start `tcpdump`.
2. Reproduce the SIP problem.
3. Stop the capture.
4. Copy the PCAP file to your local computer.
5. Open it in Wireshark.

---

# 3. Wireshark

Wireshark allows you to inspect:

- SIP requests
- SIP responses
- Message headers
- SDP details
- Complete call flow

It makes it easier to follow a call from `INVITE` through ringing, answer, and hangup.

---

# Recommended Resource

The tutorial recommends a SIP troubleshooting video on the Kamailio World YouTube channel.

It covers:

- SIP debugging tools
- Common mistakes
- Typical SIP problems
- Practical troubleshooting methods

---

# Summary

A codec determines how voice is encoded into digital packets.

Important codecs include:

- G.711
- G.729
- G.722
- Opus

For SIP debugging:

1. Use the Asterisk CLI for live debugging.
2. Use `tcpdump` to capture traffic.
3. Open the capture in Wireshark.
4. Follow the complete SIP message flow.

---

# Next Step

The next tutorial will use Wireshark to inspect SIP traces, understand call flow, and identify problems.
