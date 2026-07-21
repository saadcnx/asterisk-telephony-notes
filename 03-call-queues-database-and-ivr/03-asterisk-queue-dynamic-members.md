# Dynamic Queue Members

## The Problem with Static Members

Previously, queue members were defined directly inside:

```text
/etc/asterisk/queues.conf
```

Example:

```ini
[support]
member => SIP/james
member => SIP/matthias
```

These are static members.

Their phones remain part of the queue even when they are:

- On holiday
- At a customer site
- Away from their desk
- Not available to answer calls

This can be inefficient and annoying for other team members.

---

# The Solution: Dynamic Members

Dynamic members can be added to or removed from a queue while Asterisk is running.

This means only available users receive queue calls.

There are two main ways to manage dynamic members:

1. Asterisk CLI commands for administrators
2. Dialplan applications for users

This lesson focuses on CLI management.

---

# Step 1: Removing Static Members

A member defined with:

```ini
member => SIP/james
```

inside `queues.conf` is static.

Static members cannot be removed with dynamic queue-member commands.

To make a member dynamic:

1. Remove or comment out the `member =>` line.
2. Reload the queue configuration.

Example:

```ini
[support]
; member => SIP/james
; member => SIP/matthias
```

Reload:

```asterisk
queue reload all
```

The queue now has no members unless other static members remain.

---

# Step 2: Adding a Dynamic Member

Use the following command inside the Asterisk CLI:

```asterisk
queue add member SIP/james to support
```

No reload is required.

The member is added immediately.

Verify:

```asterisk
queue show support
```

The member should appear with a dynamic indicator.

---

# Removing a Dynamic Member

Use:

```asterisk
queue remove member SIP/james from support
```

The member is removed immediately.

No reload is required.

---

# Trying to Remove a Static Member

Suppose Matthias is still defined inside `queues.conf`:

```ini
member => SIP/matthias
```

Running:

```asterisk
queue remove member SIP/matthias from support
```

returns an error similar to:

```text
Unable to remove interface because the member is not dynamic.
```

Static members must be removed from the configuration file and then reloaded.

---

# Persistent Dynamic Members

In older Asterisk versions, dynamic members could be lost after a restart.

With persistent member storage enabled, dynamic queue memberships are stored in the Asterisk database.

This allows dynamic members to remain after:

- Queue reload
- Asterisk reload
- Full Asterisk restart

---

# Demonstrating Dynamic Behavior

Assume both members are added dynamically:

```asterisk
queue add member SIP/james to support
queue add member SIP/matthias to support
```

The queue strategy is:

```ini
strategy = ringall
```

## Initial Call

A caller enters the queue.

Both phones ring:

```text
James
Matthias
```

## Remove James During the Call

While the caller is still waiting, run:

```asterisk
queue remove member SIP/james from support
```

After the current agent attempt ends, the queue re-evaluates its member list.

Now only Matthias remains.

The next queue attempt rings:

```text
Matthias only
```

James no longer receives queue calls.

---

# Relationship with Agent Timeout

The agent timeout determines how long the current queue attempt lasts.

When that attempt ends, the caller returns to the queue and Asterisk re-evaluates:

- Current members
- Available members
- Queue strategy
- Member status

If a member is removed before the next attempt, that member is no longer selected.

If the overall queue timeout expires, the caller exits the queue and the dialplan continues.

---

# The Problem with CLI-Only Management

CLI commands are useful for administrators.

However, they are not convenient for normal users.

A user should not need to contact an administrator and ask:

```text
Please add me to the support queue.
```

Users need a self-service method.

---

# Next Step

Asterisk provides dialplan applications that allow users to manage their own queue membership.

The next tutorial will create extensions that users can dial to:

- Add themselves to a queue
- Remove themselves from a queue

This makes dynamic queue membership easier and more user-friendly.
