# Applications Part 1: Playback & Audio Files

## Recap and Goal

In the previous part, we used several basic Asterisk dialplan applications:

```asterisk
NoOp()
Hangup()
Dial()
```

Now we will explore the `Playback()` application.

The long-term goal is to build an IVR menu where a caller can:

1. Hear an announcement or list of options.
2. Press a key to select an option.

To create that flow, we need:

- An application that plays audio.
- An application that reads DTMF key presses.

This lesson focuses on audio playback. DTMF input will be covered separately.

---

# 1. What Is the `Playback()` Application?

The `Playback()` application plays an audio file to the caller.

Basic syntax:

```asterisk
Playback(filename)
```

Example:

```asterisk
Playback(tt-monkeys)
```

When Asterisk reaches this application, it searches for the requested audio prompt and plays it on the active call channel.

---

# 2. Adding `Playback()` to the Dialplan

Open the dialplan file:

```bash
nano /etc/asterisk/extensions.conf
```

Example dialplan:

```ini
[phones]
exten => 100,1,NoOp(First Line)
exten => 100,2,Playback(tt-monkeys)
exten => 100,3,Dial(SIP/james)
exten => 100,4,Hangup()
```

When extension `100` is dialed, Asterisk performs the following steps:

1. Prints `First Line` in the CLI.
2. Plays the `tt-monkeys` audio prompt.
3. Waits for playback to finish.
4. Rings the SIP peer named `james`.
5. Ends the call when execution reaches `Hangup()`.

---

# 3. Cleaner Priority Syntax

The same dialplan can be written using:

```ini
same => n
```

Example:

```ini
[phones]
exten => 100,1,NoOp(First Line)
same => n,Playback(tt-monkeys)
same => n,Dial(SIP/james)
same => n,Hangup()
```

The letter `n` means:

```text
Next priority
```

This makes the dialplan easier to edit and maintain.

---

# 4. `Playback()` Is a Blocking Application

`Playback()` is a blocking or long-running application.

This means the dialplan remains inside the application until the audio file finishes playing.

Example:

```ini
same => n,Playback(tt-monkeys)
same => n,Dial(SIP/james)
```

Asterisk does not execute `Dial()` immediately.

The execution flow is:

```text
Start Playback
      ↓
Play complete audio file
      ↓
Playback finishes
      ↓
Continue to Dial
```

The destination phone rings only after the audio prompt ends.

---

# 5. Finding Available Audio Files

Asterisk sound files are commonly stored under:

```text
/var/lib/asterisk/sounds/
```

List the directory:

```bash
ls -lah /var/lib/asterisk/sounds/
```

You may see language-specific directories such as:

```text
en/
en_US/
fr/
de/
es/
```

The exact directories depend on which sound packages were installed.

---

## English Sound Files

English prompts may be stored under:

```text
/var/lib/asterisk/sounds/en/
```

List them using:

```bash
ls -lah /var/lib/asterisk/sounds/en/
```

Search for test prompts:

```bash
find /var/lib/asterisk/sounds -type f -name 'tt-*'
```

A common test file is:

```text
tt-monkeys
```

The physical files may include formats such as:

```text
tt-monkeys.wav
tt-monkeys.gsm
tt-monkeys.ulaw
tt-monkeys.alaw
```

Availability varies by the installed sounds package.

---

# 6. Language-Based Sound Selection

Asterisk channels can have a language setting.

For example:

```text
en
```

When the channel language is English and the dialplan requests:

```asterisk
Playback(tt-monkeys)
```

Asterisk may search inside the English sound directory automatically.

Conceptually:

```text
Channel language: en
Requested prompt: tt-monkeys
Resolved location: /var/lib/asterisk/sounds/en/tt-monkeys.<format>
```

This allows the same dialplan to support multiple languages.

For example:

```text
/var/lib/asterisk/sounds/en/welcome.wav
/var/lib/asterisk/sounds/fr/welcome.wav
/var/lib/asterisk/sounds/de/welcome.wav
```

The dialplan can request:

```asterisk
Playback(welcome)
```

Asterisk then chooses the version that matches the channel language.

---

# 7. Rule One: Do Not Use the Full Linux Path

Incorrect:

```asterisk
Playback(/var/lib/asterisk/sounds/en/tt-monkeys)
```

Recommended:

```asterisk
Playback(tt-monkeys)
```

Using the logical filename allows Asterisk to:

- Search its configured sound directories.
- Select the correct language directory.
- Select a compatible audio format.
- Keep dialplan paths portable.

A relative subdirectory can be used when prompts are organized into folders.

Example:

```asterisk
Playback(company/welcome)
```

This may resolve to a file such as:

```text
/var/lib/asterisk/sounds/en/company/welcome.wav
```

---

# 8. Rule Two: Do Not Add the File Extension

Incorrect:

```asterisk
Playback(tt-monkeys.wav)
```

Recommended:

```asterisk
Playback(tt-monkeys)
```

Asterisk can search for supported versions of the same prompt.

For example:

```text
tt-monkeys.wav
tt-monkeys.gsm
tt-monkeys.ulaw
tt-monkeys.alaw
```

The available format modules and channel codec influence which file Asterisk can use.

Leaving out the extension gives Asterisk more flexibility.

---

# 9. Audio Format and Codec Selection

A caller's audio channel uses a codec such as:

```text
ulaw
alaw
gsm
g722
```

Asterisk may have several versions of the same prompt in different file formats.

Example:

```text
welcome.ulaw
welcome.alaw
welcome.gsm
welcome.wav
```

When possible, Asterisk selects a format it can play efficiently.

If the stored audio format does not match the channel's media format, Asterisk may need to transcode the audio.

---

## What Is Transcoding?

Transcoding means converting audio from one format or codec to another.

Example:

```text
Stored prompt: WAV
Channel codec: A-law
```

Asterisk may need to convert the audio during playback.

Transcoding consumes:

- CPU resources
- Memory
- Processing time

On a small test server, this may not be noticeable.

On a large call-center server with hundreds of simultaneous calls, unnecessary transcoding can become expensive.

---

# 10. Preparing Prompts for the Main Codec

If the deployment primarily uses A-law, prompts can be prepared in A-law-compatible format.

If it primarily uses μ-law, prompts can be prepared in μ-law-compatible format.

Examples:

```text
welcome.alaw
welcome.ulaw
```

This may reduce real-time transcoding.

However, production audio preparation should also consider:

- Sample rate
- Bit depth
- Mono versus stereo
- Codec support
- Recording quality
- Provider codec negotiation
- Future codec changes

Asterisk telephony prompts are commonly prepared as mono audio at telephony-compatible sample rates.

---

# 11. Reloading the Dialplan

After editing `extensions.conf`, connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Reload the dialplan:

```asterisk
dialplan reload
```

Verify extension `100`:

```asterisk
dialplan show 100@phones
```

Expected logic:

```text
NoOp
Playback
Dial
Hangup
```

---

# 12. Testing `Playback()`

From a registered phone or softphone, dial:

```text
100
```

Expected behavior:

1. The call enters the `phones` context.
2. Asterisk executes `NoOp()`.
3. Asterisk opens the `tt-monkeys` sound file.
4. The caller hears the complete prompt.
5. The caller cannot skip it using normal key presses.
6. Playback finishes.
7. Asterisk executes `Dial(SIP/james)`.
8. James's phone begins ringing.

---

# 13. Expected CLI Output

The CLI may display output similar to:

```text
Executing [100@phones:1] NoOp("SIP/matthias-00000001", "First Line")
Executing [100@phones:2] Playback("SIP/matthias-00000001", "tt-monkeys")
Playing 'tt-monkeys.gsm' (language 'en')
Executing [100@phones:3] Dial("SIP/matthias-00000001", "SIP/james")
Called SIP/james
```

The exact channel name and selected file format may differ.

For example, Asterisk may play:

```text
tt-monkeys.ulaw
```

instead of:

```text
tt-monkeys.gsm
```

depending on installed files, codecs, and format modules.

---

# 14. `Playback()` Does Not Normally Accept DTMF Input

During normal `Playback()` execution, key presses do not usually interrupt the audio or select an IVR option.

This makes `Playback()` useful for announcements that must play completely.

Examples:

- Legal notices
- Call-recording notices
- Mandatory security warnings
- Welcome messages
- Informational prompts

For an interactive IVR where callers can press keys during the audio, applications such as the following are more appropriate:

```asterisk
Background()
WaitExten()
Read()
```

These will be introduced in later dialplan lessons.

---

# 15. Playing Multiple Audio Files

Multiple prompts can be played in sequence.

Example:

```ini
[phones]
exten => 100,1,Playback(welcome)
same => n,Playback(please-wait)
same => n,Dial(SIP/james,20)
same => n,Hangup()
```

Each `Playback()` finishes before the next application begins.

Some Asterisk versions also support joining filenames with an ampersand:

```asterisk
Playback(welcome&please-wait)
```

Confirm the exact behavior using:

```asterisk
core show application Playback
```

---

# 16. Playing a Custom Audio File

Suppose you create a custom announcement named:

```text
company-welcome.wav
```

Copy it into the appropriate sounds directory:

```bash
cp company-welcome.wav /var/lib/asterisk/sounds/en/
```

Set ownership and permissions:

```bash
chown asterisk:asterisk /var/lib/asterisk/sounds/en/company-welcome.wav
chmod 644 /var/lib/asterisk/sounds/en/company-welcome.wav
```

Use it in the dialplan without the path or extension:

```ini
same => n,Playback(company-welcome)
```

---

# 17. Organizing Custom Prompts

For larger systems, create a dedicated subdirectory.

Example:

```bash
mkdir -p /var/lib/asterisk/sounds/en/company
```

Copy prompts:

```bash
cp welcome.wav /var/lib/asterisk/sounds/en/company/
cp sales-menu.wav /var/lib/asterisk/sounds/en/company/
cp support-menu.wav /var/lib/asterisk/sounds/en/company/
```

Set ownership:

```bash
chown -R asterisk:asterisk /var/lib/asterisk/sounds/en/company
```

Use them in the dialplan:

```ini
Playback(company/welcome)
```

```ini
Playback(company/sales-menu)
```

This keeps custom prompts separate from Asterisk's built-in sound files.

---

# 18. Common Problems

## Audio File Not Found

The CLI may display a warning similar to:

```text
File tt-monkeys does not exist in any format
```

Check whether the file exists:

```bash
find /var/lib/asterisk/sounds -type f -name 'tt-monkeys*'
```

Possible causes:

- The sound package is not installed.
- The filename is incorrect.
- The language directory is different.
- The file permissions are incorrect.
- The required format module is not loaded.
- An extension was included incorrectly.

---

## Wrong Language Directory

Check the current channel language in the dialplan:

```ini
same => n,NoOp(Channel language is ${CHANNEL(language)})
```

You can set it explicitly:

```ini
same => n,Set(CHANNEL(language)=en)
same => n,Playback(tt-monkeys)
```

---

## Unsupported File Format

Check loaded format modules:

```asterisk
module show like format_
```

Examples may include:

```text
format_wav
format_gsm
format_pcm
format_mp3
```

If the required format module is missing, Asterisk cannot play that file type.

---

## File Permissions

Check permissions:

```bash
ls -l /var/lib/asterisk/sounds/en/
```

The Asterisk service user must be able to read the file.

Example:

```bash
chown asterisk:asterisk /var/lib/asterisk/sounds/en/company-welcome.wav
chmod 644 /var/lib/asterisk/sounds/en/company-welcome.wav
```

---

## Playback Is Silent

Possible causes:

- RTP ports are blocked.
- The call has one-way audio.
- The wrong sound file was selected.
- The audio format is invalid.
- The file contains unsupported encoding.
- The channel was not answered in the current call flow.

For some incoming call scenarios, answer the channel before playback:

```ini
exten => 100,1,Answer()
same => n,Playback(tt-monkeys)
same => n,Hangup()
```

---

# 19. Useful CLI Commands

## Show Playback Documentation

```asterisk
core show application Playback
```

## Show Available Applications

```asterisk
core show applications
```

## Search for Playback-Related Applications

```asterisk
core show applications like play
```

## Show Loaded Format Modules

```asterisk
module show like format_
```

## Reload the Dialplan

```asterisk
dialplan reload
```

## Inspect the Extension

```asterisk
dialplan show 100@phones
```

## Increase CLI Verbosity

```asterisk
core set verbose 3
```

---

# 20. Minimal Working Example

## `/etc/asterisk/extensions.conf`

```ini
[phones]
exten => 100,1,NoOp(Playing the test prompt)
same => n,Playback(tt-monkeys)
same => n,Dial(SIP/james,20)
same => n,Hangup()
```

Reload:

```asterisk
dialplan reload
```

Verify:

```asterisk
dialplan show 100@phones
```

Dial:

```text
100
```

Expected result:

```text
The caller hears the complete test prompt.
After playback finishes, James's phone rings.
```

---

# 21. Final Verification Checklist

- [ ] The `Playback()` application is available.
- [ ] `core show application Playback` displays documentation.
- [ ] The requested audio file exists.
- [ ] The filename is used without a full Linux path.
- [ ] The filename is used without an extension.
- [ ] The correct language sound directory is installed.
- [ ] A compatible format module is loaded.
- [ ] The Asterisk user can read the audio file.
- [ ] The dialplan reloads without errors.
- [ ] Dialing extension `100` plays the prompt.
- [ ] Playback finishes before `Dial()` executes.
- [ ] James's phone rings after the audio ends.
- [ ] Custom prompts are stored in an organized subdirectory.
- [ ] Frequently used prompts are prepared in efficient telephony formats where appropriate.

At this stage, Asterisk can play announcements before continuing the call flow. The next step toward an IVR is to collect and process DTMF key presses from the caller.
