# SIP Debugging: Capturing Traffic with TCPDump

## Two Approaches to SIP Debugging

There are two common ways to debug SIP traffic:

1. Asterisk CLI debugging
2. TCPDump with Wireshark

---

# 1. Asterisk CLI Debugging

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Enable SIP debugging:

```asterisk
sip set debug on
```

You can enable debugging globally or for a specific peer.

Example:

```asterisk
sip set debug ip <peer_name>
```

## Best Use

Asterisk CLI debugging is useful for:

- Small test systems
- A few SIP peers
- Quick troubleshooting
- Viewing SIP messages live

## Limitation

On systems with many calls, the output may scroll too quickly to analyze.

After reproducing the issue, exit the CLI so new output does not continue pushing the relevant debug messages off the screen.

You can then scroll back and inspect the captured output.

---

# 2. TCPDump and Wireshark

A more professional approach is:

1. Capture network traffic on the Asterisk server.
2. Save it to a file.
3. Copy the file to a local computer.
4. Open it in Wireshark.
5. Analyze the SIP and RTP traffic.

This method is better for detailed troubleshooting.

---

# Why Capture on the Server?

Wireshark running on a laptop or desktop normally captures traffic from that local machine's network interfaces.

The SIP calls, however, are happening on the Asterisk server.

The solution is to run:

```text
tcpdump
```

on the server and save the packets to a PCAP file.

The PCAP file can then be opened locally in Wireshark for offline analysis.

---

# Basic TCPDump Capture

```bash
tcpdump -i eth0 -w test.pcap
```

| Parameter | Meaning |
|---|---|
| `-i eth0` | Capture traffic from interface `eth0` |
| `-w test.pcap` | Write packets to `test.pcap` |

This creates a packet-capture file that can be opened in Wireshark.

The exact amount of packet data captured depends on the `tcpdump` version and its snapshot-length default.

---

# Explicit Full-Packet Capture

To explicitly request complete packets, use:

```bash
tcpdump -i eth0 -s 0 -w test.pcap
```

| Parameter | Meaning |
|---|---|
| `-s 0` | Capture the complete packet instead of limiting the snapshot length |

This is useful when troubleshooting RTP and audio-related problems.

---

# Why Full Capture Matters

A full capture includes:

- Packet headers
- SIP messages
- SDP content
- RTP payloads
- Other packet data

This is useful for investigating:

- One-way audio
- Missing audio
- RTP packet loss
- Jitter
- Media-path problems

Audio analysis works only when the RTP media is not encrypted.

---

# File Size Difference

Capturing complete packets creates larger PCAP files because RTP traffic is included.

SIP signaling alone usually creates relatively small files.

RTP media generates many packets and can increase file size quickly.

---

# Avoid Filtering Too Early

You may be tempted to capture only SIP port `5060`:

```bash
tcpdump -i eth0 port 5060 -w test.pcap
```

This limits the capture to traffic matching port `5060`.

For broad troubleshooting, it is often better to capture all relevant traffic and apply filters later in Wireshark.

The issue may involve:

- RTP ports
- DNS
- ICMP errors
- NAT behavior
- Another protocol
- General network problems

Filtering too early may remove useful evidence.

---

# Long-Term Capture with Rotating Files

A long-running capture can consume large amounts of disk space.

Use rotating files to control storage usage:

```bash
tcpdump -i eth0 -s 0 -w capture.pcap -C 100 -W 10
```

| Parameter | Meaning |
|---|---|
| `-C 100` | Start a new file when the current file reaches approximately 100 MB |
| `-W 10` | Keep a maximum of 10 rotating files |

This keeps approximately the most recent:

```text
1 GB
```

of captured traffic.

When the maximum number of files is reached, the oldest capture is overwritten.

This is useful when waiting for an intermittent problem to occur.

---

# Basic Debugging Workflow

## Step 1: Start the Capture

```bash
tcpdump -i eth0 -s 0 -w test.pcap
```

## Step 2: Reproduce the Problem

Place the failing call or reproduce the SIP issue.

## Step 3: Stop the Capture

Press:

```text
Ctrl+C
```

## Step 4: Copy the File

Use SCP or another file-transfer tool.

Example:

```bash
scp root@asterisk-server:/root/test.pcap .
```

## Step 5: Open in Wireshark

Use Wireshark to analyze:

- SIP messages
- SDP negotiation
- Call flow
- RTP streams
- Packet loss
- Audio problems

---

# Choosing the Right Method

| Method | Best For | Limitation |
|---|---|---|
| Asterisk CLI debug | Quick testing on small systems | Output becomes difficult to follow on busy systems |
| TCPDump and Wireshark | Detailed and professional troubleshooting | Requires capture, transfer, and offline analysis |

---

# Summary

For quick SIP debugging:

```asterisk
sip set debug on
```

For detailed packet capture:

```bash
tcpdump -i eth0 -s 0 -w test.pcap
```

For long-running rotating captures:

```bash
tcpdump -i eth0 -s 0 -w capture.pcap -C 100 -W 10
```

The recommended workflow is:

```text
Capture traffic
      ↓
Reproduce the issue
      ↓
Stop the capture
      ↓
Copy the PCAP file
      ↓
Open it in Wireshark
      ↓
Analyze SIP and RTP
```

---

# Next Step

The next tutorial will open the PCAP file in Wireshark and analyze a real SIP call flow.
