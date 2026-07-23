# IVR Advanced: Loop Counter & Menu Repetition

## The Problem

Currently, when the caller does not press anything, the timeout path immediately sends them to a queue or switchboard.

However, the caller may simply not have understood the menu the first time.

A better flow is:

1. Play the menu.
2. Wait for input.
3. Replay the menu once or twice if there is no response.
4. Send the caller to an operator after the retry limit is reached.

To do this, we need:

- A counter variable
- A safe point to jump back to
- A condition that stops the loop

---

# The Challenges

## Challenge 1: Where Should We Jump Back?

If the dialplan jumps to:

```asterisk
Goto(start,1)
```

the counter is reset because the beginning of the IVR contains:

```asterisk
Set(Loop=0)
```

We need to jump to a point after the counter has already been initialized.

## Challenge 2: Priority Numbers Can Change

A jump such as:

```asterisk
Goto(start,3)
```

depends on a fixed priority number.

If lines are added or removed later, priority `3` may point to the wrong application.

The solution is to use a label.

---

# Step 1: Initialize the Counter

```ini
exten => start,1,NoOp(IVR 1)
exten => start,n,Set(Loop=0)
exten => start,n,Answer()
exten => start,n(loop),Playback(ivr_q1)
exten => start,n,WaitExten(5)
```

## `Set(Loop=0)`

```ini
exten => start,n,Set(Loop=0)
```

This initializes the counter:

```text
Loop = 0
```

## The `loop` Label

```ini
exten => start,n(loop),Playback(ivr_q1)
```

The label is:

```text
loop
```

The dialplan can jump back using:

```asterisk
Goto(start,loop)
```

This returns to the `Playback()` line without resetting the counter.

---

# Step 2: Increment the Counter

```ini
exten => t,n,Set(Loop=$[${Loop} + 1])
```

Asterisk performs mathematical expressions inside:

```asterisk
$[ ... ]
```

The expression:

```asterisk
$[${Loop} + 1]
```

reads the current value, adds `1`, and stores the result back in `Loop`.

---

# Step 3: Loop or Exit

```ini
exten => t,n,GotoIf($[${Loop} < 2]?start,loop)
exten => t,n,Queue(test)
exten => t,n,Hangup()
```

The condition is:

```asterisk
$[${Loop} < 2]
```

If true, Asterisk jumps to:

```text
start,loop
```

If false, execution continues to:

```asterisk
Queue(test)
```

---

# Full Dialplan

```ini
[ivr1]
exten => start,1,NoOp(IVR 1)
exten => start,n,Set(Loop=0)
exten => start,n,Answer()
exten => start,n(loop),Playback(ivr_q1)
exten => start,n,WaitExten(5)

; Valid Options
exten => 1,1,Queue(test)
exten => 1,n,Hangup()

exten => 2,1,Queue(blah)
exten => 2,n,Hangup()

; Invalid Key
exten => i,1,Playback(invalid-key)
exten => i,n,Goto(start,loop)

; Timeout with Loop Counter
exten => t,1,NoOp(Timeout - Loop: ${Loop})
exten => t,n,Set(Loop=$[${Loop} + 1])
exten => t,n,GotoIf($[${Loop} < 2]?start,loop)
exten => t,n,Queue(test)
exten => t,n,Hangup()
```

---

# Invalid Input Flow

```ini
exten => i,1,Playback(invalid-key)
exten => i,n,Goto(start,loop)
```

When the caller presses an invalid key:

1. Asterisk plays the `invalid-key` prompt.
2. It jumps back to the `loop` label.
3. The menu plays again.

Because it does not jump to `start,1`, the counter is not reset.

---

# Timeout Flow

```ini
exten => t,1,NoOp(Timeout - Loop: ${Loop})
exten => t,n,Set(Loop=$[${Loop} + 1])
exten => t,n,GotoIf($[${Loop} < 2]?start,loop)
exten => t,n,Queue(test)
exten => t,n,Hangup()
```

The timeout handler:

1. Logs the current counter.
2. Increases the counter.
3. Checks whether another replay is allowed.
4. Replays the menu or sends the caller to the queue.

---

# Testing the Flow

Reload the dialplan:

```asterisk
dialplan reload
```

Dial the IVR extension.

## First Timeout

Initial value:

```text
Loop = 0
```

After timeout:

```text
Loop = 1
```

Condition:

```text
1 < 2
```

This is true, so the menu is replayed.

## Second Timeout

Current value:

```text
Loop = 1
```

After incrementing:

```text
Loop = 2
```

Condition:

```text
2 < 2
```

This is false, so the caller is sent to:

```asterisk
Queue(test)
```

---

# Final Behavior

```text
Play menu
    ↓
First timeout
    ↓
Replay menu
    ↓
Second timeout
    ↓
Send caller to queue
```

The caller hears the menu twice before being transferred to a human.

---

# Why Labels Are Important

## Fixed Priority Number

```ini
exten => t,n,Goto(start,4)
```

This can break if lines are added or removed above priority `4`.

## Named Label

```ini
exten => t,n,Goto(start,loop)
```

This continues to point to the correct line even if priority numbers change.

Labels make the dialplan easier to maintain.

---

# Summary

Initialize the counter:

```asterisk
Set(Loop=0)
```

Increment it:

```asterisk
Set(Loop=$[${Loop} + 1])
```

Check the limit:

```asterisk
GotoIf($[${Loop} < 2]?start,loop)
```

Use the `loop` label as a reliable jump destination.
