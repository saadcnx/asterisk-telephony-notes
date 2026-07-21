# Music on Hold Configuration

## Quick Troubleshooting: `sip show peers` Not Working

Before configuring music on hold, here are a few checks for the error where:

```asterisk
sip show peers
```

is not recognized.

---

# Possible Cause 1: `chan_sip` Is Not Loaded

Check whether the legacy SIP module is loaded:

```asterisk
module show like chan_sip
```

The required module is:

```text
chan_sip.so
```

If it is loaded, Asterisk should recognize legacy `sip` CLI commands.

You can also type:

```text
sip
```

and press the `Tab` key inside the Asterisk CLI.

If SIP commands appear through tab completion, the module is available.

> Modern Asterisk systems may use PJSIP instead of `chan_sip`. In that case, commands such as `pjsip show endpoints` are used instead.

---

# Possible Cause 2: DNS Resolution Problems

The SIP module may need DNS resolution when a SIP carrier or remote peer is configured using a hostname.

Example:

```text
sip.provider.example
```

If DNS is unavailable or misconfigured, the module may load slowly while waiting for DNS timeouts.

Check that:

- The configured DNS server is reachable.
- Domain names resolve correctly.
- `/etc/resolv.conf` contains valid DNS servers.
- The server can resolve the provider hostname.

Example test:

```bash
getent hosts sip.provider.example
```

---

# The Problem: No Music on Hold in the Queue

When a caller dials the queue extension:

```text
300
```

the Asterisk CLI may show that the default music-on-hold class is being used, but the caller hears silence or only ringback.

One common cause is that the channel has not been answered before entering the queue.

---

# The Fix: Answer the Call First

Add:

```asterisk
Answer()
```

before the `Queue()` application.

Example:

```ini
exten => 300,1,Answer()
exten => 300,2,Queue(support,,,,60)
exten => 300,3,Hangup()
```

Cleaner syntax:

```ini
exten => 300,1,Answer()
same => n,Queue(support,,,,60)
same => n,Hangup()
```

Once the call is answered, Asterisk can send music or announcements to the caller while they wait.

---

# The Music on Hold Configuration File

The configuration file is:

```text
/etc/asterisk/musiconhold.conf
```

Asterisk organizes music-on-hold settings into classes.

The default class is:

```ini
[default]
```

---

# Basic Music on Hold Class

```ini
[default]
mode=files
directory=/var/lib/asterisk/moh
```

---

# Important Parameters

| Parameter | Description |
|---|---|
| `mode` | Defines how the music is played |
| `directory` | Directory containing the audio files |
| `sort` | Controls the playback order |

---

# Music on Hold Modes

## `files`

```ini
mode=files
```

Asterisk reads audio files directly from a directory.

This is the recommended and simplest mode.

## `custom`

```ini
mode=custom
```

Asterisk starts an external application to provide audio.

Older configurations sometimes used tools such as:

```text
mpg123
```

or a streaming client.

---

# Why `files` Mode Is Preferred

With external players, playback may already be in progress when a caller joins.

The caller may hear the audio from the middle of the track rather than from the beginning.

This can be a problem when the audio contains:

- Recorded announcements
- Promotions
- Support instructions
- Important caller information

With `files` mode, Asterisk manages playback directly and can provide more predictable behavior for callers and announcements.

---

# Creating Multiple Music on Hold Classes

Different classes can be created for:

- Support queues
- Sales queues
- Different languages
- Promotional messages
- Different departments

Example class:

```ini
[support]
mode=files
directory=/var/lib/asterisk/moh/support
```

---

# Creating the Support Music Directory

Create the directory:

```bash
mkdir -p /var/lib/asterisk/moh/support
```

Set ownership:

```bash
chown asterisk:asterisk /var/lib/asterisk/moh/support
```

Copy an audio file:

```bash
cp /var/lib/asterisk/moh/your-file.wav /var/lib/asterisk/moh/support/
```

Set ownership on the files:

```bash
chown asterisk:asterisk /var/lib/asterisk/moh/support/*
```

---

# Reloading Music on Hold

Inside the Asterisk CLI, run:

```asterisk
moh reload
```

---

# Verifying Music on Hold Classes

Run:

```asterisk
moh show classes
```

You should see classes such as:

```text
default
support
```

---

# Using a Different Music on Hold Class

There are two main methods.

---

# Method 1: Queue Configuration

Inside:

```text
/etc/asterisk/queues.conf
```

set:

```ini
[support]
musiconhold = support
```

The support queue will then use the `support` music-on-hold class.

Reload the queue configuration:

```asterisk
queue reload all
```

---

# Method 2: Channel Variable

Set the music class before entering the queue.

Example:

```ini
exten => 300,1,Answer()
exten => 300,n,Set(CHANNEL(musicclass)=support)
exten => 300,n,Queue(support,,,,60)
exten => 300,n,Hangup()
```

Cleaner syntax:

```ini
exten => 300,1,Answer()
same => n,Set(CHANNEL(musicclass)=support)
same => n,Queue(support,,,,60)
same => n,Hangup()
```

This method is more dynamic because the music class can be changed according to dialplan conditions.

---

# Announcements Mixed with Music

Custom audio files can contain both music and spoken announcements.

Example flow:

```text
Music plays
      ↓
Music volume decreases
      ↓
Thank you for calling our support team.
If you do not want to wait, please email us.
      ↓
Music volume increases again
```

Different queues can use different content, such as:

- Support information
- Sales promotions
- Online-store discounts
- Waiting-time instructions
- Alternative contact methods

---

# Audio File Recommendations

Asterisk can play audio formats supported by its loaded format modules.

The simplest choice is usually:

```text
WAV
```

For telephony audio:

- Use mono audio.
- Avoid unnecessary stereo.
- Avoid extremely high bitrates.
- Keep files small and efficient.
- Use a sample rate appropriate for the codecs used by the system.

There is usually no benefit in using CD-quality stereo audio for a telephone call.
