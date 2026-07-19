# Outbound Calls & Context Best Practices

## Overview

In the previous part, we configured incoming calls from an external SIP peer.

Now we will configure the opposite direction:

```text
Internal phone → Asterisk → Outside destination
```

For this lab, the outside world is represented by one SIP softphone.

This simplified setup is enough to understand:

- Outbound call routing
- Context separation
- Dialing permissions
- Security boundaries
- How internal phones gain access to external destinations

> **Legacy configuration note:** This guide uses the legacy `chan_sip` syntax, such as `SIP/outside`. Modern Asterisk deployments normally use PJSIP and syntax such as `PJSIP/outside`.

---

# 1. Why Not Put Everything in One Context?

Technically, internal extensions and outbound dialing rules could all be placed in one context.

Example:

```ini
[phones]
exten => 100,1,Dial(SIP/james)
exten => 101,1,Dial(SIP/matthias)
exten => 8888,1,Dial(SIP/outside)
```

This can work in a small lab, but it becomes difficult to control as the PBX grows.

If everything is placed in one context, every user may gain access to every configured feature.

That can create problems such as:

- Unauthorized international dialing
- Premium-rate number abuse
- Toll fraud
- Accidental access to administrative features
- Poor separation between internal and external routing
- Difficult troubleshooting
- Repeated dialplan logic

Contexts should be treated as both organizational units and security boundaries.

---

# 2. Reasons to Separate Contexts

## Restrictions

Different groups may need different dialing permissions.

For example:

- Reception may call local and mobile numbers.
- Support agents may call only internal extensions.
- Managers may call international destinations.
- Guest phones may call only reception.
- Emergency phones may call only emergency services.

Separate contexts make these restrictions easier to implement.

---

## Security and Control

A context works like a dialplan firewall.

It controls:

```text
Which destinations can this caller reach?
```

A phone can access only:

- Extensions defined in its current context
- Contexts explicitly included
- Locations explicitly reached with `Goto()` or another dialplan application

This makes unauthorized call routing harder.

---

## Organization

A well-structured dialplan is easier to:

- Read
- Maintain
- Audit
- Troubleshoot
- Expand
- Secure

---

# 3. Recommended Minimum Context Structure

A basic Asterisk deployment should usually separate at least three responsibilities.

| Context | Purpose |
|---|---|
| `[incoming]` | Handles calls arriving from external peers or providers |
| `[phones]` | Handles internal users and internal extensions |
| `[outgoing]` | Handles calls leaving the PBX through a trunk or external peer |

A simplified architecture looks like:

```text
External caller
      ↓
[incoming]
      ↓
Internal destination
```

and:

```text
Internal phone
      ↓
[phones]
      ↓
Authorized outbound route
      ↓
[outgoing]
      ↓
SIP trunk or provider
```

---

# 4. Creating the `outgoing` Context

Open the dialplan file:

```bash
nano /etc/asterisk/extensions.conf
```

Add:

```ini
[outgoing]
exten => 8888,1,Dial(SIP/outside)
```

This creates one test destination.

---

# 5. Understanding the Outgoing Rule

Consider:

```ini
exten => 8888,1,Dial(SIP/outside)
```

| Part | Meaning |
|---|---|
| `exten =>` | Defines a dialplan extension |
| `8888` | Number dialed by the internal user |
| `1` | First priority |
| `Dial()` | Places the outbound call |
| `SIP/outside` | Calls the SIP peer named `outside` |

In this lab:

```text
8888
```

represents the only external number.

The peer:

```text
outside
```

represents the external world.

In a production deployment, this would normally be a SIP trunk or telecom provider.

---

# 6. Adding a Timeout and Hangup

A cleaner version is:

```ini
[outgoing]
exten => 8888,1,NoOp(Outbound test call to outside peer)
same => n,Dial(SIP/outside,20)
same => n,Hangup()
```

This:

1. Logs the outbound attempt.
2. Rings the outside peer for up to 20 seconds.
3. Ends the call cleanly after `Dial()` returns.

---

# 7. Why the Internal Phone Cannot Dial `8888` Yet

Suppose an internal phone is configured with:

```ini
context=phones
```

When the user dials:

```text
8888
```

Asterisk searches only inside:

```ini
[phones]
```

If extension `8888` does not exist there, Asterisk rejects the call.

It does not automatically search every context.

This is intentional.

The caller has no access to `[outgoing]` until the dialplan explicitly grants it.

---

# 8. Granting Specific Outbound Access with `Goto()`

Add the following rule to the internal context:

```ini
[phones]
exten => 8888,1,Goto(outgoing,8888,1)
```

Now an internal user dialing `8888` is transferred to:

```text
Context: outgoing
Extension: 8888
Priority: 1
```

The execution flow becomes:

```text
Internal phone dials 8888
      ↓
Asterisk searches [phones]
      ↓
Finds Goto(outgoing,8888,1)
      ↓
Asterisk enters [outgoing]
      ↓
Finds extension 8888
      ↓
Dial(SIP/outside)
      ↓
Outside phone rings
```

---

# 9. Complete Lab Dialplan

```ini
[phones]
exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()

exten => 101,1,Dial(SIP/matthias,20)
same => n,Hangup()

exten => 8888,1,NoOp(Internal user requested an outbound call)
same => n,Goto(outgoing,8888,1)

[outgoing]
exten => 8888,1,NoOp(Calling the outside peer)
same => n,Dial(SIP/outside,20)
same => n,NoOp(Outbound result: ${DIALSTATUS})
same => n,Hangup()
```

---

# 10. Why This Acts Like a Firewall

The `[phones]` context explicitly permits only extension `8888` to enter `[outgoing]`.

If a user dials another undefined number, such as:

```text
7777
```

Asterisk searches `[phones]`, finds no match, and rejects the call.

This creates a default-deny model:

```text
Only explicitly allowed destinations work.
Everything else is blocked.
```

This is a strong security principle for PBX dialplans.

---

# 11. Reloading the Dialplan

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Reload the dialplan:

```asterisk
dialplan reload
```

Verify the internal rule:

```asterisk
dialplan show 8888@phones
```

Verify the outbound rule:

```asterisk
dialplan show 8888@outgoing
```

Display the full contexts:

```asterisk
dialplan show phones
dialplan show outgoing
```

---

# 12. Verifying the External Peer

Before testing, confirm that the outside peer exists and is registered.

Run:

```asterisk
sip show peers
```

Inspect it:

```asterisk
sip show peer outside
```

Expected status may look like:

```text
outside/outside    192.168.100.210    D    OK (12 ms)
```

If the peer is unavailable, `Dial()` may return:

```text
CHANUNAVAIL
```

---

# 13. Testing the Outbound Call

From an internal phone:

```text
Dial 8888
```

Expected behavior:

1. The call starts in `[phones]`.
2. Asterisk matches extension `8888`.
3. `Goto()` sends execution to `[outgoing]`.
4. `[outgoing]` calls `SIP/outside`.
5. The outside softphone rings.
6. The outside user answers.
7. Asterisk bridges both channels.
8. The call continues until one party hangs up.

---

# 14. Expected CLI Output

The CLI may display output similar to:

```text
Executing [8888@phones:1] NoOp("SIP/james-00000001", "Internal user requested an outbound call")
Executing [8888@phones:2] Goto("SIP/james-00000001", "outgoing,8888,1")
Goto (outgoing,8888,1)
Executing [8888@outgoing:1] NoOp("SIP/james-00000001", "Calling the outside peer")
Executing [8888@outgoing:2] Dial("SIP/james-00000001", "SIP/outside,20")
Called SIP/outside
SIP/outside-00000002 is ringing
SIP/outside-00000002 answered SIP/james-00000001
```

Exact channel names and call identifiers will vary.

---

# 15. Caller ID Behavior

The external device may not display the internal caller's name exactly as expected.

Possible reasons include:

- The provider overwrites caller ID.
- The trunk requires an approved outbound number.
- The peer does not trust caller-provided identity.
- SIP headers are rewritten.
- The internal caller ID is not configured.
- The external network supports only numeric caller ID.

An internal phone may be configured with:

```ini
callerid="James Desk" <100>
```

However, a real provider may require:

```text
A verified DID
```

instead of an internal extension number.

For example:

```ini
Set(CALLERID(num)=991123123)
```

The provider's caller ID policy must always be followed.

---

# 16. Why the One-Number Setup Is Not Scalable

The test rule:

```ini
exten => 8888,1,Dial(SIP/outside)
```

works because only one external number exists in the lab.

In the real world, users may dial:

- Local landlines
- Mobile numbers
- National numbers
- International numbers
- Emergency numbers
- Service numbers

It would be impractical to define every possible number manually.

Incorrect large-scale approach:

```ini
exten => 03001234567,1,Dial(...)
exten => 03001234568,1,Dial(...)
exten => 03001234569,1,Dial(...)
```

Asterisk solves this using:

- Dialplan patterns
- Variables
- Number normalization
- Prefix handling
- Trunk routing rules

---

# 17. A Better Access Method: `include`

Instead of creating a separate `Goto()` rule for every allowed outbound number, a context can include another context.

Example:

```ini
[phones]
include => outgoing

exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()

exten => 101,1,Dial(SIP/matthias,20)
same => n,Hangup()
```

Now Asterisk searches:

1. The current `[phones]` context.
2. The included `[outgoing]` context if no local match is found.

This is convenient, but it gives every phone in `[phones]` access to every rule in `[outgoing]`.

Use it only when all phones in the context should have the same outbound permissions.

---

# 18. `Goto()` vs `include`

## `Goto()`

Example:

```ini
exten => 8888,1,Goto(outgoing,8888,1)
```

Benefits:

- Explicit
- Easy to understand
- Grants access to a specific destination
- Useful for controlled lab examples

Limitations:

- Repetitive for many numbers
- Harder to scale

---

## `include`

Example:

```ini
[phones]
include => outgoing
```

Benefits:

- Cleaner
- Scales better
- Reuses a complete routing context

Limitations:

- Grants access to all matching rules in the included context
- Must be designed carefully

---

# 19. Better Production Context Design

Instead of giving every phone the same permissions, create permission-based contexts.

Example:

```ini
[internal-only]
include => internal-extensions
```

```ini
[local-calling]
include => internal-extensions
include => outbound-local
```

```ini
[national-calling]
include => internal-extensions
include => outbound-local
include => outbound-national
```

```ini
[international-calling]
include => internal-extensions
include => outbound-local
include => outbound-national
include => outbound-international
```

Then assign phones according to their role.

Example:

```ini
[100]
context=local-calling
```

```ini
[200]
context=international-calling
```

This creates role-based call permissions.

---

# 20. Example Permission Model

| User Type | Assigned Context | Allowed Calls |
|---|---|---|
| Guest phone | `internal-only` | Internal extensions only |
| Support agent | `local-calling` | Internal and local calls |
| Supervisor | `national-calling` | Internal, local, and national calls |
| Manager | `international-calling` | All approved destinations |

This is much safer than placing all users in one unrestricted context.

---

# 21. Basic Outbound Trunk Example

In a real deployment, the outside destination would usually be reached through a SIP provider.

Example trunk:

```ini
[my-provider]
type=peer
host=sip.provider.example
context=incoming
defaultuser=my-user
secret=Replace-With-Trunk-Password

disallow=all
allow=ulaw
allow=alaw
```

Example outbound rule:

```ini
[outgoing]
exten => 8888,1,Dial(SIP/${EXTEN}@my-provider,30)
same => n,Hangup()
```

Here:

```text
${EXTEN}
```

contains the number the user dialed.

For the lab, `${EXTEN}` would equal:

```text
8888
```

---

# 22. Using `DIALSTATUS`

After `Dial()` returns, Asterisk stores the result in:

```text
${DIALSTATUS}
```

Example:

```ini
[outgoing]
exten => 8888,1,NoOp(Outbound call to ${EXTEN})
same => n,Dial(SIP/outside,20)
same => n,NoOp(Call result: ${DIALSTATUS})
same => n,Hangup()
```

Common values include:

| Status | Meaning |
|---|---|
| `ANSWER` | Destination answered |
| `BUSY` | Destination was busy |
| `NOANSWER` | No answer before timeout |
| `CHANUNAVAIL` | Peer or channel unavailable |
| `CONGESTION` | Call could not be completed |
| `CANCEL` | Caller cancelled before answer |

---

# 23. Security Best Practices

## Use a Default-Deny Model

Only define or include the outbound routes users actually need.

Do not grant international or premium dialing by default.

---

## Separate Incoming and Outgoing Contexts

Never place provider traffic directly into an unrestricted outbound context.

Unsafe:

```ini
[provider]
context=outgoing
```

Safer:

```ini
[provider]
context=incoming
```

The incoming context should route only approved DIDs to approved destinations.

---

## Restrict Expensive Destinations

Create separate contexts for:

- International calls
- Premium-rate numbers
- Satellite numbers
- High-cost regions
- Special service codes

Require explicit authorization before granting access.

---

## Use Provider Limits

Configure:

- Credit limits
- Daily spending limits
- Concurrent-call limits
- Destination restrictions
- Fraud alerts

Dialplan restrictions should not be the only protection.

---

## Set Caller ID Correctly

Use only caller IDs authorized by the provider.

Do not attempt to impersonate numbers that are not assigned to the organization.

---

## Log Outbound Calls

Enable CDR or CEL logging for:

- Caller
- Destination
- Start time
- Answer time
- Duration
- Call result
- Trunk
- Unique ID

This supports troubleshooting, billing, and fraud detection.

---

# 24. Common Problems

## Extension `8888` Not Found

Error:

```text
Extension '8888' not found in context 'phones'
```

Check:

```asterisk
dialplan show 8888@phones
```

Confirm the `Goto()` rule exists and the dialplan was reloaded.

---

## Target Context Not Found

If `Goto(outgoing,8888,1)` fails, verify:

```asterisk
dialplan show 8888@outgoing
```

---

## Outside Peer Does Not Ring

Check:

```asterisk
sip show peers
sip show peer outside
```

Possible causes:

- Peer is not registered.
- Wrong peer name in `Dial()`.
- SIP signaling is blocked.
- Phone is offline.
- Wrong codec.
- NAT issue.
- Dialplan was not reloaded.

---

## Call Connects but Has No Audio

Check:

- RTP firewall ports
- NAT configuration
- `localnet`
- `externip`
- SIP ALG
- `directmedia`
- Codec compatibility

Enable RTP debugging temporarily:

```asterisk
rtp set debug on
```

Disable it after testing:

```asterisk
rtp set debug off
```

---

## Caller ID Is Missing or Replaced

Check:

- Peer caller ID
- `CALLERID(num)`
- Provider policy
- SIP identity headers
- Trunk authentication
- Approved outbound DID list

---

# 25. Useful CLI Commands

## Reload the Dialplan

```asterisk
dialplan reload
```

## Show the Internal Rule

```asterisk
dialplan show 8888@phones
```

## Show the Outgoing Rule

```asterisk
dialplan show 8888@outgoing
```

## Show the Complete Outgoing Context

```asterisk
dialplan show outgoing
```

## Show SIP Peers

```asterisk
sip show peers
```

## Inspect the Outside Peer

```asterisk
sip show peer outside
```

## Show Active Calls

```asterisk
core show channels
```

## Show Detailed Channel Information

```asterisk
core show channel <channel-name>
```

---

# 26. Complete Lab Example

## `/etc/asterisk/sip.conf`

```ini
[general]
allowguest=no
alwaysauthreject=yes
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

[outside]
type=friend
host=dynamic
secret=Replace-With-Strong-Password-Two
context=incoming
disallow=all
allow=ulaw
allow=alaw
qualify=yes
```

## `/etc/asterisk/extensions.conf`

```ini
[incoming]
exten => 991123123,1,Goto(phones,100,1)

[phones]
exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()

exten => 8888,1,NoOp(Authorized outbound test call)
same => n,Goto(outgoing,8888,1)

[outgoing]
exten => 8888,1,NoOp(Calling outside peer)
same => n,Dial(SIP/outside,20)
same => n,NoOp(Outbound result: ${DIALSTATUS})
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
dialplan show phones
dialplan show outgoing
```

---

# 27. Final Verification Checklist

- [ ] The PBX has separate incoming, internal, and outgoing contexts.
- [ ] Internal phones start in the `[phones]` context.
- [ ] The `[outgoing]` context contains the external dialing rule.
- [ ] Extension `8888` exists in `[outgoing]`.
- [ ] The `[phones]` context explicitly grants access to `8888`.
- [ ] The dialplan reloads without errors.
- [ ] The outside peer is registered and reachable.
- [ ] Dialing `8888` from an internal phone reaches `[outgoing]`.
- [ ] The outside softphone rings.
- [ ] Both parties can hear each other.
- [ ] Undefined outbound numbers remain blocked.
- [ ] Caller ID behavior has been verified.
- [ ] Expensive destinations are not enabled by default.
- [ ] Provider limits and logging are planned for production.
- [ ] The next design step is pattern-based outbound routing.

This setup demonstrates the basic outbound call flow and shows how Asterisk contexts act as both routing containers and security boundaries. The next step is to use dialplan patterns and variables so one rule can safely match groups of external telephone numbers.
