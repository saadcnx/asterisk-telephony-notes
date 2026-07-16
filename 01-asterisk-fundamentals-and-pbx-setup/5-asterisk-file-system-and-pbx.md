# Asterisk File System & PBX Basics

## Part 1: The Asterisk File System — What Goes Where?

Understanding the Asterisk file system becomes easier if you imagine it as an office building where every folder has a specific purpose.

---

## A. `/etc/asterisk` — The Rule Book / Configuration Folder

Configuration variable:

```text
ASTETCDIR=/etc/asterisk
```

This is the most important Asterisk configuration directory. You can think of it as the **brain** or **rule book** of the PBX.

It contains the `.conf` files that define how Asterisk behaves.

### Important Files

#### `pjsip.conf`

This file defines SIP-related objects such as:

- Phone extensions
- SIP endpoints
- Authentication credentials
- Address-of-record settings
- SIP trunks
- Transport settings

For example, this is where you configure phones such as extensions `101`, `102`, and `103`.

#### `extensions.conf`

This file contains the Asterisk **dialplan**.

The dialplan defines call-routing rules, such as:

```text
If someone dials 101, where should the call go?
```

It controls actions such as:

- Calling another extension
- Sending a call to voicemail
- Routing outbound calls
- Playing announcements
- Sending calls into a queue
- Transferring calls to another destination

---

## B. `/usr/sbin/asterisk` — The Engine / Main Software

Configuration variable:

```text
DAEMON=/usr/sbin/asterisk
```

This is the main Asterisk executable binary.

You can think of it as the **engine** that runs the PBX.

On Windows, you might double-click an `.exe` file to start a program. On Linux, `/usr/sbin/asterisk` is the executable that starts the Asterisk process.

When Asterisk runs as a daemon, this program stays active in the background and handles:

- SIP signaling
- Calls
- Dialplan execution
- Audio processing
- Queues
- Voicemail
- Conference rooms
- Other PBX features

You can verify its location using:

```bash
which asterisk
```

or:

```bash
type asterisk
```

---

## C. `/var/lib/asterisk` — The Storage Room / Data Library

This directory stores static data that Asterisk needs while operating.

You can think of it as the PBX's **storage room**.

### Common Subdirectories

#### `/var/lib/asterisk/sounds`

This directory stores system voice prompts and audio files.

Examples include messages such as:

```text
The number you have dialed is busy.
```

It may contain:

- English system prompts
- Custom announcements
- IVR recordings
- Error messages
- Number and date prompts

#### `/var/lib/asterisk/moh`

`moh` stands for **Music on Hold**.

This folder stores audio files that Asterisk plays while callers are waiting or placed on hold.

Common formats include:

```text
.wav
.ulaw
.alaw
.gsm
```

MP3 playback may require additional modules or dependencies.

---

## D. `/var/log/asterisk` — The Diary / Log Folder

This directory stores Asterisk logs.

You can think of it as the PBX's **diary**, because it records system activity and problems.

Logs may contain information about:

- Calls starting and ending
- SIP registrations
- Authentication failures
- Dialplan errors
- Module-loading problems
- Queue activity
- Network or codec issues
- Warnings and notices

### Common Troubleshooting File

A commonly used log file is:

```text
/var/log/asterisk/messages
```

Depending on the logger configuration and Asterisk version, you may also see:

```text
/var/log/asterisk/full
```

To watch logs in real time, you can use:

```bash
tail -f /var/log/asterisk/full
```

or:

```bash
tail -f /var/log/asterisk/messages
```

The exact log files are controlled through:

```text
/etc/asterisk/logger.conf
```

---

## E. `/var/spool/asterisk` — The Runtime Data / Dynamic Folder

This directory stores data that Asterisk creates or changes while the system is running.

You can think of it as the PBX's **working area**.

### Common Subdirectories

#### `/var/spool/asterisk/voicemail`

This folder stores voicemail messages left by callers.

A typical voicemail structure may be organized by:

- Voicemail context
- Mailbox number
- Inbox
- Old messages
- Urgent messages
- Temporary messages

#### `/var/spool/asterisk/monitor`

This directory is commonly used for call recordings.

Recorded calls may be saved as:

```text
.wav
.gsm
.ulaw
```

The exact recording location can be changed in the dialplan or Asterisk configuration.

Other runtime-generated files may also be stored under `/var/spool/asterisk`.

---

## F. `/var/run/asterisk` — The Active Runtime / Status Folder

Configuration variable:

```text
ASTVARRUNDIR=/var/run/asterisk
```

On some newer Linux systems, this may effectively point to:

```text
/run/asterisk
```

This directory stores temporary runtime files created while Asterisk is active.

### PID File

Asterisk creates a **Process ID file**, commonly called a PID file.

The PID file tells the operating system:

```text
Asterisk is currently running, and this is its process ID.
```

A typical PID file may look like:

```text
/var/run/asterisk/asterisk.pid
```

When Asterisk stops normally, the PID file is removed.

This folder may also contain:

- Control socket files
- Runtime state files
- Temporary process-related data

---

# Quick File System Summary

| Path | Simple Meaning | Main Purpose |
|---|---|---|
| `/etc/asterisk` | Rule book / brain | Configuration files |
| `/usr/sbin/asterisk` | Engine | Main Asterisk executable |
| `/var/lib/asterisk` | Storage room | Sounds, prompts, and static data |
| `/var/log/asterisk` | Diary | Logs and troubleshooting information |
| `/var/spool/asterisk` | Working area | Voicemail, recordings, and runtime-created data |
| `/var/run/asterisk` | Active status folder | PID and temporary runtime files |

---

# Part 2: What Does PBX Mean?

PBX stands for:

```text
Private Branch Exchange
```

A PBX is a private telephone exchange used inside a business, office, organization, or call center.

It connects internal phones to each other and also manages access to external telephone networks.

---

## The Old Way — Without a PBX

Imagine an office with 10 employees.

Every employee needs a telephone.

Without a PBX, the company may need to purchase 10 separate telephone lines from a telecom provider.

That could mean:

- 10 separate external numbers
- 10 physical telephone lines
- 10 monthly bills
- Higher installation costs
- More difficult management

This becomes expensive and inefficient.

---

## The PBX Way — Using Asterisk

Instead of purchasing one external line for every employee, the company installs an Asterisk PBX server.

The office may purchase only one or two external telephone lines or SIP trunks.

Those external connections are linked to the Asterisk server.

The 10 office phones then connect to Asterisk and receive internal extension numbers.

For example:

| Employee | Extension |
|---|---:|
| Reception | `100` |
| Employee 1 | `101` |
| Employee 2 | `102` |
| Employee 3 | `103` |
| Manager | `110` |

---

## Internal Calls

Suppose extension `101` wants to call extension `102`.

The call stays inside the office network:

```text
Extension 101 → Asterisk PBX → Extension 102
```

The call does not need to pass through the external telecom provider.

This means internal extension-to-extension calls are generally free, apart from the cost of operating the local system and network.

---

## External Calls

Suppose extension `101` wants to call an outside mobile or landline number.

Asterisk routes the call through an available external line or SIP trunk:

```text
Extension 101
      ↓
Asterisk PBX
      ↓
SIP Trunk or Telecom Line
      ↓
External Phone Number
```

All office employees can share the available outside lines.

If one external line is already in use, Asterisk can use another available line, depending on the configuration.

---

## Incoming Calls

When a customer calls the company's main number, Asterisk receives the call and decides where it should go.

For example:

```text
Incoming Call
      ↓
Asterisk PBX
      ↓
Reception, IVR, Queue, or Extension
```

Asterisk can:

- Ring the receptionist
- Play an IVR menu
- Send the caller to a department
- Place the caller in a queue
- Route the call to an available agent
- Send the call to voicemail
- Record the call

---

## A Simple PBX Example

Imagine a company with:

- 10 employees
- 10 internal IP phones
- 1 Asterisk server
- 2 SIP trunk channels

The employees can make unlimited internal calls between extensions.

However, only two external calls can happen at the same time because the company has two external channels.

For example:

```text
101 calls 102       → Internal call
103 calls 104       → Internal call
105 calls a customer → Uses external channel 1
106 calls a customer → Uses external channel 2
107 calls outside    → Must wait until a channel becomes free
```

---

# PBX in Simple Terms

A PBX is an **internal telephone exchange**.

It:

- Connects employees' phones to each other
- Assigns internal extension numbers
- Routes incoming and outgoing calls
- Shares external telephone lines
- Reduces telecom costs
- Provides features such as voicemail, queues, IVR, recording, and conferencing

Asterisk is the software that can turn a Linux server into a powerful PBX.
