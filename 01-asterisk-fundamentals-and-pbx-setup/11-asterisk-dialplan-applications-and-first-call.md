# Dialplan Part 2: Applications & Making the First Call

## Recap of the Previous Session

In the previous lesson, we used `NoOp()` and `Hangup()` to observe how Asterisk executes a dialplan line by line. The dialplan worked, but it did not ring an actual phone.

In this part, we use the `Dial()` application to make the first real call between two registered SIP peers.

> **Legacy note:** This guide uses the old `chan_sip` syntax, such as `SIP/james`. Modern Asterisk systems generally use PJSIP and syntax such as `PJSIP/james`.

---

# 1. What Is an Asterisk Application?

An application is a command that performs an action inside the dialplan.

Examples include:

```asterisk
NoOp()
Hangup()
Dial()
Answer()
Playback()
VoiceMail()
Queue()
ConfBridge()
```

A dialplan extension executes one or more applications in priority order.

Example:

```ini
exten => 100,1,NoOp(Call started)
same => n,Dial(SIP/james)
same => n,Hangup()
```

---

# 2. Finding Available Applications

The applications available on a server depend on:

- The installed Asterisk version
- Which modules were compiled
- Which modules are installed and loaded
- Which optional dependencies are available

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

## 2.1 List All Applications

```asterisk
core show applications
```

This displays every dialplan application available in the running Asterisk instance.

## 2.2 Search Applications by Name

```asterisk
core show applications like dial
```

This may return applications such as:

```text
Dial
RetryDial
```

## 2.3 Search Application Descriptions

```asterisk
core show applications describing dial
```

This finds applications whose documentation contains the word `dial`.

## 2.4 Display Detailed Help

```asterisk
core show application Dial
```

This documentation is generated for the Asterisk version and modules running on the server. It includes supported syntax, parameters, options, return values, and related variables.

---

# 3. Understanding `Dial()` Syntax

A simplified form is:

```asterisk
Dial(Technology/Resource,timeout,options)
```

Example:

```asterisk
Dial(SIP/james,20)
```

| Component | Example | Meaning |
|---|---|---|
| Technology | `SIP` | Channel driver or communication technology |
| Resource | `james` | Peer, endpoint, trunk, or destination |
| Timeout | `20` | Maximum ringing time in seconds |
| Options | Optional | Additional call behavior |

The timeout and options can be omitted:

```asterisk
Dial(SIP/james)
```

---

# 4. Technology and Resource

## Technology

Because this lab uses `chan_sip`, the channel technology is:

```text
SIP
```

Other examples include:

```text
PJSIP
Local
DAHDI
IAX2
```

Example channel names:

```asterisk
Dial(PJSIP/1001)
Dial(IAX2/remote-office)
Dial(Local/200@internal)
```

## Resource

The resource identifies the configured destination.

If the peer is defined in `sip.conf` as:

```ini
[james]
```

then the resource is:

```text
james
```

The complete channel name is:

```text
SIP/james
```

---

# 5. Creating the First Real Call

Open the dialplan file:

```bash
nano /etc/asterisk/extensions.conf
```

Add or update the `phones` context:

```ini
[phones]
exten => 100,1,NoOp(First Line)
exten => 100,2,NoOp(Second Line)
exten => 100,3,Dial(SIP/james)
exten => 100,4,Hangup()
```

When a phone in the `phones` context dials `100`, Asterisk executes each priority in order.

---

# 6. Dialplan Explanation

## Priority 1

```ini
exten => 100,1,NoOp(First Line)
```

Asterisk prints `First Line` and immediately continues.

## Priority 2

```ini
exten => 100,2,NoOp(Second Line)
```

Asterisk prints `Second Line` and continues.

## Priority 3

```ini
exten => 100,3,Dial(SIP/james)
```

Asterisk creates a SIP channel and attempts to ring the peer named `james`.

The call remains inside `Dial()` while the destination is ringing or while the two parties are connected.

## Priority 4

```ini
exten => 100,4,Hangup()
```

This ends the call if `Dial()` returns and execution reaches this priority.

A caller hangup may terminate the channel before the next priority executes, so `Hangup()` is not guaranteed to run after every completed call.

---

# 7. Cleaner Priority Syntax

Instead of manually numbering every line, use `same => n`:

```ini
[phones]
exten => 100,1,NoOp(First Line)
same => n,NoOp(Second Line)
same => n,Dial(SIP/james)
same => n,Hangup()
```

The letter `n` means **next priority**.

---

# 8. Add a Ringing Timeout

It is better to define how long the phone should ring:

```ini
[phones]
exten => 100,1,NoOp(Calling James)
same => n,Dial(SIP/james,20)
same => n,Hangup()
```

Asterisk rings James for up to 20 seconds. If nobody answers, `Dial()` returns and the dialplan continues.

---

# 9. Reloading the Dialplan

From the Asterisk CLI, run:

```asterisk
dialplan reload
```

A full Asterisk restart is not required.

Verify the extension:

```asterisk
dialplan show 100@phones
```

You can also display the complete context:

```asterisk
dialplan show phones
```

---

# 10. Verify the SIP Peer

Before calling, confirm that James is registered:

```asterisk
sip show peers
```

Example:

```text
Name/username    Host               Dyn  Status
james/james      192.168.100.200     D   OK (18 ms)
```

Inspect the peer in detail:

```asterisk
sip show peer james
```

If the peer is not registered, `Dial()` may return a channel-unavailable result.

---

# 11. Executing the First Call

From the registered softphone, dial:

```text
100
```

Asterisk should:

1. Receive the call in the `phones` context.
2. Find extension `100`.
3. Execute both `NoOp()` applications.
4. Create an outbound channel to `SIP/james`.
5. Ring James's phone.
6. Bridge the channels when James answers.
7. Keep the call active until one side hangs up.
8. End or continue the dialplan according to the result.

---

# 12. Expected CLI Output

With sufficient verbosity, output may look similar to:

```text
Executing [100@phones:1] NoOp("SIP/matthias-00000001", "First Line")
Executing [100@phones:2] NoOp("SIP/matthias-00000001", "Second Line")
Executing [100@phones:3] Dial("SIP/matthias-00000001", "SIP/james")
Called SIP/james
SIP/james-00000002 is ringing
SIP/james-00000002 answered SIP/matthias-00000001
```

When a party hangs up, you may see:

```text
Spawn extension (phones, 100, 3) exited non-zero
```

The exact channel names and messages vary by Asterisk version.

---

# 13. How `Dial()` Behaves

`Dial()` is different from a quick application such as `NoOp()`.

While inside `Dial()`, Asterisk may be:

- Creating an outbound channel
- Sending SIP signaling
- Waiting for ringing
- Waiting for an answer
- Bridging both callers
- Keeping the call connected
- Waiting for a timeout or hangup

The dialplan does not immediately continue while this work is active.

---

# 14. Blocking vs. Fast Applications

A useful beginner model is to distinguish applications that remain active for some time from applications that complete quickly.

The exact behavior still depends on the application, options, timeout, and channel state.

## Blocking or Long-Running Applications

### `Dial()`

```asterisk
Dial(SIP/james,20)
```

It may remain active while the phone rings or the call remains connected.

### `VoiceMail()`

```asterisk
VoiceMail(1001@default)
```

It may remain active while prompts play and the caller records a message.

### `Queue()`

```asterisk
Queue(support)
```

It may keep the caller waiting until an agent answers, the caller exits, or a timeout occurs.

### `Playback()`

```asterisk
Playback(welcome)
```

It remains active until the selected audio prompt finishes or is interrupted.

## Fast or Pass-Through Applications

### `NoOp()`

```asterisk
NoOp(Test message)
```

It prints information and immediately returns.

### `Set()`

```asterisk
Set(DEPARTMENT=support)
```

It assigns a channel variable and continues.

### `Goto()`

```asterisk
Goto(support,s,1)
```

It immediately changes the current dialplan location.

### `Hangup()`

```asterisk
Hangup()
```

It terminates the channel immediately. Normal execution does not continue after it.

---

# 15. Important `Dial()` Results

After `Dial()` finishes, Asterisk stores the result in:

```text
${DIALSTATUS}
```

Common values include:

| Value | Meaning |
|---|---|
| `ANSWER` | The destination answered |
| `BUSY` | The destination was busy |
| `NOANSWER` | The destination did not answer |
| `CHANUNAVAIL` | The destination or channel was unavailable |
| `CONGESTION` | The call could not be completed due to congestion |
| `CANCEL` | The caller cancelled before answer |

Display the result:

```ini
[phones]
exten => 100,1,NoOp(Calling James)
same => n,Dial(SIP/james,20)
same => n,NoOp(Dial status is ${DIALSTATUS})
same => n,Hangup()
```

---

# 16. Basic Failure Handling

```ini
[phones]
exten => 100,1,NoOp(Calling James)
same => n,Dial(SIP/james,20)
same => n,NoOp(Dial completed with status ${DIALSTATUS})
same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?busy)
same => n,GotoIf($["${DIALSTATUS}" = "NOANSWER"]?noanswer)
same => n,Hangup()

same => n(busy),Playback(the-line-is-busy)
same => n,Hangup()

same => n(noanswer),Playback(vm-nobodyavail)
same => n,Hangup()
```

This demonstrates how the dialplan can react differently to busy and unanswered calls.

---

# 17. CDR Warnings

After a call, Asterisk may display a warning related to CDR.

CDR stands for:

```text
Call Detail Record
```

A CDR can store:

- Caller number
- Destination number
- Start time
- Answer time
- End time
- Call duration
- Billable seconds
- Disposition
- Unique call ID

A missing CDR backend does not necessarily prevent a basic lab call from working.

CDR storage can later be configured using CSV, ODBC, MySQL or MariaDB, PostgreSQL, or another supported backend.

---

# 18. Common Problems

## Peer Not Found

Check:

```asterisk
sip show peers
sip show peer james
```

Confirm that `Dial(SIP/james)` exactly matches the section name in `sip.conf`.

## Phone Does Not Ring

Possible causes include:

- The peer is not registered.
- The phone is offline.
- The peer name is incorrect.
- The dialplan was not reloaded.
- The phone is in Do Not Disturb mode.
- SIP traffic is blocked.
- NAT configuration is incorrect.

Check:

```asterisk
sip show peers
dialplan show 100@phones
```

## No Audio or One-Way Audio

Check:

- RTP port range
- Firewall rules
- NAT settings
- `externip`
- `localnet`
- `directmedia`
- SIP ALG
- Codec compatibility

Enable RTP debugging temporarily:

```asterisk
rtp set debug on
```

Disable it after testing:

```asterisk
rtp set debug off
```

## Dialplan Does Not Reach `Hangup()`

This can be normal when a party hangs up while the channel is inside `Dial()`.

Use the special `h` extension for hangup logic:

```ini
[phones]
exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()

exten => h,1,NoOp(Call has ended)
```

---

# 19. Minimal Complete Configuration

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
qualify=yes

[matthias]
type=friend
host=dynamic
secret=Replace-With-Strong-Password-Two
context=phones
disallow=all
allow=ulaw
allow=alaw
qualify=yes
```

## `/etc/asterisk/extensions.conf`

```ini
[phones]
exten => 100,1,NoOp(Calling James)
same => n,Dial(SIP/james,20)
same => n,NoOp(Dial status: ${DIALSTATUS})
same => n,Hangup()
```

Reload:

```asterisk
sip reload
dialplan reload
```

Verify:

```asterisk
sip show peers
dialplan show 100@phones
```

---

# 20. Final Verification Checklist

- [ ] Both SIP peers are configured.
- [ ] Both phones are registered.
- [ ] The `phones` context exists.
- [ ] Extension `100` exists.
- [ ] `Dial(SIP/james)` uses the correct peer name.
- [ ] The dialplan reloads without errors.
- [ ] `dialplan show 100@phones` displays the expected priorities.
- [ ] `sip show peers` shows James as reachable.
- [ ] Dialing `100` rings James's phone.
- [ ] James can answer the call.
- [ ] Both parties can hear each other.
- [ ] The call remains active until one party hangs up.
- [ ] `${DIALSTATUS}` can be inspected after unsuccessful calls.
- [ ] Any CDR warning is understood as a separate logging issue.

After completing this lesson, the first real voice call between two registered Asterisk peers should work successfully.
