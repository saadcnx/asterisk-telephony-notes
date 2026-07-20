# Voicemail Configuration Basics

## The Problem

Configuring voicemail in Asterisk requires two separate steps:

1. Configure the voicemail system itself.
2. Integrate voicemail into the dialplan.

This lesson focuses on the first step: mailbox and voicemail-system configuration.

---

# The Voicemail Configuration File

The main file is:

```text
/etc/asterisk/voicemail.conf
```

The default file is often large because it includes documentation and sample settings.

## Cleanup Steps

Create a backup:

```bash
cd /etc/asterisk
cp voicemail.conf voicemail.conf.original
```

In Vim, remove comment lines:

```vim
:g/^\s*;/d
```

Remove empty lines:

```vim
:g/^\s*$/d
```

This leaves a shorter and more readable configuration file.

---

# Understanding Contexts

`voicemail.conf` uses contexts, like `extensions.conf` and `sip.conf`.

## The `[general]` Context

The `[general]` context contains global voicemail settings, including:

- Recording format
- Email attachments
- Maximum silence
- Email notification text

### Recording Format

The format setting defines which audio formats Asterisk uses for voicemail recordings.

Example:

```ini
format=wav49|gsm|wav
```

### Email Attachments

Asterisk can attach the recorded voicemail file to an email notification.

### Maximum Silence

This setting controls how long Asterisk waits during silence before stopping the recording.

### Email Text

The subject and body of voicemail notification emails can also be customized.

A common workflow is:

1. The caller records a voicemail.
2. Asterisk stores it locally.
3. Asterisk creates an email.
4. The voicemail file is attached.
5. The email is sent to the mailbox owner.

Voicemails can also remain on the local server and be accessed by dialing into the voicemail system.

---

# The `[default]` Context

The `[default]` context contains mailbox definitions.

Example:

```ini
[default]
123 => 1234,Example Mailbox,user@example.com
```

## Mailbox Format

```text
MailboxNumber => PIN,Name,EmailAddress
```

| Field | Description |
|---|---|
| `MailboxNumber` | Voicemail mailbox ID |
| `PIN` | Password used to access the mailbox |
| `Name` | Mailbox owner's name |
| `EmailAddress` | Address for voicemail notifications |

---

# Why Use Multiple Contexts?

You can create separate contexts for different groups.

Examples:

```ini
[default]
[mycompany]
[support]
```

This helps organize mailboxes by department, company, location, or language.

The same mailbox number can exist in different contexts:

```text
100@mycompany
100@support
```

These are treated as separate mailboxes.

---

# Simple Mailbox Setup

For this lab, use the default context:

```ini
[default]
100 => 1234,James,james@example.com
200 => 1234,Matthias,matthias@example.com
```

This creates:

| Mailbox | PIN | Name | Email |
|---|---:|---|---|
| `100` | `1234` | James | `james@example.com` |
| `200` | `1234` | Matthias | `matthias@example.com` |

The mailbox numbers match the extension numbers because they are easier to remember, although this is not required.

---

# Reloading the Voicemail Configuration

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Reload voicemail settings:

```asterisk
voicemail reload
```

A full Asterisk restart is not required.

## Show Configured Mailboxes

Inside the Asterisk CLI, run:

```asterisk
voicemail show users
```

You should see mailboxes such as:

```text
100@default
200@default
```

---

# Where Voicemails Are Stored

Asterisk stores voicemail files under:

```text
/var/spool/asterisk/voicemail/
```

Even when email delivery is enabled, the recording is first created locally.

## Directory Structure

A folder is created for each context:

```text
/var/spool/asterisk/voicemail/default/
```

Inside the context folder, Asterisk creates a folder for each mailbox:

```text
/var/spool/asterisk/voicemail/default/100/
/var/spool/asterisk/voicemail/default/200/
```

A mailbox folder is normally created when the first voicemail is left.

---

# Mailbox Folders

Inside a mailbox directory, Asterisk creates folders such as:

| Folder | Purpose |
|---|---|
| `INBOX/` | New and unread messages |
| `Old/` | Messages that have been listened to |
| `tmp/` | Temporary files used during recording |

Example:

```text
/var/spool/asterisk/voicemail/default/100/INBOX/
```

---

# Files Inside `INBOX`

A voicemail may be stored in multiple audio formats:

```text
msg0000.wav
msg0000.gsm
msg0000.WAV
```

A metadata file may also be created:

```text
msg0000.txt
```

Messages are numbered sequentially:

```text
msg0000
msg0001
msg0002
```

The exact audio formats depend on the `format` setting in `voicemail.conf`.

---

# Why This Storage Location Matters

## Migration

When moving Asterisk to another server, copy:

```text
/var/spool/asterisk/voicemail/
```

to preserve existing voicemail messages.

## Storage Planning

If many voicemails are expected, a separate disk or network storage can be mounted at:

```text
/var/spool/asterisk/voicemail/
```

---

# Current Status

The voicemail system is now configured.

Two mailboxes exist:

```text
100@default
200@default
```

The next step is to integrate the voicemail application into the dialplan so unanswered calls can be sent to the correct mailbox.
