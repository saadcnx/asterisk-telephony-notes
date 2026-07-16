# Adding SIP Peers & Softphones to Asterisk

> **Legacy configuration note:** This guide uses `chan_sip`, `sip.conf`, and commands such as `sip show peers`. Modern Asterisk installations usually use PJSIP with `pjsip.conf`. Follow this guide when working with an older Asterisk version or a lab specifically based on `chan_sip`.

## Why Use a Softphone?

A softphone is an application that turns a computer or mobile device into a SIP telephone.

### Benefits

- **Cost:** Softphones are generally free or inexpensive.
- **Testing:** They are useful for testing extensions, registration, codecs, and call flows.
- **Scalability in a lab:** You can create several test accounts without purchasing multiple physical phones.
- **Convenience:** Multiple softphones can be used to simulate callers, agents, transfers, and other call scenarios.

### Example Softphones

- **X-Lite:** Traditionally used as a cross-platform SIP softphone for testing.
- **Telephone for macOS:** A simple SIP client that is useful for creating multiple test accounts.

> Software availability and licensing can change over time. Any standards-compatible SIP softphone can be used for this lab.

---

# Initial State

Connect to the Asterisk CLI:

```bash
asterisk -rvvv
```

Display the configured SIP peers:

```asterisk
sip show peers
```

At the beginning, the list may be empty because no SIP peers have been configured.

---

# Step 1: Cleaning `sip.conf`

Legacy SIP peer configuration is stored in:

```text
/etc/asterisk/sip.conf
```

The sample file can be very large because it contains extensive documentation as comments.

## 1. Create a Backup

Before editing the file, create a backup:

```bash
cp /etc/asterisk/sip.conf /etc/asterisk/sip.conf.orig
```

You can later read the original documentation from:

```text
/etc/asterisk/sip.conf.orig
```

## 2. Open the File in Vim

```bash
vim /etc/asterisk/sip.conf
```

## 3. Remove Full-Line Comments

Asterisk configuration comments commonly begin with a semicolon:

```text
;
```

In Vim command mode, remove all lines whose first non-whitespace character is a semicolon:

```vim
:g/^\s*;/d
```

## 4. Remove Empty Lines

Remove blank or whitespace-only lines:

```vim
:g/^\s*$/d
```

Save and exit:

```vim
:wq
```

The file should now contain mostly active configuration instead of hundreds of documentation lines.

> Be careful when deleting comments automatically. Review the remaining file before reloading Asterisk, because sample configurations vary between versions.

---

# Step 2: Understanding `sip.conf` Sections

Asterisk configuration files are divided into named sections.

A section begins with a name inside square brackets:

```ini
[general]
```

The settings below it belong to that section until the next section begins.

## The `[general]` Section

The `[general]` section contains global SIP settings that apply to the entire `chan_sip` service.

Example:

```ini
[general]
bindaddr=0.0.0.0
udpbindaddr=0.0.0.0:5060
qualify=yes
```

### `bindaddr`

```ini
bindaddr=0.0.0.0
```

This defines the local IP address on which the SIP service listens.

Using `0.0.0.0` means Asterisk listens on all available IPv4 interfaces.

### `udpbindaddr`

```ini
udpbindaddr=0.0.0.0:5060
```

This defines the IP address and UDP port used for SIP signaling. The standard SIP UDP port is `5060`.

> SIP signaling and RTP audio are different. SIP commonly uses UDP port `5060`, while RTP uses a separate dynamically selected UDP port range.

### UDP and TCP

Legacy SIP deployments commonly use UDP. Asterisk can also support SIP over TCP or TLS when configured, but endpoint support must be verified.

### `qualify=yes`

```ini
qualify=yes
```

This tells Asterisk to monitor the reachability of SIP peers. Asterisk periodically sends SIP `OPTIONS` requests and measures the response time.

A reachable peer may appear with a status such as:

```text
OK (25 ms)
```

> The displayed time is a SIP reachability measurement, not exactly the same as an ICMP `ping`.

## Templates and Macros

Some sample configurations contain reusable templates, macros, or sections referenced by other peers. If a section is not used or included anywhere, it may have no effect.

Do not delete unfamiliar sections unless you understand their purpose and have confirmed they are unused.

---

# Step 3: Creating SIP Peer Definitions

At least two endpoints are useful for testing calls.

Add the following peer definitions to the bottom of `/etc/asterisk/sip.conf`.

## Peer 1: James

```ini
[james]
type=friend
context=phones
host=dynamic
secret=Use-A-Strong-Unique-Password-Here
disallow=all
allow=ulaw
allow=alaw
qualify=yes
```

## Peer 2: Matthias

```ini
[matthias]
type=friend
context=phones
host=dynamic
secret=Use-Another-Strong-Unique-Password
disallow=all
allow=ulaw
allow=alaw
qualify=yes
```

# Parameter Explanations

## Peer Name

```ini
[james]
```

The value inside the brackets is the SIP peer name. It is commonly used as the phone's SIP username, authentication username, or account name.

Peer names may be case-sensitive depending on the client and configuration, so use the exact same spelling on the phone.

## `type=friend`

```ini
type=friend
```

In `chan_sip`, `friend` combines the behavior of both `user` and `peer`. It is commonly used for phones that must both place and receive calls.

> In production legacy configurations, using `type=peer` may provide more predictable matching and better security in some designs. `type=friend` is kept here because it is common in introductory labs.

## `context=phones`

```ini
context=phones
```

This determines which dialplan context handles calls coming from the phone.

In this example, calls from the peer enter `[phones]` inside `/etc/asterisk/extensions.conf`.

If that context does not exist or does not contain the dialed extension, the call will fail.

## `host=dynamic`

```ini
host=dynamic
```

This means the phone does not have a permanently configured IP address in `sip.conf`. Instead, the phone registers with Asterisk and reports its current contact address.

This is suitable for DHCP-based phones, softphones, mobile SIP clients, and devices whose IP addresses may change.

For a trusted remote server or provider with a fixed IP, a configuration may instead use:

```ini
host=203.0.113.10
```

## `secret`

```ini
secret=Use-A-Strong-Unique-Password-Here
```

This is the shared password used by the SIP device to authenticate.

For a secure deployment:

- Use a long, unique password.
- Do not reuse passwords between extensions.
- Avoid names, extension numbers, and dictionary words.
- Restrict SIP access using firewall rules.
- Use TLS and SRTP where supported and required.
- Never commit real SIP credentials to a public Git repository.

Although the password appears as plain text in `sip.conf`, SIP digest authentication does not normally send the raw password directly over the network. However, storing secrets in plaintext configuration files still requires strict file permissions.

Recommended permissions:

```bash
chown root:asterisk /etc/asterisk/sip.conf
chmod 640 /etc/asterisk/sip.conf
```

## `disallow=all`

```ini
disallow=all
```

This first disables all codecs. It makes the allowed codec list explicit and avoids accidentally enabling unwanted codecs.

## `allow=ulaw`

```ini
allow=ulaw
```

This enables the G.711 μ-law codec. It provides high-quality audio but uses more bandwidth than compressed codecs.

## `allow=alaw`

```ini
allow=alaw
```

This enables the G.711 A-law codec. It provides similar quality and bandwidth usage to μ-law.

## `qualify=yes`

```ini
qualify=yes
```

This enables reachability monitoring for the individual peer.

---

# Step 4: Validate and Apply the Configuration

Asterisk does not always apply file changes immediately.

## 1. Reload the SIP Configuration

From the Asterisk CLI, run:

```asterisk
sip reload
```

Asterisk will reload `sip.conf`. If there is a syntax or configuration problem, warnings may appear in the CLI.

## 2. Display the Peers

Run:

```asterisk
sip show peers
```

Before registration, you may see entries similar to:

```text
Name/username    Host           Dyn  Status
james            (Unspecified)  D   UNKNOWN
matthias         (Unspecified)  D   UNKNOWN
```

The important points are:

- The peers exist.
- `Dyn` indicates dynamic registration.
- No device IP appears until the phone registers.
- The status may vary depending on `qualify` and the Asterisk version.

## 3. Inspect a Specific Peer

```asterisk
sip show peer james
```

or:

```asterisk
sip show peer matthias
```

This displays details such as the current address, context, codecs, qualify status, SIP port, and authentication settings.

---

# Step 5: Registering the Phones

## Hardware SIP Phone Example

Open the phone's account or identity configuration page.

| Phone Setting | Value |
|---|---|
| Account or username | `james` |
| Authentication username | `james` |
| Password | The value configured in `secret` |
| Registrar or SIP server | Asterisk server IP address |
| SIP port | `5060` unless changed |
| Transport | UDP for this lab |

Example:

```text
Account: james
Authentication username: james
Registrar: 192.168.100.55
Port: 5060
Transport: UDP
```

Apply the configuration and allow the phone to register.

### Verify the Hardware Phone

Run:

```asterisk
sip show peers
```

A registered and reachable phone may appear similar to:

```text
james/james    192.168.100.200    D    OK (18 ms)
```

Because `host=dynamic` is configured, Asterisk learns the phone's IP address during registration.

If the phone is unplugged, it may eventually appear as `UNREACHABLE`.

## Softphone Example

Create a SIP account in the softphone using:

| Softphone Setting | Example |
|---|---|
| Display name | `Matthias` |
| Domain or SIP server | `192.168.100.55` |
| Username | `matthias` |
| Authentication username | `matthias` |
| Password | The configured `secret` |
| Port | `5060` |
| Transport | UDP |

Save the account and wait for the registration status to become active.

### Verify the Softphone

In the Asterisk CLI:

```asterisk
sip show peers
```

Both peers should now show their contact IP addresses and reachability status.

Example:

```text
Name/username      Host               Dyn  Status
james/james        192.168.100.200     D   OK (18 ms)
matthias/matthias  192.168.100.201     D   OK (7 ms)
```

---

# Useful SIP Debugging Commands

## Show All SIP Peers

```asterisk
sip show peers
```

## Show One Peer

```asterisk
sip show peer james
```

## Show SIP Settings

```asterisk
sip show settings
```

## Show Registered SIP Trunks

```asterisk
sip show registry
```

This is mainly useful when Asterisk itself registers with an external SIP provider.

## Enable SIP Message Debugging

For a specific peer:

```asterisk
sip set debug peer james
```

Enable all SIP debugging:

```asterisk
sip set debug on
```

Disable debugging after testing:

```asterisk
sip set debug off
```

> SIP debugging can produce a large amount of output and may expose usernames, IP addresses, and authentication-related information. Use it carefully.

---

# Step 6: The Next Problem — Dialplan Configuration

Even when both phones are registered successfully, they may not yet be able to call each other.

For example, Asterisk may report:

```text
Call from 'matthias' to extension '100' rejected because extension not found in context 'phones'
```

## Why This Happens

Both peers use:

```ini
context=phones
```

This means calls from these phones enter the `[phones]` dialplan context.

Asterisk then looks in `/etc/asterisk/extensions.conf` for a matching extension inside `[phones]`.

If no matching rule exists, Asterisk does not know what action to perform.

Registration only tells Asterisk where a phone is located. It does not automatically create dialing rules.

# Example of the Required Dialplan

A future dialplan configuration could include:

```ini
[phones]
exten => 101,1,Dial(SIP/james,20)
same  => n,Hangup()

exten => 102,1,Dial(SIP/matthias,20)
same  => n,Hangup()
```

With this dialplan:

- Dialing `101` rings the `james` SIP peer.
- Dialing `102` rings the `matthias` SIP peer.

After editing `extensions.conf`, reload the dialplan:

```asterisk
dialplan reload
```

Verify it:

```asterisk
dialplan show phones
```

> This example uses the legacy `SIP/peer-name` channel syntax. PJSIP configurations use `PJSIP/endpoint-name`.

---

# Complete Example `sip.conf`

```ini
[general]
bindaddr=0.0.0.0
udpbindaddr=0.0.0.0:5060
qualify=yes
allowguest=no

[james]
type=friend
context=phones
host=dynamic
secret=Use-A-Strong-Unique-Password-Here
disallow=all
allow=ulaw
allow=alaw
qualify=yes

[matthias]
type=friend
context=phones
host=dynamic
secret=Use-Another-Strong-Unique-Password
disallow=all
allow=ulaw
allow=alaw
qualify=yes
```

---

# Security Checklist

Before exposing SIP services beyond a private lab network:

- [ ] Disable anonymous or guest SIP calls.
- [ ] Use long and unique extension passwords.
- [ ] Restrict UDP port `5060` with firewall rules.
- [ ] Do not expose the Asterisk management interface publicly.
- [ ] Enable Fail2Ban or another intrusion-prevention system.
- [ ] Use a VPN for remote phones when practical.
- [ ] Consider SIP TLS and SRTP for encrypted signaling and media.
- [ ] Keep Asterisk and the operating system updated.
- [ ] Limit dialplan permissions for each context.
- [ ] Protect configuration files with appropriate ownership and permissions.
- [ ] Never publish real SIP passwords in GitHub repositories.

---

# Final Verification Checklist

- [ ] `sip.conf` has been backed up.
- [ ] The `[general]` section is correctly configured.
- [ ] `james` and `matthias` are defined as SIP peers.
- [ ] Each peer has a strong, unique password.
- [ ] The peers use `context=phones`.
- [ ] Only required codecs are allowed.
- [ ] `sip reload` completes without configuration errors.
- [ ] `sip show peers` lists both peers.
- [ ] The hardware phone registers successfully.
- [ ] The softphone registers successfully.
- [ ] Both peers show a reachable status.
- [ ] A `[phones]` dialplan context is created before call testing.

At this stage, the SIP devices are registered with Asterisk. The next step is to build the dialplan that allows the extensions to call each other.


# Screenshot 

