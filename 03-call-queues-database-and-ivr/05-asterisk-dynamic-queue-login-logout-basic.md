# Dynamic Queue Login/Logout: Basic Implementation

## Why Manage Queue Membership by Dialing?

Previously, dynamic queue members were added and removed through the Asterisk CLI.

That works for administrators, but it is not practical for daily use.

An employee should not need to ask an administrator every morning:

```text
Please log me into the queue.
```

A better approach is to let users manage their own queue membership by dialing an extension.

This lesson builds a simple first implementation to understand the concept.

---

# Required Queue Applications

Search for queue-related applications inside the Asterisk CLI:

```asterisk
core show applications like queue
```

We already know:

```asterisk
Queue()
```

which sends callers into a queue.

For dynamic membership, we need:

| Application | Purpose |
|---|---|
| `AddQueueMember()` | Adds an interface as a dynamic queue member |
| `RemoveQueueMember()` | Removes a dynamic member from a queue |

---

# `AddQueueMember()` Syntax

Display its documentation:

```asterisk
core show application AddQueueMember
```

Basic syntax:

```asterisk
AddQueueMember(queuename,interface)
```

Example:

```asterisk
AddQueueMember(support,SIP/james)
```

## Parameters

| Parameter | Example | Purpose |
|---|---|---|
| `queuename` | `support` | Queue to modify |
| `interface` | `SIP/james` | Member interface to add |

---

# Preparing the Queue

James must be a dynamic member, not a static member.

Open:

```text
/etc/asterisk/queues.conf
```

Remove or comment out:

```ini
member => SIP/james
```

Matthias can remain as a static member:

```ini
[support]
member => SIP/matthias
```

This means:

- Matthias is always part of the queue.
- James must log in dynamically.

Reload the queue configuration:

```asterisk
queue reload all
```

Verify:

```asterisk
queue show support
```

James should not appear in the queue yet.

---

# Quick Login Implementation

Create a login extension for James:

```ini
exten => *201,1,Answer()
exten => *201,n,AddQueueMember(support,SIP/james)
exten => *201,n,Playback(beep)
exten => *201,n,Hangup()
```

Cleaner syntax:

```ini
exten => *201,1,Answer()
same => n,AddQueueMember(support,SIP/james)
same => n,Playback(beep)
same => n,Hangup()
```

---

# Quick Logout Implementation

Create a logout extension:

```ini
exten => *202,1,Answer()
exten => *202,n,RemoveQueueMember(support,SIP/james)
exten => *202,n,Playback(beep)
exten => *202,n,Hangup()
```

Cleaner syntax:

```ini
exten => *202,1,Answer()
same => n,RemoveQueueMember(support,SIP/james)
same => n,Playback(beep)
same => n,Hangup()
```

---

# How the Login Extension Works

## `Answer()`

```asterisk
Answer()
```

Answers the call so Asterisk can play audio feedback.

## `AddQueueMember()`

```asterisk
AddQueueMember(support,SIP/james)
```

Adds James to the `support` queue as a dynamic member.

## `Playback(beep)`

```asterisk
Playback(beep)
```

Plays a confirmation sound.

Asterisk also includes sounds such as:

```text
beep
beep-error
```

## `Hangup()`

```asterisk
Hangup()
```

Ends the call.

---

# Testing the Basic Implementation

Reload the dialplan:

```asterisk
dialplan reload
```

Check the current queue status:

```asterisk
queue show support
```

James should not be a member yet.

---

# Test Login

Dial:

```text
*201
```

Expected behavior:

1. The call is answered.
2. James is added to the queue.
3. A confirmation beep plays.
4. The call ends.

Verify:

```asterisk
queue show support
```

James should appear as a dynamic member.

---

# Test Logout

Dial:

```text
*202
```

Expected behavior:

1. The call is answered.
2. James is removed from the queue.
3. A feedback sound plays.
4. The call ends.

Verify:

```asterisk
queue show support
```

James should no longer appear.

---

# Test Login Again

Dial:

```text
*201
```

James should be added back as a dynamic member.

---

# Problems with This Basic Implementation

The configuration works, but it is not suitable for production.

---

# 1. No Duplicate Login Check

If James dials:

```text
*201
```

while already logged in, the dialplan still plays the success beep.

However, the Asterisk CLI may report that the member already exists.

This gives the user incorrect feedback.

---

# 2. No Logout Validation

If James dials:

```text
*202
```

while he is not a member, the application may fail.

The current dialplan still plays the same feedback sound without checking whether the removal succeeded.

---

# 3. Two Different Extensions

The current setup uses:

```text
*201 = Login
*202 = Logout
```

A better design would use one extension that toggles the current state:

```text
If logged out → Log in
If logged in → Log out
```

---

# 4. No Caller Verification

The current dialplan always modifies:

```text
SIP/james
```

It does not verify who dialed the extension.

Anyone with access to the dialplan context could:

- Log James into the queue
- Log James out of the queue
- Change his availability without permission

The system should verify that the caller is actually James before modifying his membership.

---

# Next Step

A production-ready solution should:

- Use one extension for login and logout
- Check the current membership status
- Verify the caller's identity
- Provide different feedback for success and failure
- Prevent duplicate login attempts
- Prevent invalid logout attempts

The next tutorial will improve this basic implementation.
