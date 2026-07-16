# Network Configuration, DHCP & NTP Setup

## Preliminary Fix: Start Asterisk Automatically on Reboot

In the previous setup, we created an Asterisk init script but did not enable it to start automatically when the server reboots.

Run:

```bash
update-rc.d asterisk defaults
```

This registers the Asterisk init script with the appropriate system runlevels so that Asterisk starts automatically during boot.

To confirm that the service links were created, you can inspect:

```bash
ls -l /etc/rc*.d/ | grep asterisk
```

> On newer Linux systems that use `systemd`, service enablement is usually handled with `systemctl enable`, but this guide follows the traditional SysV init setup.

---

# Part 1: Network Environment Setup

In a test or studio environment using a physical switch and router, the Asterisk server should usually be connected directly to the local network.

If the server is running inside a virtual machine, change the virtual network adapter from **NAT** mode to **Bridged** mode.

## Why Use Bridged Networking?

With bridged networking:

- The Asterisk server appears as a separate device on the local network.
- SIP phones can communicate with it directly.
- The server receives an IP address from the same network as the phones.
- Web interfaces and SIP traffic are easier to test.

## 1. Configure a Static IP Address

Open the network interface configuration file:

```bash
nano /etc/network/interfaces
```

Configure a static address for the studio network.

Example:

```ini
auto eth0
iface eth0 inet static
    address 192.168.100.55
    netmask 255.255.255.0
    gateway 192.168.100.1
    dns-nameservers 8.8.8.8 1.1.1.1
```

### Example Network Values

| Setting | Example Value |
|---|---|
| IP address | `192.168.100.55` |
| Netmask | `255.255.255.0` |
| Gateway | `192.168.100.1` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `1.1.1.1` |

> Replace `eth0` with the actual network interface name on your system.

To find the interface name, run:

```bash
ip address
```

or:

```bash
ip link
```

## 2. Restart the Network Service

Apply the new network configuration:

```bash
service networking restart
```

Alternatively:

```bash
/etc/init.d/networking restart
```

Check the assigned IP address:

```bash
ip address show
```

Check the default gateway:

```bash
ip route
```

Test local network connectivity:

```bash
ping -c 4 192.168.100.1
```

Test internet connectivity using an IP address:

```bash
ping -c 4 8.8.8.8
```

## 3. Fix DNS Resolution

If you can ping `8.8.8.8` but cannot ping a domain name, the network is working but DNS resolution is failing.

Test DNS:

```bash
ping -c 4 google.com
```

Add a DNS server to the network configuration:

```ini
dns-nameservers 8.8.8.8 1.1.1.1
```

On older systems, you may temporarily add a resolver directly to:

```text
/etc/resolv.conf
```

Example:

```text
nameserver 8.8.8.8
nameserver 1.1.1.1
```

> Changes made directly to `/etc/resolv.conf` may be overwritten by the networking service, DHCP client, or resolver manager. The preferred method is to define DNS in the network configuration.

---

# Part 2: DHCP Server Setup for SIP Phones

SIP phones need IP addresses before they can communicate with the Asterisk server.

A DHCP server automatically provides phones with:

- IP addresses
- Netmask
- Default gateway
- DNS server
- Lease duration
- Optional provisioning information

Common DHCP server packages include:

```text
udhcpd
isc-dhcp-server
```

Use only one DHCP server on the same network segment to avoid conflicting responses.

## Option A: Using `udhcpd`

### 1. Install the DHCP Server

```bash
apt-get update
apt-get install udhcpd
```

### 2. Enable the Service

Open:

```bash
nano /etc/default/udhcpd
```

Depending on the version, remove or disable the line that prevents the service from starting.

For example, change:

```text
DHCPD_ENABLED="no"
```

to:

```text
DHCPD_ENABLED="yes"
```

Some versions use:

```text
ENABLED="yes"
```

Check the file comments and use the variable expected by your installed package.

### 3. Configure the Address Pool

Open:

```bash
nano /etc/udhcpd.conf
```

Example configuration:

```ini
start           192.168.100.200
end             192.168.100.220

interface       eth0

opt subnet      255.255.255.0
opt router      192.168.100.1
opt dns         8.8.8.8 1.1.1.1
opt lease       86400
```

### Configuration Explanation

| Option | Purpose |
|---|---|
| `start` | First IP address in the DHCP pool |
| `end` | Last IP address in the DHCP pool |
| `interface` | Network interface serving DHCP |
| `opt subnet` | Subnet mask |
| `opt router` | Default gateway |
| `opt dns` | DNS servers |
| `opt lease` | Lease duration in seconds |

Remove or comment out unnecessary options such as WINS server settings when they are not required.

### 4. Restart the DHCP Server

```bash
service udhcpd restart
```

Check its status:

```bash
service udhcpd status
```

## Option B: Using `isc-dhcp-server`

### 1. Install the Package

```bash
apt-get update
apt-get install isc-dhcp-server
```

### 2. Select the Network Interface

Open:

```bash
nano /etc/default/isc-dhcp-server
```

Set the interface:

```ini
INTERFACESv4="eth0"
```

Replace `eth0` with the correct interface name.

### 3. Configure the DHCP Scope

Open:

```bash
nano /etc/dhcp/dhcpd.conf
```

Example:

```dhcp
authoritative;

subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.200 192.168.100.220;
    option routers 192.168.100.1;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 8.8.8.8, 1.1.1.1;
    default-lease-time 3600;
    max-lease-time 86400;
}
```

### 4. Restart the Service

```bash
service isc-dhcp-server restart
```

Check its status:

```bash
service isc-dhcp-server status
```

## Important DHCP Warning

Before enabling a DHCP server, confirm that another DHCP server is not already active on the same network.

A home or office router usually runs its own DHCP service.

If both the router and Asterisk server respond to DHCP requests, phones may receive incorrect or inconsistent settings.

For a controlled lab, either:

- Disable DHCP on the router and use the Asterisk server, or
- Keep the router DHCP enabled and do not run another DHCP server.

## Monitoring DHCP Activity

To watch DHCP requests in real time:

```bash
tail -f /var/log/syslog
```

Filter DHCP-related messages:

```bash
tail -f /var/log/syslog | grep -i dhcp
```

You can also inspect DHCP traffic with:

```bash
tcpdump -ni eth0 port 67 or port 68
```

A typical DHCP exchange follows this process:

```text
DHCP Discover
      ↓
DHCP Offer
      ↓
DHCP Request
      ↓
DHCP Acknowledgement
```

---

# Part 3: NTP Server and Time Synchronization

Accurate time is important for both the Asterisk server and SIP phones.

Correct time affects:

- Call detail records
- Log timestamps
- Voicemail timestamps
- Scheduled dialplan actions
- Certificate validation
- Troubleshooting
- Phone display clocks

## 1. Check the Current Server Time

Run:

```bash
date
```

Check the timezone:

```bash
timedatectl
```

If required, set the correct timezone.

Example for Pakistan:

```bash
timedatectl set-timezone Asia/Karachi
```

Verify:

```bash
timedatectl
```

## 2. Stop the NTP Service Temporarily

If an NTP daemon is already running, stop it before using `ntpdate` manually.

For the traditional `ntp` service:

```bash
service ntp stop
```

For `systemd-timesyncd`:

```bash
systemctl stop systemd-timesyncd
```

## 3. Synchronize the Clock Once

Install `ntpdate` if it is not already available:

```bash
apt-get update
apt-get install ntpdate
```

Synchronize the local time:

```bash
ntpdate pool.ntp.org
```

`pool.ntp.org` automatically selects available public time servers.

Verify the corrected time:

```bash
date
```

## 4. Start the NTP Server Again

Start the NTP daemon:

```bash
service ntp start
```

Restart it if necessary:

```bash
service ntp restart
```

Check its status:

```bash
service ntp status
```

You can inspect synchronization peers using:

```bash
ntpq -p
```

## 5. Allow Phones to Use the Asterisk Server as Their NTP Source

Once the server's clock is correct and the NTP daemon is running, SIP phones can use the Asterisk server as their time source.

Configure each phone with the Asterisk server's IP address:

```text
192.168.100.55
```

The phone will then request time from the local server instead of contacting an external NTP server directly.

This provides consistent time across:

- The Asterisk server
- SIP phones
- Call logs
- Recordings
- Voicemail messages

## NTP Firewall Requirement

NTP uses:

```text
UDP port 123
```

If a firewall is enabled, allow NTP traffic from the local network.

Example with UFW:

```bash
ufw allow from 192.168.100.0/24 to any port 123 proto udp
```

---

# Part 4: Setting Up Test Phones

After networking, DHCP, and time synchronization are working, you can connect physical and software-based SIP phones.

## 1. Hardware SIP Phone Test

### Connect the Phone

Connect a standard SIP phone to the network switch.

The phone should:

1. Power on.
2. Send a DHCP request.
3. Receive an IP address.
4. Receive gateway and DNS information.
5. Synchronize its time through NTP, if configured.

### Verify the DHCP Lease

Check server logs:

```bash
tail -f /var/log/syslog | grep -i dhcp
```

Look for an assigned address within the configured range, such as:

```text
192.168.100.200
```

You may also check the phone's screen or network settings menu.

## 2. Test Network Reachability

From the Asterisk server, ping the phone:

```bash
ping -c 4 192.168.100.200
```

Replace the example address with the actual IP assigned to the phone.

## 3. Test the Phone's Web Interface

Many SIP phones include a browser-based management interface.

Open a browser and visit:

```text
http://192.168.100.200
```

or, when HTTPS is supported:

```text
https://192.168.100.200
```

If the login page opens, the phone is active and reachable.

> Default usernames and passwords vary by manufacturer. Change default credentials before using the phone in a production environment.

## 4. Prepare a Softphone

A softphone is a software-based SIP telephone installed on a computer or mobile device.

It is useful because:

- No additional hardware is required.
- It allows quick extension testing.
- It can test internal calls between two endpoints.
- It helps verify SIP registration, audio, codecs, and dialplan rules.

The next step is to configure:

- One extension for the hardware phone
- One extension for the softphone
- Authentication credentials
- Dialplan rules for calling between them

---

# Final Environment Status

After completing this setup, the lab environment should contain:

- [ ] Asterisk enabled at system startup
- [ ] A static IP address on the Asterisk server
- [ ] Working gateway connectivity
- [ ] Working DNS resolution
- [ ] A DHCP service or router assigning phone addresses
- [ ] Correct server date and timezone
- [ ] An active NTP service
- [ ] A physical SIP phone connected to the network
- [ ] Access to the phone's web interface
- [ ] A softphone ready for registration
- [ ] All devices connected to the same local network

The environment is now ready for SIP extension registration and internal call testing.