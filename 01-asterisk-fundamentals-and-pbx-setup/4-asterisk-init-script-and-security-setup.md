# Asterisk Init Script & Security Setup

## Goals for This Setup

Before configuring Asterisk, we should create a proper and professional runtime environment.

This setup has two main goals:

1. **A working init script**
   - Asterisk should run as a background service or daemon.
   - We should be able to start, stop, and restart it.
   - Closing the Asterisk console should not stop the server.

2. **A dedicated non-root user**
   - Asterisk should not run as the `root` user.
   - If Asterisk is compromised while running as root, an attacker could gain complete control of the server.
   - A separate `asterisk` user and group reduce the possible damage.

---

# Part 1: Fixing the Init Script Placeholders

The init script copied to:

```text
/etc/init.d/asterisk
```

may contain placeholders that must be replaced with the correct paths.

## 1. Locate the Asterisk Binary

To find the installed Asterisk binary, run:

```bash
type asterisk
```

It is commonly installed at:

```text
/usr/sbin/asterisk
```

You can also use:

```bash
which asterisk
```

## 2. Edit the Init Script

Open the init script:

```bash
nano /etc/init.d/asterisk
```

Replace the placeholders with the correct paths.

| Setting | Path |
|---|---|
| Asterisk daemon binary | `/usr/sbin/asterisk` |
| Runtime directory | `/var/run/asterisk` |
| Configuration directory | `/etc/asterisk` |

The runtime directory is normally created during the Asterisk installation process.

## 3. Make the Script Executable

Make sure the init script has execute permissions:

```bash
chmod +x /etc/init.d/asterisk
```

## 4. Start Asterisk as a Daemon

Start Asterisk using the init script:

```bash
service asterisk start
```

Alternatively:

```bash
/etc/init.d/asterisk start
```

Asterisk should now run as a background daemon.

## 5. Connect to the Asterisk CLI

To connect to the console of the already-running Asterisk server, use:

```bash
asterisk -r
```

For more verbose output:

```bash
asterisk -rvvv
```

To leave the CLI without stopping the server, enter:

```asterisk
exit
```

Closing the remote console does not terminate the Asterisk daemon.

---

# Part 2: Creating a Safe Non-Root User

Even after the init script works, Asterisk may still be running as `root`. This is a security risk.

We should run it under a dedicated system user named `asterisk`.

## 1. Stop the Asterisk Server

Stop the currently running Asterisk process:

```bash
service asterisk stop
```

Confirm that it has stopped:

```bash
ps aux | grep asterisk
```

## 2. Create the Asterisk Group and User

Create a dedicated system group:

```bash
groupadd --system asterisk
```

Create the `asterisk` system user:

```bash
useradd   --system   --home-dir /var/lib/asterisk   --gid asterisk   --shell /usr/sbin/nologin   asterisk
```

This configuration:

- Uses `/var/lib/asterisk` as the home directory.
- Assigns the user to the `asterisk` group.
- Prevents interactive shell login.
- Creates a system account rather than a normal login user.

> No command output usually means the user was created successfully.

To verify the account, run:

```bash
id asterisk
```

## 3. Change Directory Ownership

The `asterisk` user needs permission to write to Asterisk runtime, data, log, and spool directories.

Run:

```bash
chown -R asterisk:asterisk /var/lib/asterisk
chown -R asterisk:asterisk /var/spool/asterisk
chown -R asterisk:asterisk /var/run/asterisk
chown -R asterisk:asterisk /var/log/asterisk
```

You may also need to give the user ownership of the configuration directory when Asterisk or related tools must modify configuration files:

```bash
chown -R asterisk:asterisk /etc/asterisk
```

> On tightly controlled production systems, configuration files can remain owned by `root` and only be made readable by the `asterisk` group. This reduces the risk of unauthorized configuration changes.

## 4. Copy the Default Configuration File

The source code includes a defaults file for the init script.

From the Asterisk source directory, copy it using:

```bash
cp contrib/init.d/etc_default_asterisk /etc/default/asterisk
```

Open the copied file:

```bash
nano /etc/default/asterisk
```

Find the user and group settings and configure them as:

```bash
AST_USER="asterisk"
AST_GROUP="asterisk"
```

Depending on the Asterisk version, the variables may appear as:

```bash
AST_USER=asterisk
AST_GROUP=asterisk
```

The important point is that they must not remain commented out.

These variables tell the init script to run the Asterisk process under the dedicated user and group.

## 5. Configure `asterisk.conf`

Open the main Asterisk configuration file:

```bash
nano /etc/asterisk/asterisk.conf
```

Under the `[options]` section, add or confirm:

```ini
[options]
runuser = asterisk
rungroup = asterisk
```

This gives Asterisk another explicit instruction to drop root privileges and run as the dedicated user.

## 6. Start Asterisk Again

Start the server:

```bash
service asterisk start
```

Connect to the CLI:

```bash
asterisk -rvvv
```

## 7. Verify the Running User

Check the process owner:

```bash
ps -eo user,group,pid,cmd | grep '[a]sterisk'
```

The output should show:

```text
asterisk  asterisk  ...
```

It should not show `root` as the owner of the main Asterisk process.

You can also check with:

```bash
pgrep -a asterisk
```

---

# Part 3: Fixing the `live_dangerously` Warning

You may see a warning related to:

```text
Privilege escalation protection disabled
```

or the `live_dangerously` option.

## What Is `live_dangerously`?

Asterisk dialplans can use functions that read or write information during call processing.

Some write operations can modify files or system-related data. Although this functionality can be useful, it may become dangerous if an attacker gains access to the dialplan or manager interfaces.

Older configurations may enable this behavior for backward compatibility.

## How to Disable It

Open the main Asterisk configuration file:

```bash
nano /etc/asterisk/asterisk.conf
```

Locate the `[options]` section and set:

```ini
live_dangerously = no
```

A secure example configuration is:

```ini
[options]
runuser = asterisk
rungroup = asterisk
live_dangerously = no
```

Save the file and restart Asterisk:

```bash
service asterisk restart
```

Reconnect to the CLI:

```bash
asterisk -rvvv
```

The warning should no longer appear.

---

# Final Verification Checklist

Use the following checklist to confirm that the setup is complete:

- [ ] `/etc/init.d/asterisk` exists.
- [ ] The init script is executable.
- [ ] The Asterisk binary path is correct.
- [ ] Asterisk starts as a background daemon.
- [ ] `asterisk -r` connects to the running server.
- [ ] Typing `exit` closes only the CLI.
- [ ] The `asterisk` user and group exist.
- [ ] Asterisk directories have the correct ownership.
- [ ] `/etc/default/asterisk` defines `AST_USER` and `AST_GROUP`.
- [ ] `asterisk.conf` defines `runuser` and `rungroup`.
- [ ] `live_dangerously` is set to `no`.
- [ ] The Asterisk process runs as `asterisk:asterisk`, not as `root`.

---

# Useful Service Commands

## Start Asterisk

```bash
service asterisk start
```

## Stop Asterisk

```bash
service asterisk stop
```

## Restart Asterisk

```bash
service asterisk restart
```

## Check Service Status

```bash
service asterisk status
```

## Open the Asterisk CLI

```bash
asterisk -rvvv
```

## Exit the CLI Without Stopping Asterisk

```asterisk
exit
```

After completing these steps, Asterisk should run in the background under a restricted user account with safer privilege settings.
