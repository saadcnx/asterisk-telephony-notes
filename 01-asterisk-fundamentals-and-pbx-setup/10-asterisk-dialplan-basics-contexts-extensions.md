# Dialplan Basics: Contexts & Extensions

## The Problem Recap

Previously, two SIP phones were registered successfully, but they could not call each other.

When one phone dialed an extension, Asterisk returned an error similar to:

```text
Call from 'Matthias' to extension '100' rejected because extension not found in context 'phones'.
```

This happened because registration only tells Asterisk where a phone is located.

It does not automatically tell Asterisk what to do when someone dials a number.

For that, we need a **dialplan**.

---

# 1. What Is a Context?

A **context** is a named section inside the Asterisk dialplan.

You can think of it as:

- A namespace
- A group of dialing rules
- A security boundary
- An entry point into the dialplan

A context begins with a name inside square brackets:

```ini
[phones]
```

Everything written below that heading belongs to the `phones` context until another context begins.

## Context Isolation

Contexts are isolated from each other.

For example, both of these contexts can contain extension `100`:

```ini
[office-a]
exten => 100,1,NoOp(Office A Extension 100)
```

```ini
[office-b]
exten => 100,1,NoOp(Office B Extension 100)
```

Even though both contexts contain extension `100`, they are separate dialplan destinations.

A call entering `office-a` will not automatically use the extension from `office-b`.

## Phone Entry Point

In the legacy `sip.conf` configuration, each phone was assigned:

```ini
context=phones
```

Example:

```ini
[matthias]
type=friend
host=dynamic
secret=Replace-With-A-Strong-Password
context=phones
```

This means that whenever Matthias originates a call, Asterisk begins searching inside:

```ini
[phones]
```

in the dialplan.

If Matthias dials:

```text
100
```

Asterisk searches for extension `100` inside the `phones` context.

## Example Context Organization

A larger PBX may use separate contexts for different responsibilities.

### Internal Phones

```ini
[phones]
```

Used for calls originating from employee extensions.

### Incoming Provider Calls

```ini
[incoming]
```

Used for calls arriving from a SIP trunk or telecom provider.

### Outbound Routing

```ini
[outgoing]
```

Used for external mobile, landline, or international calls.

### Other Possible Contexts

```ini
[management]
[call-center-agents]
[ivr-main]
[voicemail-access]
[emergency-calls]
[international-calls]
```

Separating contexts helps organize the dialplan and restrict what each phone or trunk is allowed to access.

---

# 2. What Is a Dialplan?

The **dialplan** is the call-processing logic of Asterisk.

It determines what happens after someone dials a number or when an incoming call reaches the PBX.

The dialplan can:

- Ring a phone
- Connect two extensions
- Play an audio prompt
- Send a call to voicemail
- Place a caller in a queue
- Route a call through a SIP trunk
- Start call recording
- Open an IVR menu
- Transfer a caller
- End the call

The main dialplan file is:

```text
/etc/asterisk/extensions.conf
```

## How Asterisk Searches the Dialplan

When a phone starts a call, Asterisk performs the following process:

```text
Phone originates call
        ↓
Asterisk checks the phone's context
        ↓
Asterisk searches for the dialed extension
        ↓
Asterisk executes the matching priorities
        ↓
The call ends or moves elsewhere
```

For example:

```text
Phone context: phones
Dialed number: 100
```

Asterisk searches for:

```ini
[phones]
exten => 100,...
```

If no matching extension exists, the call is rejected.

---

# 3. Dialplan Execution Order

Asterisk executes extension logic according to **priorities**.

Example:

```ini
[phones]
exten => 100,1,NoOp(First Line)
exten => 100,2,NoOp(Second Line)
exten => 100,3,Hangup()
```

When extension `100` is dialed, Asterisk executes:

```text
Priority 1 → NoOp(First Line)
Priority 2 → NoOp(Second Line)
Priority 3 → Hangup()
```

The dialplan runs one step at a time in priority order.

---

# 4. Preparing `extensions.conf`

The default `extensions.conf` file may contain extensive sample configuration and documentation.

For a beginner lab, it can be easier to back up the original file and create a clean configuration.

## 1. Back Up the Original File

Run:

```bash
cp /etc/asterisk/extensions.conf /etc/asterisk/extensions.conf.original
```

This preserves the original sample file for future reference.

Verify the backup:

```bash
ls -l /etc/asterisk/extensions.conf*
```

## 2. Create a Fresh File

You can clear the existing file using:

```bash
truncate -s 0 /etc/asterisk/extensions.conf
```

Alternatively, open it in a text editor and remove the existing contents:

```bash
nano /etc/asterisk/extensions.conf
```

> Always create a backup before replacing the sample configuration.

---

# 5. Creating the First Dialplan

Add the following configuration:

```ini
[phones]
exten => 100,1,NoOp(First Line)
exten => 100,2,NoOp(Second Line)
exten => 100,3,Hangup()
```

Save the file.

---

# 6. Understanding the Syntax

Consider this line:

```ini
exten => 100,1,NoOp(First Line)
```

It contains several parts.

| Part | Meaning |
|---|---|
| `exten =>` | Defines an extension rule |
| `100` | The dialed extension |
| `1` | The execution priority |
| `NoOp()` | The Asterisk application |
| `First Line` | Text passed to the application |

## `[phones]`

```ini
[phones]
```

This opens the `phones` context.

Phones configured with:

```ini
context=phones
```

will begin their calls inside this context.

## `exten =>`

```ini
exten =>
```

This keyword defines a dialplan extension.

It tells Asterisk:

```text
When this number is dialed, execute the following application.
```

## Extension Number

```ini
100
```

This is the number that the caller dials.

## Priority

```ini
1
```

The priority controls the order in which applications execute.

The first priority should normally begin with:

```text
1
```

## `NoOp()`

```ini
NoOp(First Line)
```

`NoOp` means:

```text
No Operation
```

It does not perform call routing or media processing.

Its main purpose is to print information in the Asterisk CLI and logs.

It is useful for:

- Testing the dialplan
- Debugging variables
- Confirming execution flow
- Displaying call information
- Marking important stages

## `Hangup()`

```ini
Hangup()
```

This ends the active call cleanly.

Without an explicit call-ending action, Asterisk may wait for additional dialplan logic or a timeout depending on the call state.

Using `Hangup()` makes the intended behavior clear.

---

# 7. Reloading the Dialplan

Asterisk does not automatically reread `extensions.conf` after every edit.

You do not need to restart the complete Asterisk service.

Connect to the CLI:

```bash
asterisk -rvvv
```

Reload only the dialplan:

```asterisk
dialplan reload
```

If the file contains a syntax problem, warnings or errors may appear in the CLI.

## Verify the Context

Display the complete `phones` context:

```asterisk
dialplan show phones
```

You should see extension `100` with its configured priorities.

You can also inspect one extension:

```asterisk
dialplan show 100@phones
```

---

# 8. Testing the Dialplan

Make sure the Asterisk CLI has enough verbosity.

Connect using:

```bash
asterisk -rvvv
```

You can also increase verbosity from inside the CLI:

```asterisk
core set verbose 3
```

Now dial:

```text
100
```

from one of the registered phones.

## Expected Execution Flow

Asterisk should:

1. Receive the call from the phone.
2. Check that the phone belongs to `context=phones`.
3. Search for extension `100` in `[phones]`.
4. Execute priority `1`.
5. Print `First Line`.
6. Execute priority `2`.
7. Print `Second Line`.
8. Execute priority `3`.
9. Hang up the call.

Expected CLI output may look similar to:

```text
Executing [100@phones:1] NoOp("SIP/matthias-00000001", "First Line")
Executing [100@phones:2] NoOp("SIP/matthias-00000001", "Second Line")
Executing [100@phones:3] Hangup("SIP/matthias-00000001", "")
```

The exact channel name and call identifier may differ.

---

# 9. What Has Been Achieved?

Previously, Asterisk returned:

```text
Extension not found in context 'phones'
```

After adding the dialplan:

- Asterisk finds extension `100`.
- The dialplan executes successfully.
- The `NoOp()` messages appear.
- The call ends through `Hangup()`.
- The extension-not-found error disappears.

However, this dialplan still does not ring another phone.

It only proves that the call successfully entered and executed the dialplan.

---

# 10. Cleaner Priority Syntax with `same => n`

Instead of manually numbering every priority, Asterisk supports:

```ini
same => n
```

The letter `n` means:

```text
Next priority
```

The same dialplan can be written as:

```ini
[phones]
exten => 100,1,NoOp(First Line)
same => n,NoOp(Second Line)
same => n,Hangup()
```

This is easier to maintain.

---

# 11. Adding More Debug Information

`NoOp()` can display Asterisk channel variables.

Example:

```ini
[phones]
exten => 100,1,NoOp(Call reached extension ${EXTEN})
same => n,NoOp(Caller ID number is ${CALLERID(num)})
same => n,NoOp(Caller ID name is ${CALLERID(name)})
same => n,Hangup()
```

Useful variables include:

| Variable | Meaning |
|---|---|
| `${EXTEN}` | Dialed extension |
| `${CONTEXT}` | Current dialplan context |
| `${CALLERID(num)}` | Caller ID number |
| `${CALLERID(name)}` | Caller ID name |
| `${CHANNEL}` | Current Asterisk channel |
| `${UNIQUEID}` | Unique call identifier |

---

# 12. Common Errors

## Extension Not Found

Error:

```text
Extension '100' not found in context 'phones'
```

Possible causes:

- The `[phones]` context does not exist.
- Extension `100` is missing.
- The phone is assigned to a different context.
- The dialplan was not reloaded.
- A syntax error prevented the extension from loading.

Check:

```asterisk
dialplan show phones
```

## Invalid Priority

Asterisk may fail to execute the extension if priority `1` is missing.

Incorrect example:

```ini
exten => 100,2,NoOp(Second Line)
```

Correct example:

```ini
exten => 100,1,NoOp(First Line)
exten => 100,2,Hangup()
```

## No CLI Output

If `NoOp()` runs but no useful output appears, increase verbosity:

```asterisk
core set verbose 3
```

or reconnect using:

```bash
asterisk -rvvv
```

## Dialplan Changes Not Applied

Reload the dialplan:

```asterisk
dialplan reload
```

Then verify:

```asterisk
dialplan show 100@phones
```

---

# 13. Minimal Lab Configuration

## `/etc/asterisk/sip.conf`

```ini
[general]
allowguest=no
udpbindaddr=0.0.0.0:5060

[james]
type=friend
host=dynamic
secret=Replace-With-Strong-Password-One
context=phones
disallow=all
allow=ulaw
allow=alaw

[matthias]
type=friend
host=dynamic
secret=Replace-With-Strong-Password-Two
context=phones
disallow=all
allow=ulaw
allow=alaw
```

## `/etc/asterisk/extensions.conf`

```ini
[phones]
exten => 100,1,NoOp(First Line)
same => n,NoOp(Second Line)
same => n,Hangup()
```

Reload both configurations:

```asterisk
sip reload
dialplan reload
```

Verify:

```asterisk
sip show peers
dialplan show phones
```

---

# 14. Final Verification Checklist

- [ ] Both SIP phones are registered.
- [ ] Both phones use `context=phones`.
- [ ] `extensions.conf` has been backed up.
- [ ] The `[phones]` context exists.
- [ ] Extension `100` starts with priority `1`.
- [ ] `NoOp()` lines are configured.
- [ ] `Hangup()` ends the call.
- [ ] `dialplan reload` completes successfully.
- [ ] `dialplan show phones` displays extension `100`.
- [ ] CLI verbosity is set to at least level `3`.
- [ ] Dialing `100` prints both test messages.
- [ ] The previous extension-not-found error no longer appears.

At this stage, the dialplan logic works successfully. The next step is to replace the test-only `NoOp()` application with `Dial()` so one registered phone can ring another.
