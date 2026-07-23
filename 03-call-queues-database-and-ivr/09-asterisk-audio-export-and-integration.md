# Exporting & Integrating Custom Audio into Asterisk

## Recap

Previously, we recorded and edited a high-quality prompt in Audacity.

Now we need to prepare that file for Asterisk by:

- Converting it to mono
- Using the correct sample rate
- Exporting it in a suitable WAV format
- Copying it to the Asterisk server
- Playing it from the dialplan

---

# Step 1: Prepare the Audio in Audacity

A raw recording may not be suitable for standard telephony use.

Two common issues are:

- Stereo audio
- A sample rate that is too high

---

# 1. Convert Stereo to Mono

A voice microphone recording is normally mono.

If Audacity shows two identical waveforms, the file may contain the same audio in both the left and right channels.

This is effectively dual mono.

## Action

Split the stereo track into mono tracks.

Keep one mono track and delete the duplicate.

You can also configure Audacity to record in mono from the beginning.

---

# When to Work in Stereo

If you plan to mix:

- Voice
- Background music
- Multiple sound effects

it may be easier to work in stereo during production.

After completing the mix, convert the final result to mono before exporting it for standard telephony use.

---

# 2. Downsample to 8000 Hz

Common sample rates include:

```text
CD quality: 44,100 Hz
Standard telephony: 8,000 Hz
```

Standard A-law and μ-law telephony audio commonly uses:

```text
8,000 Hz
```

## Action

Change the Audacity project rate from:

```text
44,100 Hz
```

to:

```text
8,000 Hz
```

At 8,000 Hz, the audio sounds more like a telephone call.

That is expected for standard narrowband telephony codecs.

---

# HD Codec Audio

For wideband codecs such as G.722, a higher audio rate may be used.

Example:

```text
16,000 Hz
```

The required format depends on the codec and audio format being used.

---

# 3. Keep the Original Master

Always keep the original Audacity project in high quality.

Example:

```text
44,100 Hz
Stereo, if music was mixed
Original uncompressed project
```

Only create the lower-quality telephony export when preparing the final Asterisk file.

Once audio has been downsampled, the removed quality cannot be restored.

Keep the master file for:

- Future editing
- Different codecs
- HD voice systems
- New exports
- Other platforms

---

# 4. Command-Line Conversion with SoX

SoX is a command-line audio conversion tool.

It is useful when many files need to be converted.

For example, if a recording studio provides 50 prompt files, manually opening every file in Audacity would be inefficient.

A command-line tool can batch-process them.

It is better to receive high-quality master files from the studio and perform the Asterisk conversion yourself.

This preserves the original quality for future use.

---

# Step 2: Export from Audacity

Select the final mono track.

Use:

```text
File → Export → Export Selected Audio
```

or:

```text
File → Export → Export Audio
```

Use the following export settings:

| Setting | Value |
|---|---|
| Format | WAV |
| Encoding | Signed 16-bit PCM |
| Sample Rate | 8,000 Hz |
| Channels | 1 — Mono |

Example filename:

```text
asterisk-tutorial.wav
```

Save the file to the local computer.

---

# Verify the Exported File

Check the file information or properties.

Confirm:

```text
Sample Rate: 8,000 Hz
Channels: 1
Bit Depth: 16-bit
```

The file size should be relatively small.

A rough estimate may be around:

```text
1 MB per minute
```

depending on the exact WAV format.

---

# Step 3: Copy the File to the Asterisk Server

Use SCP or another file-transfer tool.

Example:

```bash
scp asterisk-tutorial.wav root@asterisk-server:/root/
```

This copies the file to:

```text
/root/asterisk-tutorial.wav
```

Move or copy it into the Asterisk sounds directory:

```bash
cp /root/asterisk-tutorial.wav /var/lib/asterisk/sounds/
```

The final location becomes:

```text
/var/lib/asterisk/sounds/asterisk-tutorial.wav
```

---

# Step 4: Use the Prompt in the Dialplan

Create a test extension:

```ini
exten => *600,1,Answer()
exten => *600,n,Playback(asterisk-tutorial)
exten => *600,n,Hangup()
```

Cleaner syntax:

```ini
exten => *600,1,Answer()
same => n,Playback(asterisk-tutorial)
same => n,Hangup()
```

---

# Important `Playback()` Rules

## Do Not Include the File Extension

Use:

```asterisk
Playback(asterisk-tutorial)
```

Do not use:

```asterisk
Playback(asterisk-tutorial.wav)
```

Asterisk searches for supported versions of the requested filename.

---

# Do Not Include the Full Path

Use:

```asterisk
Playback(asterisk-tutorial)
```

Asterisk searches its configured sounds directories automatically.

The file is stored at:

```text
/var/lib/asterisk/sounds/asterisk-tutorial.wav
```

but the dialplan references only:

```text
asterisk-tutorial
```

---

# Step 5: Test the Prompt

Reload the dialplan:

```asterisk
dialplan reload
```

Dial:

```text
*600
```

The call should:

1. Answer.
2. Play the custom prompt.
3. Hang up.

---

# Test on the Actual Target System

Always listen to the prompt through:

- A real IP phone
- A softphone
- An actual call through Asterisk

Audio may sound different over a telephone codec than it does through studio speakers or headphones.

The prompt may be:

- Too loud
- Too quiet
- Distorted
- Difficult to understand
- Too compressed

If the level is incorrect:

1. Return to Audacity.
2. Adjust gain or amplification.
3. Export the file again.
4. Replace the existing file.
5. Test again through Asterisk.

---

# Complete Workflow

```text
Record in high quality
        ↓
Edit mistakes and silence
        ↓
Apply compression if needed
        ↓
Mix voice and music
        ↓
Convert final audio to mono
        ↓
Downsample to 8,000 Hz
        ↓
Export as 16-bit PCM WAV
        ↓
Copy to /var/lib/asterisk/sounds/
        ↓
Use Playback(filename)
        ↓
Test through a real phone
        ↓
Adjust and export again if required
```

---

# Summary

The complete process is:

1. Record and produce the prompt in Audacity.
2. Keep the original high-quality project.
3. Convert the final version to mono.
4. Downsample it to 8,000 Hz for standard telephony.
5. Export it as a signed 16-bit PCM WAV file.
6. Copy it to:

```text
/var/lib/asterisk/sounds/
```

7. Play it using:

```asterisk
Playback(asterisk-tutorial)
```

8. Test it from a real phone or softphone.

---

# Next Step

The next tutorial will combine custom prompts with dialplan logic to build an interactive voice response menu.
