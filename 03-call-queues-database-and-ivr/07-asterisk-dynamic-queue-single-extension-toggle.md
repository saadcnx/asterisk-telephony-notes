# Dynamic Queue Membership: Single Extension Toggle

## The Goal

We now combine:

- Dynamic queue member management
- The Asterisk Database
- Dialplan functions
- Conditional branching

The goal is to create one extension that toggles a user's queue membership.

The behavior is:

```text
If the user is logged in → Log them out
If the user is logged out → Log them in
```

This removes the need for separate login and logout extensions.

---

# Persistent Queue Members in AstDB

When persistent dynamic queue members are enabled, Asterisk stores queue membership information in AstDB.

Display the database:

```asterisk
database show
```

You may see an entry similar to:

```text
/Q/PersistentMembers/support = SIP/james
```

The structure is:

```text
Family: Q/PersistentMembers
Key: support
Value: SIP/james
```

The exact family name may differ between Asterisk versions or configurations.

Always confirm the actual path by running:

```asterisk
database show
```

---

# The Two Main Functions

## 1. `DB()`

The `DB()` function reads a value from AstDB.

Syntax:

```asterisk
${DB(family/key)}
```

Example:

```asterisk
${DB(Q/PersistentMembers/support)}
```

This may return:

```text
SIP/james
```

or a value containing multiple queue members.

---

## 2. `REGEX()`

The `REGEX()` function searches for a pattern inside a string.

Syntax:

```asterisk
${REGEX("pattern" string)}
```

It returns:

```text
1
```

when a match is found.

It returns:

```text
0
```

when no match is found.

---

# Combining `DB()` and `REGEX()`

To check whether James exists in the persistent member value:

```asterisk
${REGEX("SIP/james" ${DB(Q/PersistentMembers/support)})}
```

This performs two steps:

1. Read the current queue-member value from AstDB.
2. Search that value for `SIP/james`.

The result is:

```text
1 = James is present
0 = James is not present
```

---

# Single-Extension Toggle Dialplan

```ini
; === Single Extension Toggle for James ===

exten => *300,1,NoOp(Queue Toggle for James)

; Check whether James exists in the persistent members value
exten => *300,n,GotoIf($[${REGEX("SIP/james" ${DB(Q/PersistentMembers/support)})}]?300-logout)

; James was not found, so log him in
exten => *300,n,AddQueueMember(support,SIP/james)
exten => *300,n,Playback(beep)
exten => *300,n,Hangup()

; James was found, so log him out
exten => 300-logout,1,RemoveQueueMember(support,SIP/james)
exten => 300-logout,n,Playback(beep)
exten => 300-logout,n,Hangup()
```

---

# How the Toggle Works

## When James Is Logged Out

The expression:

```asterisk
${REGEX("SIP/james" ${DB(Q/PersistentMembers/support)})}
```

does not find James.

It returns:

```text
0
```

The `GotoIf()` condition is false, so execution continues to the next priority.

Asterisk executes:

```asterisk
AddQueueMember(support,SIP/james)
```

James is added as a dynamic member.

Then Asterisk:

1. Plays a confirmation beep.
2. Hangs up.

---

## When James Is Logged In

The database value contains:

```text
SIP/james
```

The `REGEX()` function returns:

```text
1
```

The `GotoIf()` condition is true, so Asterisk jumps to:

```text
300-logout
```

Asterisk executes:

```asterisk
RemoveQueueMember(support,SIP/james)
```

James is removed from the queue.

Then Asterisk:

1. Plays a confirmation beep.
2. Hangs up.

---

# Complete Login Flow

```text
James dials *300
        ↓
Read persistent queue members from AstDB
        ↓
Search for SIP/james
        ↓
No match found
        ↓
AddQueueMember()
        ↓
Play beep
        ↓
Hangup
```

---

# Complete Logout Flow

```text
James dials *300
        ↓
Read persistent queue members from AstDB
        ↓
Search for SIP/james
        ↓
Match found
        ↓
Jump to 300-logout
        ↓
RemoveQueueMember()
        ↓
Play beep
        ↓
Hangup
```

---

# Testing the Toggle

Reload the dialplan:

```asterisk
dialplan reload
```

Check the current queue state:

```asterisk
queue show support
```

## First Call

Dial:

```text
*300
```

If James is currently a member:

- A beep plays.
- James is removed.

Verify:

```asterisk
queue show support
```

## Second Call

Dial:

```text
*300
```

again.

Now:

- A beep plays.
- James is added back.

Each call toggles the current membership state.

---

# Security Limitation

The current dialplan is hardcoded to modify:

```text
SIP/james
```

Anyone who can dial:

```text
*300
```

can toggle James's queue membership.

---

# Caller ID Verification

A Caller ID check can restrict the toggle to James's own extension.

Example:

```ini
exten => *300,n,GotoIf($["${CALLERID(num)}" != "200"]?hangup)
```

This means only calls with Caller ID:

```text
200
```

can continue.

A separate destination must exist:

```ini
exten => hangup,1,Hangup()
```

---

# PIN Verification

Another option is to request a PIN using:

```asterisk
Read()
```

The entered value can then be compared before allowing the queue membership change.

---

# Making the Logic Generic

The current solution is hardcoded for:

```text
Queue: support
Member: SIP/james
```

A reusable subroutine could accept:

- Queue name
- Member interface

Example calls:

```asterisk
GoSub(queue-toggle,start,1(support,SIP/james))
```

```asterisk
GoSub(queue-toggle,start,1(sales,SIP/matthias))
```

This would allow the same logic to be reused for multiple users and queues.

---

# Different Feedback Sounds

Different sounds can identify which action occurred.

Example:

```text
beep = Login successful
beep-error = Logout successful
```

This allows the user to recognize the result without checking the queue status.

---

# Final Result

The single-extension toggle combines:

- `DB()`
- `REGEX()`
- `GotoIf()`
- `AddQueueMember()`
- `RemoveQueueMember()`
- Persistent queue-member data

The final behavior is:

```text
One extension
      ↓
Check current state
      ↓
Log in or log out
      ↓
Provide audio feedback
```

This creates a cleaner and more user-friendly dynamic queue membership system.
