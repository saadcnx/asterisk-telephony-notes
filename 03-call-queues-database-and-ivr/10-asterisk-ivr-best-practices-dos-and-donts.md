# IVR Best Practices: Do's and Don'ts

## What Is an IVR?

IVR stands for:

```text
Interactive Voice Response
```

It is the menu system that asks callers to choose options such as:

```text
Press 1 for Support
Press 2 for Sales
Press 0 for an Operator
```

Before building an IVR, it is important to understand when it is useful and when it should be avoided.

---

# Rule 1: Avoid IVR Menus When Possible

The best caller experience is usually to speak with a human as quickly as possible.

If there is no clear reason to use a menu, do not build one.

## Sales Calls

Someone calling a sales department usually wants to buy something.

A better experience is:

```text
Connect the caller directly to a salesperson.
```

The caller should not have to listen to a long greeting or navigate several options before reaching sales.

## High-End Services

Examples include:

- Prestige hotels
- Luxury services
- High-touch customer support
- Premium concierge services

Callers in these situations often expect to speak directly with a person.

They do not want a machine to replace personal service.

## Golden Rule

```text
If there is no clear reason for an IVR, do not use one.
```

---

# Rule 2: Valid Reasons to Use an IVR

## 1. Language Selection

Language selection is a valid IVR use case.

Example:

```text
Press 1 for German
Press 2 for English
Press 3 for French
```

Caller ID cannot always determine the correct language because:

- People travel
- Phone numbers may not match the caller's location
- Some countries use multiple languages
- Customers may prefer a language different from the local language

A caller who cannot understand the menu may simply hang up.

---

## 2. Customer Convenience During Long Waits

An IVR can give callers more control when support queues are busy.

Example:

```text
Press 1 to continue holding
Press 2 to leave a voicemail
Press 3 to request a callback
```

This allows the caller to choose what happens next instead of being forced to wait.

---

# Rule 3: Common IVR Design Mistakes

## 1. Long Greetings Before the Menu

Bad example:

```text
Hello and welcome to our company.
We are the greatest company in the world...
For Sales, press 1.
```

Do not make callers listen to marketing language before hearing the available options.

A better approach is to get to the point quickly.

---

## 2. Asking for Information After Routing

Bad flow:

```text
Caller presses 1 for Sales
        ↓
System asks for customer number
```

If information is required, collect it before routing the caller.

Better flow:

```text
Collect customer information
        ↓
Send the caller to the correct department
```

Do not make callers navigate a menu and then ask them for additional information unnecessarily.

---

## 3. Too Many Options

Callers can remember only a limited number of choices at one time.

Avoid long menus such as:

```text
Press 1...
Press 2...
Press 3...
Press 4...
Press 5...
Press 6...
Press 7...
Press 8...
```

Keep options:

- Short
- Clear
- Easy to remember
- Focused on the caller's main needs

---

## 4. No Option to Reach a Human

Always provide a path to a real person.

Example:

```text
Press 0 to speak to an operator.
```

Do not trap callers inside automated menus.

---

# Summary: Do's and Don'ts

| Don't | Do |
|---|---|
| Use an IVR when a person can answer directly | Use an IVR for real needs such as language selection |
| Put a long greeting before the options | Present the choices quickly |
| Ask for information after routing | Collect required information before routing |
| Provide too many menu options | Keep menus short and clear |
| Trap callers inside the menu | Provide a way to reach a human |
| Use IVR for high-touch services | Let people handle calls where personal service matters |

---

# Practical Design Principles

A good IVR should:

- Solve a real problem
- Save the caller time
- Offer clear choices
- Avoid unnecessary marketing
- Collect information in the correct order
- Provide an escape to a human
- Keep the menu short

A bad IVR usually:

- Delays the caller
- Repeats unnecessary information
- Uses too many options
- Makes the caller enter information multiple times
- Prevents access to a real person

---

# Additional Resource

The tutorial also references a blog post and infographic based on research from Software Advice in the United States.

It covers topics such as:

- The ideal number of menu options
- What questions to ask
- How to design self-service flows
- How callers respond to different IVR structures

---

# Next Step

The next tutorial will build an IVR directly in the Asterisk dialplan.
