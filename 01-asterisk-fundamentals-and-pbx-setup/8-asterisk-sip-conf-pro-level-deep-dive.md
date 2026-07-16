# `sip.conf` Pro-Level Deep Dive

> **Legacy configuration note:** This guide covers the legacy `chan_sip` channel driver and its `sip.conf` file. Modern Asterisk deployments generally use PJSIP with `pjsip.conf`. Use this material when working with an older Asterisk version, legacy PBX, or a lab specifically based on `chan_sip`.

---

# 1. The Structure of `sip.conf`

The `sip.conf` file is mainly divided into two types of sections:

## A. The `[general]` Section

The `[general]` section contains global SIP settings.

These settings can control:

- Default dialplan context
- SIP listening address and port
- UDP and TCP support
- Guest-call behavior
- NAT handling
- Local and external network information
- Codec preferences
- SIP registration strings
- Authentication behavior

Example:

```ini
[general]
context=public
allowguest=no
udpbindaddr=0.0.0.0:5060
disallow=all
allow=ulaw
allow=alaw
```

## B. Device and Trunk Sections

Each additional section defines a SIP device, account, or trunk.

Examples:

```ini
[1001]
```

```ini
[reception]
```

```ini
[my_sip_provider]
```

These sections can contain settings such as authentication credentials, dialplan context, host address, codec permissions, NAT behavior, caller ID, DTMF mode, and reachability monitoring.

---

# 2. Deep Dive into `[general]`

Example:

```ini
[general]
context=public
allowguest=no
allowoverlap=no

udpbindaddr=0.0.0.0:5060
tcpenable=yes
tcpbindaddr=0.0.0.0:5060

localnet=192.168.1.0/255.255.255.0
externip=203.0.113.50

nat=force_rport,comedia

disallow=all
allow=ulaw
allow=alaw

alwaysauthreject=yes
```

## `context`

```ini
context=public
```

This defines the default dialplan context for calls that do not match a more specific peer configuration.

For security, use a highly restricted context such as:

```ini
context=public
```

```ini
context=unauthenticated
```

```ini
context=dummy
```

The selected context should not allow unrestricted outbound or international dialing.

> Never place unknown callers directly into a privileged internal or outbound context.

## `allowguest`

```ini
allowguest=no
```

This controls whether Asterisk accepts unauthenticated SIP calls that do not match a configured peer.

For most production systems, use:

```ini
allowguest=no
```

This helps prevent unsolicited SIP calls and unauthorized use of the PBX.

It should be combined with firewall restrictions, strong passwords, Fail2Ban, limited dialplan contexts, provider IP allowlists, VPN access, and TLS/SRTP where appropriate.

## `allowoverlap`

```ini
allowoverlap=no
```

Overlap dialing allows digits to be sent gradually instead of as one complete number.

Modern SIP environments usually do not require it, so `no` is common.

## `udpbindaddr`

```ini
udpbindaddr=0.0.0.0:5060
```

This defines the IPv4 address and UDP port on which Asterisk listens for SIP signaling.

`0.0.0.0` means listen on all IPv4 interfaces.

For a more restricted setup:

```ini
udpbindaddr=192.168.1.10:5060
```

> Binding to all interfaces is convenient, but firewall rules must still restrict who can reach the SIP service.

## `tcpenable`

```ini
tcpenable=yes
tcpbindaddr=0.0.0.0:5060
```

This enables SIP over TCP. SIP can use UDP, TCP, or TLS. The device and server must support and use the same transport.

---

# 3. Network and NAT Configuration

Incorrect NAT handling can cause:

- One-way audio
- No audio
- Failed registrations
- Calls dropping after several seconds
- Private IP addresses appearing in SIP or SDP
- Phones becoming unreachable

## `localnet`

```ini
localnet=192.168.1.0/255.255.255.0
```

This tells Asterisk which networks are local.

You can define more than one:

```ini
localnet=192.168.1.0/255.255.255.0
localnet=10.10.0.0/255.255.0.0
```

## `externip`

```ini
externip=203.0.113.50
```

This defines the public IP of an Asterisk server behind NAT.

For a dynamic public IP, older `chan_sip` deployments may use:

```ini
externhost=pbx.example.com
externrefresh=300
```

> `203.0.113.50` is an example documentation address. Replace it with the actual public IP or hostname.

## `nat=force_rport,comedia`

```ini
nat=force_rport,comedia
```

### `force_rport`

Asterisk replies to the source IP and port from which the SIP request actually arrived.

### `comedia`

Asterisk learns the actual RTP source address before sending audio back.

> NAT settings can help, but routers, firewalls, SIP ALG, blocked RTP ports, and incorrect SDP addresses must also be checked.

---

# 4. Codec Configuration

Start with:

```ini
disallow=all
```

Then enable only required codecs:

```ini
allow=ulaw
allow=alaw
```

## `ulaw`

G.711 μ-law:

- High voice quality
- Low compression
- About 64 kbit/s payload bandwidth
- Common in North America and Japan

## `alaw`

G.711 A-law:

- Similar quality and bandwidth to μ-law
- Common in Europe and many international telecom networks

## `g729`

```ini
allow=g729
```

G.729 uses less bandwidth, but:

- Availability depends on the Asterisk build
- Some implementations may require licensing
- Transcoding can consume CPU
- Both sides must support it
- Voice quality may be lower than G.711

Do not enable it unless installed, legally available, and needed.

---

# 5. Understanding `type=peer`, `type=user`, and `type=friend`

| Type | Main Behavior |
|---|---|
| `type=peer` | Primarily matches by source IP or host; commonly used for trunks and devices |
| `type=user` | Primarily matches by the SIP `From` username |
| `type=friend` | Creates both user-style and peer-style matching behavior |

The common summary `peer = outgoing`, `user = incoming`, `friend = both` is oversimplified.

The real difference is mainly how Asterisk matches and identifies SIP traffic.

## Recommended Practice

- Use `type=peer` for most trunks.
- Use `type=peer` for many production phone configurations.
- Use `type=friend` in simple labs where bidirectional matching is convenient.
- Avoid `type=user` unless you specifically need username-based matching.

---

# 6. Example Extension Configuration

```ini
[1001]
type=friend
host=dynamic
secret=Replace-With-A-Strong-Unique-Password
context=office-extensions

disallow=all
allow=ulaw
allow=alaw

qualify=yes
dtmfmode=rfc2833
directmedia=no

callerid="Asif Desk" <1001>
```

## Section Name

```ini
[1001]
```

The section name is commonly used as the SIP username, authentication username, peer name, or endpoint identifier.

The dialplan extension can also be `1001`, but it does not have to match the peer name.

## `host=dynamic`

```ini
host=dynamic
```

This allows the phone to register from a DHCP or changing IP address.

For a fixed device or provider:

```ini
host=192.0.2.25
```

## `secret`

```ini
secret=Replace-With-A-Strong-Unique-Password
```

Use a long, unique password per account. Never publish real SIP secrets in GitHub.

## `context`

```ini
context=office-extensions
```

This controls which dialplan rules the phone can access and is one of Asterisk's most important security boundaries.

## `qualify`

```ini
qualify=yes
```

This enables SIP `OPTIONS` reachability checks.

## `dtmfmode`

```ini
dtmfmode=rfc2833
```

This controls how keypad digits are sent for IVR menus, voicemail, conference PINs, and similar functions.

RFC 2833 DTMF is often referred to as RFC 4733 in newer terminology.

## `directmedia`

```ini
directmedia=no
```

This keeps RTP flowing through Asterisk:

```text
Phone A → Asterisk → Phone B
```

Useful for:

- Recording
- Transcoding
- Music on hold
- NAT handling
- DTMF processing
- Monitoring

With `directmedia=yes`, phones may send RTP directly to each other, which can reduce server load but may break recording, NAT traversal, and media features.

## `callerid`

```ini
callerid="Asif Desk" <1001>
```

This sets the internal caller name and number.

---

# 7. SIP Trunk Registration

Some providers require Asterisk to register with them.

A registration string is placed under `[general]`:

```ini
register => username:password@sip.provider.example
```

A more detailed form may include:

```ini
register => username:password:authuser@sip.provider.example/inbound-user
```

> The value after `/` is generally used as the contact user for matching or routing inbound calls. It does not automatically send the call to a local extension unless the dialplan and peer configuration do so.

---

# 8. Example SIP Trunk Peer

```ini
[my_sip_provider]
type=peer
host=203.0.113.100

defaultuser=my_trunk_user
secret=Replace-With-Trunk-Password

context=incoming-calls

disallow=all
allow=ulaw
allow=alaw

qualify=yes
insecure=port,invite
```

> Use `insecure=port,invite` only when required by the provider and after understanding its security impact.

## `type=peer`

Generally appropriate for SIP trunks.

## `host`

```ini
host=203.0.113.100
```

Use the provider's official IP or hostname.

## `defaultuser`

```ini
defaultuser=my_trunk_user
```

Some older configurations use `username=` instead. Follow the provider and Asterisk version requirements.

## `context=incoming-calls`

Incoming provider calls enter this restricted dialplan context.

Never send provider traffic directly into an unrestricted internal or outbound context.

## Trunk Codec Selection

Do not automatically use G.729 just because the trunk is on the internet.

Choose codecs based on provider support, quality requirements, bandwidth, transcoding cost, licensing, and recording needs.

---

# 9. Security: Preventing Unauthorized Calls

Public SIP servers are frequently scanned by automated systems.

Common attacks include:

- Extension enumeration
- Password guessing
- Unauthorized registration
- Toll fraud
- International call abuse
- SIP flooding
- Malformed SIP messages
- INVITE scanning

Attack traffic can come from anywhere. Avoid attributing attacks to specific countries without verified evidence.

## Disable Guest Calls

```ini
allowguest=no
```

## Hide Username Validation Differences

```ini
alwaysauthreject=yes
```

This makes username enumeration more difficult by returning similar authentication failures for invalid users and invalid passwords.

## Restrict SIP at the Firewall

Example for a provider IP:

```bash
ufw allow from 203.0.113.100 to any port 5060 proto udp
```

Example for a trusted office network:

```bash
ufw allow from 192.168.1.0/24 to any port 5060 proto udp
```

Remote phones should preferably use WireGuard, IPsec, OpenVPN, a secure SBC, or TLS with strong authentication.

## Protect Against Toll Fraud

Use:

- Strong extension secrets
- Restricted dialplan contexts
- International dialing restrictions
- Rate limits
- Call-duration limits
- Provider credit limits
- Real-time CDR monitoring
- Fail2Ban
- IP allowlists
- Alerts for unusual call patterns

---

# 10. Troubleshooting One-Way Audio

Common causes:

- Incorrect NAT configuration
- Blocked RTP ports
- SIP ALG interference
- Wrong external IP
- Wrong local network definition
- Private IP addresses in SDP
- Multiple NAT layers
- Firewall session timeout
- Direct-media problems
- Asymmetric routing

## Check NAT Settings

```ini
localnet=192.168.1.0/255.255.255.0
externip=203.0.113.50
nat=force_rport,comedia
```

For individual remote peers:

```ini
nat=force_rport,comedia
directmedia=no
```

## Check the RTP Port Range

Asterisk commonly uses UDP `10000–20000`.

Configured in:

```text
/etc/asterisk/rtp.conf
```

Example:

```ini
[general]
rtpstart=10000
rtpend=20000
```

Firewall example:

```bash
ufw allow from 203.0.113.100 to any port 10000:20000 proto udp
```

A VPN or SBC is safer than exposing the RTP range globally.

## Disable SIP ALG

SIP ALG can cause broken registrations, one-way audio, dropped calls, incorrect ports, and invalid SDP.

## Inspect SIP and RTP

```asterisk
sip set debug on
sip set debug off
```

```asterisk
rtp set debug on
rtp set debug off
```

```bash
tcpdump -ni any udp port 5060
tcpdump -ni any udp portrange 10000-20000
```

---

# 11. `sip reload` vs `core reload`

## Reload Only `chan_sip`

```asterisk
sip reload
```

Use this after changing `sip.conf`.

## Reload Multiple Asterisk Modules

```asterisk
core reload
```

This has broader effects.

## Restarting Asterisk

Avoid restarting during active production calls because it can disconnect calls, remove registrations temporarily, interrupt queues, and stop conferences.

Not every setting can be safely applied through a reload, so some changes may still require a module or service restart.

---

# 12. Professional Example Configuration

```ini
[general]
context=public

allowguest=no
alwaysauthreject=yes
allowoverlap=no

udpbindaddr=192.168.1.10:5060
tcpenable=yes
tcpbindaddr=192.168.1.10:5060

localnet=192.168.1.0/255.255.255.0
externip=203.0.113.50

disallow=all
allow=ulaw
allow=alaw

register => my_trunk_user:Replace-With-Trunk-Password@sip.provider.example

[1001]
type=friend
host=dynamic
secret=Replace-With-A-Strong-Unique-Password
context=office-extensions

disallow=all
allow=ulaw
allow=alaw

qualify=yes
dtmfmode=rfc2833
nat=force_rport,comedia
directmedia=no

callerid="Asif Desk" <1001>

[my_sip_provider]
type=peer
host=sip.provider.example
defaultuser=my_trunk_user
secret=Replace-With-Trunk-Password
context=incoming-calls

disallow=all
allow=ulaw
allow=alaw

qualify=yes
```

---

# 13. Validation Commands

```asterisk
sip reload
sip show settings
sip show peers
sip show peer 1001
sip show registry
dialplan show office-extensions
core show channels
module show like chan_sip
```

---

# 14. Final Pro-Level Checklist

- [ ] The system intentionally uses legacy `chan_sip`.
- [ ] The default context is restricted.
- [ ] `allowguest=no` is enabled.
- [ ] `alwaysauthreject=yes` is enabled.
- [ ] SIP binds only to required interfaces.
- [ ] Firewall rules restrict SIP access.
- [ ] Local network ranges are correct.
- [ ] External IP or hostname is correct.
- [ ] RTP ports match `rtp.conf`.
- [ ] SIP ALG is disabled.
- [ ] Every extension uses a strong, unique password.
- [ ] Each device has a least-privileged dialplan context.
- [ ] Trunks use `type=peer`.
- [ ] Trunk traffic enters a restricted incoming context.
- [ ] Only required codecs are enabled.
- [ ] G.729 is used only when available, legal, and needed.
- [ ] `directmedia` matches recording, NAT, and media requirements.
- [ ] SIP and RTP debugging are disabled after troubleshooting.
- [ ] Configuration files have secure permissions.
- [ ] Real credentials are never committed to GitHub.
- [ ] Call limits and billing alerts reduce toll-fraud risk.

A professional `sip.conf` setup is not only about making phones register. It must also control authentication, dialing permissions, network exposure, media routing, and failure behavior.
