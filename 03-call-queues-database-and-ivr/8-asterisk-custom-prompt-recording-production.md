# Custom Prompts: Recording & Production

## Why Custom Prompts Are Needed

Custom prompts are required whenever the PBX needs to play a business-specific message.

Common examples include:

- IVR menus
- Business-hours messages
- Holiday announcements
- Queue announcements
- Department greetings
- Dialplan decision points

So far, built-in Asterisk audio files such as:

```text
tt-monkeys
```

have been used for testing. A real system needs custom recordings.

---

# Two Recording Methods

## Method 1: Phone Recording

A quick and inexpensive method using an Asterisk dialplan extension.

## Method 2: Professional Production

A higher-quality method using recording software and a proper microphone.

---

# Method 1: Recording Through Asterisk

Create a dialplan extension that records audio directly from a phone.

```ini
exten => *555,1,NoOp(Recording)
exten => *555,n,Answer()
exten => *555,n,Record(test.wav)
exten => *555,n,Hangup()
```

## How It Works

Dial:

```text
*555
```

The call flow is:

1. Asterisk answers the call.
2. A beep plays.
3. Recording starts immediately.
4. Speak the message.
5. Press `#` to stop.
6. The dialplan continues to `Hangup()`.

Pressing `#` stops the recording more cleanly than simply hanging up.

---

# Where the File Is Saved

The file is saved under:

```text
/var/lib/asterisk/sounds/
```

Example:

```text
/var/lib/asterisk/sounds/test.wav
```

It is saved in the main sounds directory rather than a language-specific directory such as:

```text
/var/lib/asterisk/sounds/en/
```

This allows the prompt to work regardless of the channel language setting.

---

# Recorded File Format

The dialplan uses:

```asterisk
Record(test.wav)
```

The resulting file is:

```text
test.wav
```

It is mono audio and suitable for normal telephony use.

The approximate size may be around:

```text
1 MB per minute
```

depending on the exact WAV format used.

---

# Drawbacks of Phone Recording

## No Editing

You cannot easily:

- Remove mistakes
- Trim silence
- Cut unwanted parts
- Improve the beginning or ending

## Lower Quality

Phone and softphone microphones may introduce:

- Background noise
- Echo
- Weak microphone quality
- Codec limitations
- Uneven volume

## No Mixing

You cannot easily combine:

- Voice and music
- Multiple clips
- Background announcements
- Different audio tracks

## Timing Problems

If speaking starts too early or too late, the complete prompt usually has to be recorded again.

---

# Method 2: Professional Production with Audacity

For customer-facing prompts, use recording software and a proper microphone.

A recommended tool is:

```text
Audacity
```

Audacity is free and open-source and is available on:

- Windows
- Linux
- macOS
- BSD and other supported platforms

---

# What Audacity Provides

## Clean Recording

A proper microphone can provide much better quality than a phone microphone.

An entry-level USB microphone in the approximate range of:

```text
€20–€100
```

can be enough for clear PBX prompts.

## Editing

Audacity allows you to:

- Remove mistakes
- Cut unwanted sections
- Trim silence
- Adjust timing
- Remove clicks
- Arrange multiple clips

## Compression

A compressor effect can make speech:

- Clearer
- More consistent
- Louder
- Easier to understand

Compression reduces the difference between quiet and loud sections without causing distortion when configured correctly.

## Mixing Music and Voice

Audacity can combine background music with voice announcements.

Example:

```text
Music plays
      ↓
Music fades down
      ↓
Thank you for calling our support team.
      ↓
Music fades back up
```

## Multi-Track Production

Multiple tracks can combine:

- Voice recordings
- Background music
- Multiple speakers
- Different microphones
- Sound effects
- Separate language recordings

---

# Recording Quality Matters Most

The microphone and recording environment matter more than simply choosing between:

```text
WAV
```

and:

```text
GSM
```

Important factors include:

- Microphone quality
- Quiet environment
- Distance from the microphone
- Room echo
- Speaking clarity
- Consistent volume

---

# Multilingual Business Prompts

International businesses may need prompts in multiple languages.

For better results:

- Use native speakers.
- Record separate files for each language.
- Keep wording consistent.
- Use a professional voice agency when needed.

Professional services may start around:

```text
€50–€100 per file
```

depending on language, voice actor, and production requirements.

---

# Comparison

| Aspect | Phone Recording | Audacity and Microphone |
|---|---|---|
| Cost | Free with an existing phone | Microphone may cost €20–€100 |
| Quality | Basic or poor | Very good to professional |
| Editing | None | Full editing |
| Silence Removal | Not practical | Easy |
| Compression | Not available | Supported |
| Music Mixing | Not practical | Easy |
| Multi-Track | Not available | Supported |
| Best Use | Testing and personal use | Production and customer-facing systems |

---

# Recommended Use

## Phone Recording

Use for:

- Quick testing
- Temporary messages
- Personal systems
- Development environments

## Professional Recording

Use for:

- Customer-facing IVRs
- Business greetings
- Queue announcements
- Music on hold
- Production PBX systems
- Multilingual prompts

---

# Next Step

After recording and editing the audio, the files must be:

1. Converted into suitable Asterisk audio formats.
2. Placed in the correct Asterisk sounds directory.
3. Referenced correctly from the dialplan.

Audio conversion and file placement will be covered in the next tutorial.
