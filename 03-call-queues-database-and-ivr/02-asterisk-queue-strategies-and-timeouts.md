# Queue Strategies & Understanding Timeouts

## What This Lesson Covers

This lesson focuses on two important queue concepts:

1. **Queue strategies** — how calls are distributed among queue members.
2. **Two different timeouts** — agent timeout and application timeout.

These two timeout settings are often confused because both are called `timeout`, but they control different parts of the queue process.

---

# Part 1: Queue Ring Strategies

The queue strategy is configured inside:

```text
/etc/asterisk/queues.conf
```

Example:

```ini
[support]
strategy = random
member => SIP/james
member => SIP/matthias
```

---

# Available Queue Strategies

| Strategy | How It Works | Pros and Cons |
|---|---|---|
| `ringall` | Rings all available members at the same time | Simple and commonly used |
| `leastrecent` | Rings the member who was called least recently | Provides fair distribution |
| `fewestcalls` | Rings the member who has answered the fewest calls | Useful when call lengths are similar |
| `random` | Selects a random available member | Fair and unpredictable |
| `rrmemory` | Round robin with memory | Remembers the last member called and selects the next one |
| `ordered` | Uses the member order defined in the configuration | Provides predictable ordering |
| `linear` | Always starts with the first member, then tries the next | Useful when one member should receive calls first |
| `wrandom` | Weighted random selection using member penalties | More advanced and requires penalty configuration |

---

# `ringall`

```ini
strategy = ringall
```

This rings all available members simultaneously. The first member who answers receives the call.

---

# `leastrecent`

```ini
strategy = leastrecent
```

This selects the member who was called least recently. It helps distribute calls fairly among team members.

---

# `fewestcalls`

```ini
strategy = fewestcalls
```

This selects the member who has answered the fewest calls.

It can work well when call durations are similar. It may be less fair when some agents handle long support calls and others handle short calls.

---

# `random`

```ini
strategy = random
```

This selects a random available member for each call attempt.

The same member may occasionally be selected multiple times because the strategy is random.

---

# `rrmemory`

```ini
strategy = rrmemory
```

This is round robin with memory.

Asterisk remembers the last member it called and starts with the next member for the next attempt.

---

# `ordered`

```ini
strategy = ordered
```

This follows the order of the members in the configuration file and uses that order as its predictable starting point.

---

# `linear`

```ini
strategy = linear
```

This always tries the first configured member first. If that member does not answer, it tries the next member.

This is useful when one member should receive calls first and others act as backups.

---

# `wrandom`

```ini
strategy = wrandom
```

This uses weighted random selection based on member penalties.

It is an advanced strategy and requires understanding queue-member penalties.

---

# Choosing a Strategy

## `ringall`

Useful when:

- The team is small.
- All available agents should ring.
- Members can pause or log out when unavailable.

## `random`

Useful when:

- Calls should be distributed unpredictably.
- No strict rotation is required.

## `leastrecent` or `fewestcalls`

Useful when:

- Fair distribution is important.
- The queue is used by a support team or call center.

## `linear`

Useful when:

- One member should always be tried first.
- Other members act as backups.

---

# Part 2: The Two Different Timeouts

Asterisk queues use two separate timeout values.

They do not override each other.

---

# 1. Agent Timeout

The agent timeout is configured in:

```text
queues.conf
```

Example:

```ini
timeout = 15
```

It controls how long the queue tries a member during one call attempt.

## With `ringall`

```text
Ring all available members
        ↓
Wait 15 seconds
        ↓
Nobody answers
        ↓
Stop the attempt
        ↓
Re-evaluate the members
        ↓
Ring again
```

## With `random`

```text
Select a random member
        ↓
Ring for 15 seconds
        ↓
No answer
        ↓
Select another random member
```

A common agent timeout is:

```text
15–20 seconds
```

A shorter timeout, such as:

```text
5–6 seconds
```

moves to the next attempt more quickly.

---

# 2. Application Timeout

The application timeout is configured in:

```text
extensions.conf
```

It is the final parameter of the `Queue()` application.

Example:

```asterisk
Queue(support,,,,60)
```

This means the caller can remain in the queue for a maximum of:

```text
60 seconds
```

If no member answers within 60 seconds, the caller exits the queue and the dialplan continues to the next priority.

---

# The Important Difference

## Agent Timeout

```text
How long one queue attempt rings an agent
```

## Application Timeout

```text
How long the caller remains in the queue overall
```

The application timeout does not replace the agent timeout.

Both operate at the same time but control different things.

---

# Full Example

## `queues.conf`

```ini
[support]
strategy = random
timeout = 10
member => SIP/james
member => SIP/matthias
```

## `extensions.conf`

```ini
exten => 300,1,Queue(support,,,,60)
exten => 300,2,Hangup()
```

---

# Execution Flow

```text
Caller enters support queue
        ↓
Queue randomly selects James
        ↓
James rings for 10 seconds
        ↓
No answer
        ↓
Queue randomly selects another member
        ↓
Matthias rings for 10 seconds
        ↓
No answer
        ↓
Queue continues trying members
        ↓
60-second total queue timeout expires
        ↓
Caller exits Queue()
        ↓
Hangup()
```

The random strategy may select the same member more than once.

Unlike `rrmemory`, it does not remember who was selected previously.

---

# Testing the Changes

Reload the queue configuration:

```asterisk
queue reload all
```

Reload the dialplan:

```asterisk
dialplan reload
```

Verify the queue:

```asterisk
queue show support
```

Confirm that the strategy shows:

```text
random
```

Place a test call to:

```text
300
```

Observe the Asterisk CLI to see:

- Which member is selected
- How long the member rings
- When the queue retries
- When the overall queue timeout expires
