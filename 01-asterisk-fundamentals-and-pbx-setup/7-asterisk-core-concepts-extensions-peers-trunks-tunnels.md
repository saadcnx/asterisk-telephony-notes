# Asterisk Basics: Extension, SIP Peer, Trunk & Tunnel

## 1. Extension — The Home Address / Member Number

An **extension** is a unique internal phone number assigned to a user, phone, department, or service inside an organization.

In a typical office, every employee may have a desk phone with a number such as:

```text
101
102
103
```

These numbers are called extensions.

### Society Analogy

Imagine a housing society where every house has a unique address:

```text
House No. 101
House No. 102
House No. 103
```

The extension works in the same way. It identifies a specific phone or destination inside the private telephone system.

### VoIP Example

Suppose:

| User | Extension |
|---|---:|
| Asif | `101` |
| Bilal | `102` |

When Asif dials:

```text
102
```

Asterisk checks its dialplan and routes the call to Bilal's registered phone.

The call remains inside the Asterisk system and usually does not use an external telecom provider.

### Important Point

An extension is mainly a **logical number** used by the dialplan.

It may point to:

- A SIP phone
- A softphone
- A queue
- Voicemail
- An IVR
- A conference room
- A ring group
- Another server or trunk

---

## 2. SIP Peer or Endpoint — The Registered Resident

A SIP peer or endpoint represents a SIP device, user, application, or remote system that communicates with Asterisk.

In modern Asterisk systems using PJSIP, the term **endpoint** is more common.

Older Asterisk configurations using `chan_sip` often use the term **peer**.

### Society Analogy

Imagine a registered resident in a housing society.

The resident:

- Has a house number
- Has an identity record
- Is known to the security gate
- Is allowed to enter and communicate within the society

In the same way, a SIP phone must be configured and recognized by Asterisk.

### Registration Process

A SIP phone is usually configured with:

```text
Asterisk server IP address
SIP username
SIP password
Extension or account ID
```

For example:

```text
Server: 192.168.100.55
Username: 101
Password: StrongPasswordHere
```

The phone sends a SIP registration request to Asterisk.

If the credentials are correct, Asterisk records the phone's current IP address and contact information.

You can think of Asterisk responding:

```text
Extension 101 is registered and currently reachable at this IP address.
```

### VoIP Example

A softphone configured as extension `101` registers with Asterisk.

Asterisk then knows:

- The identity of the device
- Its authentication details
- Its current network address
- Which codecs it supports
- How to contact it when someone calls `101`

### Important Clarification

An extension and an endpoint are related, but they are not exactly the same thing.

- **Extension:** The number dialed in the dialplan
- **Endpoint or peer:** The SIP device or account Asterisk communicates with

For example:

```text
Extension 101 → Dial PJSIP endpoint 101
```

This is common, but the names do not have to be identical.

---

## 3. SIP Trunk — The Main Highway

A **trunk** connects an Asterisk system to an external telephone network, telecom provider, another PBX, or another Asterisk server.

It is used when calls need to leave the internal extension network.

### Society Analogy

Imagine a housing society with internal roads.

Residents can travel between houses using those internal roads.

To leave the society and travel to another city, they need access to the main highway.

That main highway is similar to a trunk.

### VoIP Example

An Asterisk server may connect to:

- PTCL
- Jazz
- A SIP service provider
- A call-center carrier
- Another office PBX
- An international VoIP provider

That connection is called a SIP trunk.

Asterisk uses the trunk for:

- Outbound calls
- Incoming public calls
- DID routing
- Inter-office communication
- Call-center traffic

### Outbound Call Example

Suppose extension `101` dials a mobile number:

```text
03001234567
```

Asterisk checks the dialplan and determines that this is not an internal extension.

It routes the call through the configured SIP trunk:

```text
Extension 101
      ↓
Asterisk Dialplan
      ↓
SIP Trunk
      ↓
Telecom Provider
      ↓
Mobile Number
```

### Incoming Call Example

A customer calls the company's public number.

The telecom provider sends the call over the SIP trunk to Asterisk.

Asterisk can then route it to:

- Reception
- An IVR
- A queue
- A specific extension
- Voicemail

### Simultaneous Call Capacity

A trunk may support one or many simultaneous calls.

The actual capacity depends on:

- Provider agreement
- Purchased channels
- Available bandwidth
- Server performance
- Codec choice
- Licensing or carrier restrictions

A trunk does not automatically carry 100 to 1,000 calls. Its capacity must be confirmed with the provider and infrastructure design.

---

## 4. Tunnel — The Private Network Passage

A network tunnel creates a logical path between two locations over another network, usually the internet.

Common tunnel technologies include:

- IPsec VPN
- WireGuard
- OpenVPN
- GRE
- SSH tunneling
- Site-to-site VPN

### Society Analogy

Imagine a private underground passage connecting two secure locations.

Instead of using public roads, authorized traffic travels through the dedicated passage.

A tunnel works similarly by carrying network traffic between two endpoints through an encapsulated path.

### VoIP Example

Suppose a company has:

- An Asterisk server in Lahore
- Another office in Karachi
- SIP phones in both locations

A site-to-site VPN tunnel can securely connect both office networks.

The phones may then communicate with Asterisk as though they are on one private network.

```text
Lahore Office
      ↓
Encrypted VPN Tunnel
      ↓
Karachi Office
```

### Tunneling and Encryption Are Not Always the Same

A tunnel is not automatically encrypted.

For example:

- **GRE** creates a tunnel but does not provide encryption by itself.
- **IPsec**, **WireGuard**, and **OpenVPN** can provide encrypted tunnels.

Therefore, use an encryption-capable VPN when confidentiality is required.

### What a Secure Tunnel Can Protect

A properly configured encrypted tunnel can help protect:

- SIP signaling
- RTP voice traffic
- Extension registration
- Inter-office calls
- Device provisioning
- Management traffic

However, a tunnel alone does not make the entire VoIP system completely secure.

You still need:

- Strong SIP passwords
- Firewall rules
- Restricted management access
- Fail2Ban or intrusion prevention
- Updated Asterisk packages
- TLS and SRTP where appropriate
- Network monitoring
- Proper user permissions

---

# Connecting Everything Together

Imagine an employee at extension `101` needs to call a customer's mobile number.

## Step 1: The Extension

The employee uses the phone assigned to:

```text
Extension 101
```

This is the internal number used within the office PBX.

## Step 2: The SIP Endpoint

The phone is configured as a SIP endpoint and is registered with Asterisk.

Asterisk knows the phone's:

- Username
- Authentication details
- IP address
- Contact status

## Step 3: The Dialplan Decision

The employee dials an external mobile number.

Asterisk checks the dialplan and recognizes that the number is outside the internal extension range.

## Step 4: The SIP Trunk

Asterisk sends the call to the telecom provider through the configured SIP trunk.

```text
Extension 101
      ↓
Registered SIP Endpoint
      ↓
Asterisk Dialplan
      ↓
SIP Trunk
      ↓
Telecom Provider
      ↓
Customer Mobile
```

## Step 5: The Secure Tunnel

If the Asterisk server and provider or remote office are connected through an encrypted VPN, the network traffic travels through that secure tunnel.

The tunnel protects the traffic while it moves between the two network endpoints.

---

# Quick Comparison

| Concept | Simple Meaning | Main Purpose |
|---|---|---|
| Extension | Internal phone number | Identifies a destination inside the PBX |
| SIP peer or endpoint | Registered SIP device or account | Allows Asterisk to communicate with a phone or system |
| SIP trunk | External telephone connection | Carries calls outside the internal PBX |
| Tunnel | Private logical network path | Carries network traffic between two locations |

---

# Simple Real-World Example

Consider this office setup:

```text
Asif's phone: Extension 101
Bilal's phone: Extension 102
Asterisk server: 192.168.100.55
Provider trunk: company-sip-trunk
VPN tunnel: Lahore office ↔ Karachi office
```

### Internal Call

```text
101 calls 102
```

Flow:

```text
Asif's SIP Endpoint
      ↓
Asterisk
      ↓
Bilal's SIP Endpoint
```

### External Call

```text
101 calls 03001234567
```

Flow:

```text
Asif's SIP Endpoint
      ↓
Asterisk
      ↓
SIP Trunk
      ↓
Mobile Network
```

### Inter-Office Secure Call

```text
Lahore extension 101 calls Karachi extension 201
```

Flow:

```text
Extension 101
      ↓
Lahore Asterisk Server
      ↓
Encrypted VPN Tunnel
      ↓
Karachi Network or PBX
      ↓
Extension 201
```

These four concepts form the foundation of most Asterisk and VoIP environments.
