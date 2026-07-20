# Call Distribution Basics & Its Drawbacks

## Current Setup Limitation

So far, the setup uses one-to-one call routing:

```text
One outside number → One internal phone
```

The real strength of Asterisk is distributing calls to the correct person or team.

For example, if a caller needs support, they should be able to reach any available support team member instead of one specific person.

---

# The Simplest Approach: Ring Multiple Phones at Once

A basic group can be created by placing multiple SIP peers inside one `Dial()` application.

This is sometimes called a call group, hunt group, or ring group.

## Dialplan Example

Create extension `300` for the support team:

```ini
[phones]
exten => 300,1,NoOp(Support Team)
exten => 300,2,Dial(SIP/james&SIP/matthias,120)
exten => 300,3,Hangup()
```

## Syntax

```asterisk
Dial(SIP/james&SIP/matthias,120)
```

The ampersand:

```text
&
```

separates multiple destinations.

This rings James and Matthias at the same time.

The timeout:

```text
120
```

allows the phones to ring for up to 120 seconds.

---

# Testing

Dial:

```text
300
```

Expected result:

- James's phone rings.
- Matthias's phone rings.
- Both ring at the same time.
- The first person who answers receives the call.
- The other phone stops ringing.

---

# When This Approach Is Sufficient

For a small office or home-office environment with only two or three people, ringing all phones simultaneously may be enough.

Benefits:

- Easy to configure
- No extra configuration file
- Simple to understand
- Suitable for small teams

---

# Drawbacks

## 1. No Fair Queuing or Priority

If multiple callers call at the same time, there is no proper queue order.

The system does not clearly manage:

```text
First caller
Second caller
Third caller
```

You also cannot easily:

- Give priority to specific callers
- Keep callers waiting fairly
- Control the order in which calls are handled

---

## 2. No Flexible Call Strategy

The simple `Dial()` command rings every listed phone at once.

You may instead want:

```text
Ring James for 5 seconds
        ↓
If James does not answer
        ↓
Ring Matthias
```

This can be created using multiple `Dial()` commands and timeouts, but:

- The dialplan quickly becomes complicated.
- Changing the strategy requires rewriting the logic.
- Adding more users makes maintenance harder.

---

## 3. Static Members Only

The members inside `Dial()` are fixed.

Example:

```asterisk
Dial(SIP/james&SIP/matthias&SIP/user3&SIP/user4&SIP/user5)
```

Imagine a five-person support team:

- Three people are in the office.
- One person is at a customer site.
- One person is on holiday.

The dialplan still attempts to ring every configured phone.

There is no simple built-in concept of:

```text
I am available today.
```

or:

```text
I am not part of the support group today.
```

Team members cannot dynamically log in or out using this basic method.

---

# The Better Solution: Queues

Asterisk provides a dedicated call-distribution subsystem called:

```text
Queues
```

Queues provide:

- Fair caller queuing
- Call-distribution strategies
- Dynamic members
- Agent login and logout
- Hold music
- Waiting announcements
- Better handling of multiple callers

Common strategies can include:

- Ring all members
- Ring members sequentially
- Send the call to the member with the fewest calls
- Use other configured distribution methods

Even with only two people, a queue provides a cleaner and more professional structure.

---

# What Is Needed for Queues?

## Configuration File

```text
/etc/asterisk/queues.conf
```

## Queue Applications

Applications are used for:

- Sending callers into a queue
- Adding members
- Removing members
- Allowing agents to log in or out

## Dialplan Integration

The dialplan must send callers to the configured queue.

---

# Next Step

The next tutorial will cover the queue-based approach using:

```text
queues.conf
```

and the required dialplan applications.
