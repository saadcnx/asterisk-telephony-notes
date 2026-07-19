# Dialplan Pattern Matching

## The Problem

Last time, we hardcoded a single external number:

```text
8888
```

in the dialplan.

In the real world, we cannot create a separate dialplan line for every possible telephone number. We need placeholders that allow one rule to match many numbers.

---

# What Is Pattern Matching in Asterisk?

Asterisk provides a limited pattern-matching syntax for telephone numbers.

It is not the same as full regular expressions used in programming languages, but it provides the placeholders needed for dialplan number matching.

---

# The Golden Rule: The Underscore (`_`)

Every extension pattern that uses placeholders must begin with an underscore:

```text
_
```

The underscore tells Asterisk that the extension contains pattern-matching symbols instead of being a literal number or name.

Example:

```ini
exten => _XXXX,1,NoOp(Four-digit number detected)
```

Without the underscore, Asterisk would treat `XXXX` as a literal extension name.

---

# Important Pattern Symbols

| Symbol | Meaning |
|---|---|
| `X` | Any single digit from `0` to `9` |
| `.` | One or more additional characters |
| `N` | Any single digit from `2` to `9` |
| `Z` | Any single digit from `1` to `9` |
| `[123]` | One digit from the specified set: `1`, `2`, or `3` |
| `[3-5]` | One digit from the specified range: `3`, `4`, or `5` |

> **Important correction:** In Asterisk pattern matching, the dot does not repeat the previous character. It matches one or more additional characters.

---

# Step-by-Step Pattern Building

## 1. Matching an Exact Length

To match any four-digit number:

```ini
exten => _XXXX,1,Goto(outgoing,8888,1)
```

### Explanation

```text
_
```

Marks the extension as a pattern.

```text
XXXX
```

Matches exactly four digits, where each `X` can be any digit from `0` to `9`.

### Matches

```text
0000
1234
8888
9999
```

### Does Not Match

```text
123
12345
```

The first number is too short, while the second is too long.

---

## 2. Matching Numbers of Different Lengths

To match a number beginning with any digit and containing at least one additional character:

```ini
exten => _X.,1,NoOp(Call Detected)
```

### Explanation

```text
X
```

Matches the first digit from `0` to `9`.

```text
.
```

Matches one or more additional characters.

### Matches

```text
12
123
1234567890
```

### Does Not Match

```text
1
```

Because the dot requires at least one additional character after the first digit.

---

## 3. Matching Numbers with a Zero Prefix

Many telephone systems require users to dial:

```text
0
```

before an outside number.

The following pattern:

```ini
exten => _0.,1,NoOp(Outside Call)
```

means:

```text
A literal 0 followed by one or more additional characters.
```

### Matches

```text
01
0123
08888
0012345
```

### Does Not Match

```text
0
```

because at least one additional character is required after the zero.

---

## 4. Zero Followed by Digits Only

To match a zero followed by at least two digits:

```ini
exten => _0X.,1,Goto(outgoing,8888,1)
```

### Explanation

```text
0
```

Matches the literal outside-line prefix.

```text
X
```

Matches the first digit after the prefix.

```text
.
```

Matches one or more additional characters.

### Matches

```text
012
0123456789
08888
```

### Does Not Match

```text
0
01
1111
```

The first two values are too short, while `1111` does not begin with zero.

---

# Testing the Patterns

After editing:

```text
/etc/asterisk/extensions.conf
```

reload the dialplan:

```asterisk
dialplan reload
```

---

## Testing `_XXXX`

Using:

```ini
exten => _XXXX,1,Goto(outgoing,8888,1)
```

Dial:

```text
8888
```

The pattern matches.

Dial:

```text
1111
```

This also matches because it contains exactly four digits.

However, both numbers are sent to the same hardcoded destination:

```text
outgoing,8888,1
```

---

## Testing `_0X.`

Using:

```ini
exten => _0X.,1,Goto(outgoing,8888,1)
```

Dial:

```text
1111
```

The call is rejected because the number does not begin with zero.

Dial:

```text
08888
```

The pattern matches because it begins with zero and contains enough additional digits.

---

# The Remaining Problem

Pattern matching allows one dialplan rule to recognize many different numbers.

However, this rule still contains a hardcoded destination:

```ini
Goto(outgoing,8888,1)
```

If the caller dials:

```text
0123
```

Asterisk still sends the call to:

```text
8888
```

This is not the required behavior.

The next concept is dialplan variables, which allow Asterisk to capture the number that was actually dialed and pass it dynamically to the outbound route or provider.
