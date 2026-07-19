# Variables & String Manipulation in the Dialplan

## The Problem Recap

Previously, we used the pattern:

```ini
_0X.
```

to match outbound calls.

However, the destination was still hardcoded. No matter what number the user dialed, Asterisk always called:

```text
8888
```

We need to capture the number the user actually dialed and pass it dynamically to the external peer or SIP provider.

---

# Step 1: Understanding the `${EXTEN}` Variable

Asterisk provides many built-in dialplan variables.

One of the most important is:

```asterisk
${EXTEN}
```

## What `${EXTEN}` Stores

`${EXTEN}` contains the extension or number that was actually dialed.

It does not contain the dialplan pattern.

For example, if the pattern is:

```ini
_0X.
```

and the user dials:

```text
02222
```

then:

```asterisk
${EXTEN}
```

contains:

```text
02222
```

It does not contain:

```text
_0X.
```

---

# Testing `${EXTEN}` with `NoOp()`

Use `NoOp()` to print the variable in the Asterisk CLI.

```ini
[phones]
exten => _0X.,1,NoOp(${EXTEN})
```

## Test Results

Dial:

```text
02222
```

The CLI prints:

```text
02222
```

Dial:

```text
0123
```

The CLI prints:

```text
0123
```

The value changes dynamically according to the number dialed by the user.

---

# Step 2: String Manipulation

In many PBX systems, the first zero is only an outside-line prefix.

For example:

```text
02222
```

The user dials `0` to request an external call, but the actual number sent to the provider should be:

```text
2222
```

Asterisk allows us to extract part of a variable using an offset and an optional length.

---

# String Slicing Syntax

```asterisk
${EXTEN:offset:length}
```

| Part | Description |
|---|---|
| `offset` | The position where extraction starts |
| `length` | The number of characters to extract |
| Positive offset | Counts from the beginning |
| Negative offset | Counts backward from the end |
| Omitted length | Extracts everything from the starting position to the end |

---

# String Manipulation Examples

Assume the dialed number is:

```text
02222
```

## Remove the First Digit

```asterisk
${EXTEN:1}
```

Result:

```text
2222
```

Explanation:

- Skip one character from the beginning.
- Return everything after it.

---

## Skip One Digit and Take Four Digits

```asterisk
${EXTEN:1:4}
```

Dialed number:

```text
02222
```

Result:

```text
2222
```

Explanation:

- Start after the first digit.
- Extract exactly four characters.

---

## Ignore Extra Digits Using a Fixed Length

```asterisk
${EXTEN:1:4}
```

Dialed number:

```text
022223
```

Result:

```text
2222
```

Explanation:

- Skip the first digit.
- Take only the next four digits.
- Ignore the remaining digit.

---

## Extract the Last Three Digits

```asterisk
${EXTEN:-3}
```

Dialed number:

```text
02222
```

Result:

```text
222
```

Explanation:

- Start three characters from the end.
- Extract everything from that position to the end.

---

# Why This Is Important

Expressions such as:

```asterisk
${EXTEN:1}
```

appear frequently in Asterisk dialplan examples.

They are commonly used to:

- Remove an outside-line prefix
- Remove a country-code prefix
- Extract part of a DID
- Normalize dialed numbers
- Pass a cleaned number to a provider

Understanding offsets and lengths is essential for dynamic call routing.

---

# Step 3: Applying String Manipulation to Outbound Calls

We will now remove the outside-line prefix and pass the remaining number to the `outgoing` context.

---

# The `phones` Context

```ini
[phones]
exten => _0X.,1,Goto(outgoing,${EXTEN:1},1)
```

## Explanation

The pattern:

```ini
_0X.
```

matches numbers that:

- Begin with `0`
- Contain at least two additional characters

The expression:

```asterisk
${EXTEN:1}
```

removes the first digit and keeps the rest.

For example:

```text
Dialed number: 02222
Result:        2222
```

The final `Goto()` becomes:

```asterisk
Goto(outgoing,2222,1)
```

---

# The `outgoing` Context

```ini
[outgoing]
exten => _X.,1,Dial(SIP/outside/${EXTEN})
```

## Explanation

The pattern:

```ini
_X.
```

matches a number beginning with any digit and containing at least one additional character.

The zero prefix is no longer present because it was removed in the `phones` context.

The command:

```asterisk
Dial(SIP/outside/${EXTEN})
```

calls the peer named:

```text
outside
```

and passes the current dialed number to it.

For example:

```asterisk
Dial(SIP/outside/2222)
```

In a real SIP provider setup, the command may look like:

```asterisk
Dial(SIP/my_provider/${EXTEN})
```

---

# Full Working Example

```ini
[phones]
; Internal call
exten => 100,1,Dial(SIP/james)

; Outbound calls: remove the leading zero
exten => _0X.,1,Goto(outgoing,${EXTEN:1},1)

[outgoing]
; Dial the cleaned number through the outside peer
exten => _X.,1,Dial(SIP/outside/${EXTEN})
```

---

# Complete Call Flow

The user dials:

```text
02222
```

## Step 1: Pattern Match

The `phones` context matches:

```ini
_0X.
```

## Step 2: Remove the Prefix

```asterisk
${EXTEN:1}
```

changes:

```text
02222
```

into:

```text
2222
```

## Step 3: Move to the Outgoing Context

Asterisk executes:

```asterisk
Goto(outgoing,2222,1)
```

## Step 4: Match the Clean Number

The `outgoing` context matches:

```ini
_X.
```

## Step 5: Place the Call

Asterisk executes:

```asterisk
Dial(SIP/outside/2222)
```

The correct number is now passed dynamically to the external peer.

---

# Final Result

The user can dial any number that begins with the outside-line prefix:

```text
0
```

Asterisk then:

1. Captures the number using `${EXTEN}`.
2. Removes the leading zero using `${EXTEN:1}`.
3. Passes the cleaned number to the `outgoing` context.
4. Sends the actual number dynamically to the external peer or SIP provider.
