# Queues: The Intelligent Replacement for Simple Call Groups

## Recap: The Simple Call-Group Solution

Previously, we rang multiple phones by combining peers inside one `Dial()` command:

```asterisk
Dial(SIP/james&SIP/matthias)
```

This works, but it has limitations:

- No fair caller queuing
- No dynamic agent membership
- No advanced distribution strategies

---

# Introducing the Queue

A queue is a dedicated Asterisk object that distributes calls intelligently.

A queue:

- Knows the concept of members or agents
- Allows dynamic login and logout
- Supports different ringing strategies
- Keeps callers waiting in line
- Can play hold music and announcements
- Tracks statistics such as wait time and talk time

---

# Step 1: Configuring `queues.conf`

The queue configuration file is:

```text
/etc/asterisk/queues.conf
```

## Cleanup

Create a backup:

```bash
cp /etc/asterisk/queues.conf /etc/asterisk/queues.conf.orig
```

In Vim, remove comment lines:

```vim
:g/^\s*;/d
```

Remove empty lines:

```vim
:g/^\s*$/d
```

---

# Basic Queue Definition

```ini
[general]

[support]
member => SIP/james
member => SIP/matthias
```

## Explanation

```ini
[support]
```

is the queue name.

Each `member` line adds a static queue member:

```ini
member => SIP/james
member => SIP/matthias
```

The format is:

```text
Technology/Resource
```

Examples:

```text
SIP/james
SIP/matthias
IAX2/remote-agent
DAHDI/1
```

For now, the default queue settings are used.

---

# Reloading the Queue Configuration

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Reload all queue settings:

```asterisk
queue reload all
```

---

# Verifying the Queue

Run:

```asterisk
queue show support
```

The output displays information such as:

| Field | Meaning |
|---|---|
| Calls | Number of callers currently waiting |
| Max | Maximum allowed callers |
| Strategy | Current distribution strategy |
| Hold Time | Average caller waiting time |
| Talk Time | Average call duration |
| Members | Configured agents and their status |

At the beginning, values may show:

```text
Calls: 0
Max: Unlimited
Strategy: ringall
Hold Time: 0
Talk Time: 0
```

The default `ringall` strategy rings all available members at the same time.

---

# Step 2: Sending Callers into the Queue

Display the `Queue()` application documentation:

```asterisk
core show application Queue
```

## Syntax

```asterisk
Queue(queuename[,options[,URL[,announceoverride[,timeout]]]])
```

| Parameter | Purpose |
|---|---|
| `queuename` | Name of the queue |
| `options` | Optional behavior flags |
| `URL` | Optional URL value |
| `announceoverride` | Optional announcement override |
| `timeout` | Maximum time the caller remains in the queue |

---

# Replacing the Old Dialplan

## Old Method

```ini
exten => 300,1,Dial(SIP/james&SIP/matthias,120)
```

## Queue Method

```ini
exten => 300,1,Queue(support,,,,120)
exten => 300,2,Hangup()
```

This sends the caller into the `support` queue for a maximum of:

```text
120 seconds
```

If nobody answers before the timeout expires, execution continues to the next priority.

---

# Step 3: Testing the Queue

Reload the dialplan:

```asterisk
dialplan reload
```

Dial:

```text
300
```

Expected behavior:

1. The caller enters the `support` queue.
2. James and Matthias ring at the same time.
3. The caller hears ringback tone.
4. The first member who answers receives the call.
5. If nobody answers before the queue timeout, the call continues to `Hangup()`.

The simultaneous ringing happens because the default strategy is:

```text
ringall
```

---

# Queue Timeout vs Agent Timeout

These are two different timeout values.

## Agent Timeout

The agent timeout controls how long the queue tries a member before stopping that attempt.

The default is commonly:

```text
15 seconds
```

After that period, the queue re-evaluates the available members.

## Queue Timeout

The queue timeout controls how long the caller can remain in the complete queue.

In this example:

```asterisk
Queue(support,,,,120)
```

the queue timeout is:

```text
120 seconds
```

---

# Example with `ringall`

```text
Caller enters the queue
        ↓
James and Matthias ring
        ↓
Nobody answers for 15 seconds
        ↓
Queue stops the attempt
        ↓
Queue re-evaluates the members
        ↓
Both phones ring again
```

This cycle continues until:

- A member answers
- The caller hangs up
- The 120-second queue timeout expires

---

# Example with a Sequential Strategy

```text
Ring James
      ↓
No answer after 15 seconds
      ↓
Ring Matthias
      ↓
No answer after 15 seconds
      ↓
Return to James
```

The cycle continues until someone answers or the queue timeout expires.

---

# Why Queues Are Better

The separation between agent timeout and queue timeout allows Asterisk to retry, re-evaluate members, and apply different distribution strategies.

This makes queues more flexible and intelligent than a simple multi-destination `Dial()` command.
