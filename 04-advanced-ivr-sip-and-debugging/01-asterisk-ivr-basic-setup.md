
# IVR Menu: Basic Setup

## What We Are Building

Using two recorded prompts, we will build a basic IVR menu in the Asterisk dialplan.

The prompts are:

```text
Question 1: Press 1 for Fruit, Press 2 for Vegetables
Question 2: Press 1 for Bananas, Press 2 for Apples
```

This lesson focuses on the first menu.

---

# The Prompt Files

The audio files were recorded in Audacity, exported, and copied to the Asterisk sounds directory.

```text
ivr_q1.wav
```

Prompt:

```text
Press 1 for Fruit, Press 2 for Vegetables
```

```text
ivr_q2.wav
```

Prompt:

```text
Press 1 for Bananas, Press 2 for Apples
```

---

# Entry Extension

Create extension:

```text
800
```

This acts as the entry point for the IVR.

---

# Critical Rule: Every IVR Gets Its Own Context

Each IVR menu should have a dedicated context.

For this menu, we use:

```ini
[ivr1]
```

This isolates the menu options from the rest of the dialplan.

Without a dedicated context, key presses such as `1` or `2` could match unrelated extensions in another context.

Each IVR menu and sub-menu should therefore have its own namespace.

---

# Basic Dialplan

```ini
exten => 800,1,Goto(ivr1,start,1)

[ivr1]
exten => start,1,NoOp(IVR 1)
exten => start,n,Answer()
exten => start,n,Playback(ivr_q1)
exten => start,n,WaitExten(5)

exten => 1,1,NoOp(Pressed 1 - Fruit)
exten => 1,n,Goto(fruit,s,1)

exten => 2,1,NoOp(Pressed 2 - Vegetables)
exten => 2,n,Goto(veg,s,1)
```

---

# Entry Point

```ini
exten => 800,1,Goto(ivr1,start,1)
```

When the caller dials `800`, Asterisk jumps to:

```text
Context: ivr1
Extension: start
Priority: 1
```

---

# The IVR Context

```ini
[ivr1]
```

This context contains only the logic for the first IVR menu.

---

# Logging the Menu Entry

```ini
exten => start,1,NoOp(IVR 1)
```

`NoOp()` writes a message to the Asterisk CLI.

It is useful for testing and troubleshooting.

---

# Answering the Call

```ini
exten => start,n,Answer()
```

This answers the call so the caller can hear the prompt and send DTMF input.

---

# Playing the Prompt

```ini
exten => start,n,Playback(ivr_q1)
```

Asterisk plays:

```text
ivr_q1.wav
```

The file extension is not included in `Playback()`.

---

# Waiting for Input

```ini
exten => start,n,WaitExten(5)
```

This waits up to five seconds for the caller to press a key.

If a valid extension exists for the pressed digit, Asterisk routes to that extension.

The current version does not yet handle timeout behavior.

---

# Handling Key 1

```ini
exten => 1,1,NoOp(Pressed 1 - Fruit)
exten => 1,n,Goto(fruit,s,1)
```

When the caller presses `1`, Asterisk:

1. Logs `Pressed 1 - Fruit`.
2. Jumps to the `fruit` context.
3. Starts at extension `s`, priority `1`.

The `fruit` context has not been created yet.

---

# Handling Key 2

```ini
exten => 2,1,NoOp(Pressed 2 - Vegetables)
exten => 2,n,Goto(veg,s,1)
```

When the caller presses `2`, Asterisk:

1. Logs `Pressed 2 - Vegetables`.
2. Jumps to the `veg` context.
3. Starts at extension `s`, priority `1`.

The `veg` context has not been created yet.

---

# Complete Call Flow

```text
Caller dials 800
        ↓
Goto(ivr1,start,1)
        ↓
Answer()
        ↓
Playback(ivr_q1)
        ↓
WaitExten(5)
        ↓
Press 1 → Goto(fruit,s,1)
Press 2 → Goto(veg,s,1)
```

---

# Testing

Reload the dialplan:

```asterisk
dialplan reload
```

Dial:

```text
800
```

Expected behavior:

1. The call is answered.
2. The caller hears the first prompt.
3. Asterisk waits for input.

## Pressing 1

The CLI shows:

```text
Pressed 1 - Fruit
```

Asterisk tries to enter:

```text
fruit,s,1
```

Because the context is not built yet, an error is expected.

## Pressing 2

The CLI shows:

```text
Pressed 2 - Vegetables
```

Asterisk tries to enter:

```text
veg,s,1
```

Because the context is not built yet, an error is expected.

---

# Current Limitations

## 1. Invalid Keys Are Not Handled

If the caller presses `3`, `4`, `5`, or `#`, there is no matching extension.

The dialplan does not yet provide an error message or replay the menu.

## 2. Timeout Is Not Handled

If the caller does not press anything within five seconds, `WaitExten()` ends.

There is no timeout-routing logic yet.

A professional IVR should replay the menu, redirect to an operator, play a timeout message, or end the call cleanly.

## 3. Sub-Menus Need Separate Contexts

The fruit sub-menu might ask:

```text
Press 1 for Bananas
Press 2 for Apples
```

This sub-menu must use its own context, such as:

```ini
[fruit]
```

Otherwise, the same digit could conflict with the main menu.

## 4. Destinations Do Not Exist Yet

The following destinations have not been created:

```text
fruit
veg
```

The actual sub-menus, queues, or departments will be added later.

---

# Next Step

The next version will add:

- Invalid input handling
- Timeout handling
- Dedicated sub-menu contexts
- Proper fruit and vegetable destinations
- More reliable IVR behavior
