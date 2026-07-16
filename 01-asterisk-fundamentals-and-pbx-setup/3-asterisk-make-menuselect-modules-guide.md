# Asterisk `make menuselect` — Module Selection Guide

When you run:

```bash
make menuselect
```

Asterisk opens a terminal-based module selection interface. The categories appear on the left side, and you can enable or disable individual modules depending on your requirements.

Below are the most important categories and modules.

---

## A. Channel Drivers (`chan_*`) — The Connection Makers

Channel drivers determine how the Asterisk server connects to different types of phones, trunks, or telephony networks.

| Module | Status | What It Does |
|---|---|---|
| `chan_pjsip` | 🟢 **Must be enabled** | The primary module for modern SIP communication. Most current VoIP phones, SIP trunks, and gateways use PJSIP. It is normally enabled by default. Do not disable it in a modern SIP setup. |
| `chan_sip` | 🔴 **Deprecated / Optional** | The legacy SIP driver. In newer Asterisk versions, it may be removed or disabled by default. Use `chan_pjsip` instead. |
| `chan_iax2` | 🟡 **Optional** | Provides the IAX2 protocol, commonly used to connect one Asterisk server to another. For example, it can connect a Lahore office PBX with a Karachi office PBX. |

---

## B. Applications (`app_*`) — The Feature Providers

Application modules perform actions during a call. When a call reaches Asterisk, these modules provide features such as dialing, queues, voicemail, and conference rooms.

| Module | Status | What It Does |
|---|---|---|
| `app_dial` | 🟢 **Must be enabled** | Provides the `Dial()` application, which rings another phone, endpoint, or trunk and connects the call. Without it, normal outbound or extension-to-extension dialing will not work. |
| `app_queue` | 🟢 **Call center module** | Manages call queues, agents, waiting callers, queue announcements, and call distribution. It is essential for call centers and helplines. |
| `app_voicemail` | 🟢/🟡 **Voicemail** | Records voicemail messages when a user does not answer and provides mailbox access for listening to stored messages. |
| `app_confbridge` | 🟢 **Conference calling** | Creates conference or meeting rooms where three or more participants can join the same call. |

---

## C. Codecs (`codec_*`) — The Audio Translators

A codec converts voice into digital audio packets and determines the audio quality, compression level, and bandwidth usage.

| Module | Status | What It Does |
|---|---|---|
| `codec_ulaw` / `codec_alaw` | 🟢 **Must be enabled** | Provides G.711 μ-law and A-law audio support. These are high-quality, widely supported VoIP codecs and are normally enabled by default. |
| `codec_g729` | 🟡 **Bandwidth saver** | Provides low-bandwidth G.729 audio support. It can reduce bandwidth usage, but availability may depend on licensing and codec installation requirements. |
| `codec_gsm` | 🟡 **Low bandwidth / mobile-like quality** | Provides GSM-compressed audio with lower quality and reduced bandwidth usage. |

> **Note:** Codec availability can vary by Asterisk version and installed dependencies. Some codecs may require separate licensing or external packages.

---

## D. File Formats (`format_*`) — The Audio File Readers

Format modules allow Asterisk to read or write different audio file formats. These are used for voicemail, system prompts, announcements, recordings, and music on hold.

| Module | Status | What It Does |
|---|---|---|
| `format_wav` | 🟢 **Enabled by default** | Allows Asterisk to read and write standard `.wav` audio files. |
| `format_mp3` | 🔴 **Optional / usually disabled by default** | Allows Asterisk to play MP3 files. Before enabling it, you may need to download the required MP3 source code. |

To download the MP3 source dependencies, run this command from the Asterisk source directory:

```bash
contrib/scripts/get_mp3_source.sh
```

After running the script, open `make menuselect` again and enable the MP3 format module if it becomes available.

> For production systems, WAV or GSM files are often preferred because they require less real-time decoding than MP3 files.

---

## E. Resource Modules (`res_*`) — The System Helpers

Resource modules provide backend services such as media transport, encryption, database connectivity, configuration storage, and protocol support.

| Module | Status | What It Does |
|---|---|---|
| `res_rtp_asterisk` | 🟢 **Must be enabled** | Provides RTP support. SIP usually handles call setup and signaling, while RTP carries the actual voice audio. Without RTP, calls may connect but have no audio. |
| `res_odbc` | 🟡 **Database connector** | Allows Asterisk to connect to databases through ODBC. It is commonly used with CDR, CEL, realtime configuration, queues, and other database-backed features. |
| `res_config_mysql` | 🟡 **Legacy / version-dependent database connector** | Provides MySQL-backed realtime configuration in some Asterisk builds. Modern installations often use ODBC instead. |

---

## Quick Guide: What to Keep Enabled or Disabled

Asterisk normally enables the essential modules automatically. For a basic SIP installation, the default selections are usually sufficient.

| Module or Category | Default Status | Recommended Action |
|---|---|---|
| PJSIP and RTP | 🟢 ON | Leave enabled. SIP devices and voice media depend on these modules. |
| WAV format | 🟢 ON | Leave enabled. It is useful for system prompts, announcements, recordings, and music on hold. |
| MP3 format | 🔴 OFF | Enable only when you specifically need to play `.mp3` files. |
| `app_voicemail` | 🟢 ON | Leave enabled unless you are intentionally creating a minimal build. |
| MySQL / ODBC modules | 🔴 Usually OFF | Enable only when Asterisk needs to connect to an SQL database. |
| English sound files | 🟢 ON | Keep installed for standard system prompts such as busy, unavailable, or invalid-number announcements. |

---

## Recommended Modules for a Basic SIP Server

For a normal SIP-only Asterisk server, make sure the following components are available:

```text
chan_pjsip
app_dial
res_rtp_asterisk
codec_ulaw
codec_alaw
format_wav
```

For a call center setup, also keep:

```text
app_queue
app_confbridge
app_voicemail
res_odbc
```

---

## Saving Your Selection

Inside the `make menuselect` interface:

1. Use the arrow keys to move between categories and modules.
2. Press the **Spacebar** or **Enter** key to enable or disable a module.
3. Select **Save & Exit** when finished.
4. Compile Asterisk using:

```bash
make
```

Then install it using:

```bash
make install
```
