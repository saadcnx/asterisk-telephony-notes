# Asterisk Database (AstDB) Basics & Functions

## Why Use the Database?

To build a professional dynamic queue login/logout system with a single toggle extension, we need a way to store the current login state.

The Asterisk Database, commonly called:

```text
AstDB
```

provides simple persistent storage inside Asterisk.

It can be used for many different purposes, including dynamic queue membership state.

---

# What Is the Asterisk Database?

AstDB is a built-in key-value database inside Asterisk.

It stores data in a simple structure:

```text
/family/key = value
```

---

# Key Characteristics

## Persistent

AstDB data survives:

- Asterisk reloads
- Asterisk restarts
- Server reboots

The data is stored on disk and loaded again when Asterisk starts.

## Portable

The database file can be copied to another Asterisk server for:

- Migration
- Backup
- Recovery

## Already Used by Asterisk

Asterisk itself can store information in AstDB.

For example, persistent dynamic queue members may appear in a structure similar to:

```text
/Queue/PersistentMembers/support = SIP/james
```

---

# Database Structure

AstDB uses three main parts:

```text
/family/key = value
```

| Part | Purpose |
|---|---|
| Family | Groups related entries |
| Key | Identifies a specific item |
| Value | Stores the associated data |

Example:

```text
/pascom/james = ok
```

Here:

```text
Family: pascom
Key: james
Value: ok
```

---

# CLI Commands

The following commands are run inside the Asterisk CLI.

## Show the Complete Database

```asterisk
database show
```

This displays all stored AstDB entries.

---

# Add or Update an Entry

Syntax:

```asterisk
database put family key value
```

Example:

```asterisk
database put pascom james ok
```

Result:

```text
/pascom/james = ok
```

If the key already exists, its value is updated.

---

# Use a Deeper Family Path

Example:

```asterisk
database put pascom/test james ok
```

Result:

```text
/pascom/test/james = ok
```

This creates a deeper organizational path.

---

# Delete a Specific Key

Syntax:

```asterisk
database del family key
```

Example:

```asterisk
database del pascom james
```

This removes:

```text
/pascom/james
```

---

# Delete an Entire Family

Syntax:

```asterisk
database deltree family
```

Example:

```asterisk
database deltree pascom
```

This removes all keys stored under:

```text
/pascom
```

Use this command carefully because it deletes the complete family tree.

---

# Functions vs Applications

So far, the dialplan has mainly used applications.

Examples:

```text
Dial()
VoiceMail()
Queue()
Playback()
```

Applications perform actions.

Functions return or manipulate values that can be used by dialplan logic.

---

# Finding Database Functions

Inside the Asterisk CLI, run:

```asterisk
core show functions like DB
```

---

# Main AstDB Functions

| Function | Purpose |
|---|---|
| `DB_EXISTS(family/key)` | Returns whether a key exists |
| `DB(family/key)` | Reads the value of a key |
| `DB_DELETE(family/key)` | Deletes a key |
| `DB(family/key)=value` | Stores a value through `Set()` |

---

# `DB_EXISTS()`

Syntax:

```asterisk
DB_EXISTS(family/key)
```

It returns:

```text
1
```

if the key exists.

It returns:

```text
0
```

if the key does not exist.

In a `GotoIf()` condition:

- `0` is false
- Any non-zero value is true

---

# Testing `DB_EXISTS()` in the Dialplan

Create a test extension:

```ini
exten => *300,1,NoOp(Database Test)
exten => *300,n,GotoIf($[${DB_EXISTS(pascom/james)}]?300-ok)
exten => *300,n,Hangup()

exten => 300-ok,1,Answer()
exten => 300-ok,n,Playback(beep)
exten => 300-ok,n,Hangup()
```

---

# How the Test Works

## Step 1: Check the Key

```ini
GotoIf($[${DB_EXISTS(pascom/james)}]?300-ok)
```

Asterisk checks whether this key exists:

```text
/pascom/james
```

## If the Key Exists

`DB_EXISTS()` returns:

```text
1
```

The condition is true, so Asterisk jumps to:

```text
300-ok
```

Then it:

1. Answers the call.
2. Plays a beep.
3. Hangs up.

## If the Key Does Not Exist

`DB_EXISTS()` returns:

```text
0
```

The condition is false.

Execution continues to:

```ini
exten => *300,n,Hangup()
```

The call ends without playing a beep.

---

# Testing When the Entry Does Not Exist

Make sure the key is absent:

```asterisk
database del pascom james
```

Dial:

```text
*300
```

Expected result:

- `DB_EXISTS()` returns `0`.
- The jump does not happen.
- The call hangs up immediately.
- No beep is played.

---

# Testing When the Entry Exists

Add the entry:

```asterisk
database put pascom james ok
```

Dial:

```text
*300
```

Expected result:

- `DB_EXISTS()` returns `1`.
- Asterisk jumps to `300-ok`.
- The call is answered.
- A beep is played.
- The call hangs up.

---

# Reading a Database Value

Use:

```asterisk
${DB(family/key)}
```

Example:

```ini
exten => *301,1,NoOp(Value is ${DB(pascom/james)})
same => n,Hangup()
```

If the database contains:

```text
/pascom/james = ok
```

then:

```asterisk
${DB(pascom/james)}
```

returns:

```text
ok
```

---

# Writing a Value from the Dialplan

Use the `Set()` application with the `DB()` function:

```ini
Set(DB(pascom/james)=ok)
```

Example:

```ini
exten => *302,1,Set(DB(pascom/james)=ok)
same => n,Hangup()
```

This creates or updates:

```text
/pascom/james = ok
```

---

# Deleting a Value from the Dialplan

Use:

```asterisk
DB_DELETE(family/key)
```

with `Set()`:

```ini
Set(RESULT=${DB_DELETE(pascom/james)})
```

This deletes:

```text
/pascom/james
```

The deleted value is stored in:

```asterisk
${RESULT}
```

---

# Why This Matters for Queue Login

AstDB gives us the ability to store queue-login state.

For example:

```text
/pascom/james = logged-in
```

The dialplan can then:

1. Check whether James is currently logged in.
2. Make a decision based on the result.
3. Add or remove James from the queue.
4. Update the AstDB entry.

The logic becomes:

```text
If login key exists
    Remove James from queue
    Delete login key
Otherwise
    Add James to queue
    Create login key
```

This makes it possible to use one extension as a login/logout toggle.

---

# Next Step

The next step is to combine:

- `DB_EXISTS()`
- `DB()`
- `DB_DELETE()`
- `AddQueueMember()`
- `RemoveQueueMember()`
- `GotoIf()`

to build a single-number dynamic queue login/logout system.
