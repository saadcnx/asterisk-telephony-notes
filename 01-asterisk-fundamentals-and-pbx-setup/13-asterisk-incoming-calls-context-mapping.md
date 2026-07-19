# Handling Incoming Calls & Context Mapping

## Overview

When a call arrives from an external SIP provider, remote PBX, or another SIP peer, Asterisk places that call into a dialplan context.

A proper incoming-call setup normally requires two things:

1. A SIP peer or trunk that sends external calls into a restricted incoming context.
2. Dialplan rules that map the external number, usually called a DID, to an internal destination.

This structure is the foundation of inbound call routing in Asterisk.

> **Legacy configuration note:** This guide uses `chan_sip` and `sip.conf`. Modern Asterisk systems normally use PJSIP with `pjsip.conf`.

---

# 1. What Is an Incoming Context?

An incoming context is a dedicated dialplan section for calls arriving from outside the internal PBX.

Example:

```ini
[incoming]
```

External calls should not normally enter the same context used by internal phones.

Separating them provides:

- Better security
- Clearer call routing
- Easier troubleshooting
- Controlled access to internal destinations
- Simple DID-to-extension mapping

---

# 2. Creating an External SIP Peer

Open the legacy SIP configuration file:

```bash
nano /etc/asterisk/sip.conf
```

Add an external peer:

```ini
[outside]
type=friend
context=incoming
host=dynamic

secret=Replace-With-A-Strong-Unique-Password

disallow=all
allow=ulaw
allow=alaw

qualify=yes
```

---

# 3. Parameter Explanation

## Peer Name

```ini
[outside]
```

This is the SIP peer name.

It identifies the external connection inside Asterisk.

The remote SIP device or PBX may use `outside` as its registration or authentication username.

## `type=friend`

```ini
type=friend
```

In legacy `chan_sip`, `type=friend` allows both peer-style and user-style matching.

It is commonly used in beginner labs because the connection can both send and receive calls.

For production trunks, `type=peer` is often preferred because matching behavior is more predictable.

## `context=incoming`

```ini
context=incoming
```

This is the most important routing setting.

It means calls originating from this peer enter the `[incoming]` dialplan context.

The peer cannot automatically access internal dialing rules unless the incoming context explicitly sends the call there.

## `host=dynamic`

```ini
host=dynamic
```

This means the remote peer registers with Asterisk and tells the server its current IP address.

This is suitable for a remote softphone, a test PBX, or another device without a fixed IP.

For a telecom provider with a fixed IP address, use:

```ini
host=203.0.113.100
```

## `secret`

```ini
secret=Replace-With-A-Strong-Unique-Password
```

This is the authentication password.

Do not use weak values such as `123456`, `password`, `outside`, or `asterisk`.

## Codec Configuration

```ini
disallow=all
allow=ulaw
allow=alaw
```

This disables all codecs first and then explicitly allows G.711 μ-law and G.711 A-law.

## `qualify=yes`

```ini
qualify=yes
```

This allows Asterisk to monitor the reachability of the external peer using SIP `OPTIONS` requests.

---

# 4. Reloading the SIP Configuration

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Reload the legacy SIP configuration:

```asterisk
sip reload
```

Verify that the peer exists:

```asterisk
sip show peers
```

Inspect it in detail:

```asterisk
sip show peer outside
```

---

# 5. Creating the Incoming Dialplan Context

Open:

```bash
nano /etc/asterisk/extensions.conf
```

Add:

```ini
[incoming]
exten => 991123123,1,Goto(phones,100,1)
```

This rule maps the external number `991123123` to context `phones`, extension `100`, priority `1`.

---

# 6. Understanding the Dialplan Syntax

```ini
exten => 991123123,1,Goto(phones,100,1)
```

| Part | Meaning |
|---|---|
| `exten =>` | Defines a dialplan extension |
| `991123123` | External number or DID being matched |
| `1` | First priority |
| `Goto()` | Transfers execution to another dialplan location |
| `phones` | Target context |
| `100` | Target extension |
| `1` | Target priority |

---

# 7. What Is a DID?

DID stands for:

```text
Direct Inward Dialing
```

A DID is an external telephone number assigned by a telecom or SIP provider.

When someone dials the DID, the provider sends the call to Asterisk.

Asterisk then matches the received destination number inside the incoming context.

---

# 8. What `Goto()` Does

The `Goto()` application changes the current dialplan location.

Syntax:

```asterisk
Goto(context,extension,priority)
```

Example:

```asterisk
Goto(phones,100,1)
```

If `[phones]` contains:

```ini
[phones]
exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()
```

then James's phone will ring.

---

# 9. Complete Example

## `/etc/asterisk/sip.conf`

```ini
[general]
allowguest=no
alwaysauthreject=yes
udpbindaddr=0.0.0.0:5060

[outside]
type=friend
context=incoming
host=dynamic

secret=Replace-With-A-Strong-Unique-Password

disallow=all
allow=ulaw
allow=alaw

qualify=yes
```

## `/etc/asterisk/extensions.conf`

```ini
[incoming]
exten => 991123123,1,NoOp(Incoming call for DID ${EXTEN})
same => n,Goto(phones,100,1)

[phones]
exten => 100,1,NoOp(Routing incoming call to James)
same => n,Dial(SIP/james,20)
same => n,Hangup()
```

---

# 10. Complete Call Flow

```text
Outside caller
      ↓
Dials 991123123
      ↓
SIP provider or external peer
      ↓
Asterisk peer [outside]
      ↓
context=incoming
      ↓
[incoming] matches 991123123
      ↓
Goto(phones,100,1)
      ↓
[phones] extension 100
      ↓
Dial(SIP/james,20)
      ↓
James's phone rings
```

---

# 11. Reloading the Dialplan

After editing `extensions.conf`, run:

```asterisk
dialplan reload
```

Verify the incoming context:

```asterisk
dialplan show incoming
```

Inspect the exact DID:

```asterisk
dialplan show 991123123@incoming
```

Verify the internal target:

```asterisk
dialplan show 100@phones
```

---

# 12. Testing the Incoming Call

1. Register or connect the `outside` peer.
2. Confirm it appears in `sip show peers`.
3. Place a call from the external device to `991123123`.
4. Watch the Asterisk CLI.
5. Confirm that the call enters `[incoming]`.
6. Confirm that `Goto()` sends it to `[phones]`.
7. Confirm that James's phone rings.

Expected CLI output may resemble:

```text
Executing [991123123@incoming:1] NoOp("SIP/outside-00000001", "Incoming call for DID 991123123")
Executing [991123123@incoming:2] Goto("SIP/outside-00000001", "phones,100,1")
Goto (phones,100,1)
Executing [100@phones:1] NoOp("SIP/outside-00000001", "Routing incoming call to James")
Executing [100@phones:2] Dial("SIP/outside-00000001", "SIP/james,20")
```

---

# 13. Mapping Multiple DIDs

```ini
[incoming]
exten => 991123123,1,Goto(phones,100,1)
exten => 991123124,1,Goto(phones,101,1)
exten => 991123125,1,Goto(support-queue,s,1)
```

| External Number | Internal Destination |
|---|---|
| `991123123` | Extension `100` |
| `991123124` | Extension `101` |
| `991123125` | Support queue |

---

# 14. Mapping a DID to an IVR

```ini
[incoming]
exten => 991123123,1,Goto(main-ivr,s,1)
```

Then:

```ini
[main-ivr]
exten => s,1,Answer()
same => n,Playback(company/welcome)
same => n,Background(company/main-menu)
same => n,WaitExten(5)
```

---

# 15. Why Context Isolation Matters

External calls should enter a restricted context.

Incorrect:

```ini
[outside]
context=phones
```

Correct:

```ini
[outside]
context=incoming
```

Then `[incoming]` should contain only explicitly allowed destinations.

Avoid including privileged outbound contexts unless there is a clear requirement and strong security controls.

---

# 16. Security Recommendations

- Set `allowguest=no`.
- Use strong authentication.
- Prefer `type=peer` for fixed provider trunks.
- Restrict provider IP addresses at the firewall.
- Keep the incoming context limited to approved destinations.
- Never allow external callers to access unrestricted outbound dialing.
- Confirm the exact DID format sent by the provider.

Example firewall rule:

```bash
ufw allow from 203.0.113.100 to any port 5060 proto udp
```

---

# 17. Provider-Specific DID Formats

Providers may deliver the destination number in formats such as:

```text
991123123
92991123123
+92991123123
1001
s
```

Do not guess the format. Confirm it from the Asterisk CLI or a SIP trace.

For a provider that sends calls to `s`:

```ini
[incoming]
exten => s,1,Goto(phones,100,1)
```

---

# 18. Pattern Matching for Multiple Numbers

Asterisk patterns begin with an underscore.

```ini
[incoming]
exten => _99112312X,1,NoOp(Incoming DID ${EXTEN})
same => n,Goto(phones,100,1)
```

`X` matches any digit from `0` to `9`.

Use patterns carefully so external callers cannot reach unintended destinations.

---

# 19. Common Problems

## Call Enters the Wrong Context

```asterisk
sip show peer outside
```

Confirm that the context is `incoming`.

## Extension Not Found

```text
Extension '991123123' not found in context 'incoming'
```

Possible causes:

- DID format mismatch
- Wrong context name
- Dialplan not reloaded
- Provider sends the call to `s`
- Provider adds a country code
- Syntax error

Check:

```asterisk
dialplan show incoming
```

## `Goto()` Target Does Not Exist

Verify:

```asterisk
dialplan show 100@phones
```

## Peer Does Not Register

Check:

```asterisk
sip show peers
sip show peer outside
```

## No Audio

Check:

- RTP firewall ports
- NAT settings
- `localnet`
- `externip`
- `directmedia`
- SIP ALG
- Codec compatibility

---

# 20. Useful CLI Commands

```asterisk
sip reload
```

```asterisk
dialplan reload
```

```asterisk
sip show peer outside
```

```asterisk
sip show peers
```

```asterisk
dialplan show incoming
```

```asterisk
dialplan show 991123123@incoming
```

```asterisk
dialplan show 100@phones
```

Enable SIP debugging for the peer:

```asterisk
sip set debug peer outside
```

Disable debugging:

```asterisk
sip set debug off
```

---

# 21. Production-Oriented Example

## `sip.conf`

```ini
[general]
allowguest=no
alwaysauthreject=yes
udpbindaddr=192.168.100.55:5060

[provider]
type=peer
host=203.0.113.100
context=incoming
qualify=yes

disallow=all
allow=ulaw
allow=alaw

insecure=port,invite
```

> Use `insecure=port,invite` only when the provider requires it and after understanding the security implications.

## `extensions.conf`

```ini
[incoming]
exten => 991123123,1,NoOp(Incoming DID ${EXTEN})
same => n,Goto(phones,100,1)

exten => 991123124,1,NoOp(Incoming DID ${EXTEN})
same => n,Goto(phones,101,1)

exten => i,1,NoOp(Invalid incoming destination ${EXTEN})
same => n,Hangup()

[phones]
exten => 100,1,Dial(SIP/james,20)
same => n,Hangup()

exten => 101,1,Dial(SIP/matthias,20)
same => n,Hangup()
```

---

# 22. Final Verification Checklist

- [ ] The external peer or provider trunk exists.
- [ ] External calls use `context=incoming`.
- [ ] Guest SIP calls are disabled.
- [ ] The peer uses a strong password or provider IP authentication.
- [ ] The incoming context exists.
- [ ] The actual DID format has been confirmed.
- [ ] The DID is mapped to the correct internal destination.
- [ ] The `Goto()` target exists.
- [ ] SIP configuration reloads without errors.
- [ ] Dialplan reloads without errors.
- [ ] Incoming calls ring the intended internal phone.
- [ ] External callers cannot access unrestricted outbound contexts.
- [ ] Firewall rules restrict SIP traffic where possible.
- [ ] RTP ports and NAT settings are correctly configured.
- [ ] SIP debugging is disabled after testing.

This incoming-context design provides a clean, secure, and scalable foundation for receiving external calls and mapping DIDs to internal extensions, queues, IVRs, or other PBX services.
