
# ☎️ 02 — Dialplan, Voicemail & Call Routing

> Part of the [**Asterisk Telephony Notes**](https://github.com/saadcnx/asterisk-telephony-notes) series — a hands-on, from-scratch guide to building, understanding, and running an Asterisk PBX.

This module builds on the fundamentals from [Module 01](../01-asterisk-fundamentals-and-pbx-setup) and dives deeper into advanced dialplan logic, voicemail configuration, and call distribution — the core building blocks of a real-world PBX.

---

## 📚 What You'll Learn

By the end of this module, you'll be able to:

- Structure outbound call contexts following best practices
- Use pattern matching and regular expressions in the dialplan
- Work with dialplan variables and string manipulation
- Build time-based call routing conditions with `GotoIfTime`
- Use subroutines with `Gosub`/`Return` for reusable dialplan logic
- Configure and integrate voicemail into your dialplan
- Set up busy/unavailable greetings for extensions
- Understand the basics (and limitations) of call distribution

---

## 🗂️ Tutorials in This Module

| # | Tutorial | Description |
|---|----------|--------------|
| 01 | [Outbound Calls & Context Best Practices](./01-asterisk-outbound-calls-context-best-practices.md) | Structuring outbound call contexts the right way |
| 02 | [Dialplan Pattern Matching & Regular Expressions](./02-asterisk-dialplan-pattern-matching-regular-expressions.md) | Using pattern matching and regex for flexible extension logic |
| 03 | [Variables & String Manipulation](./03-asterisk-variables-and-string-manipulation.md) | Working with dialplan variables and manipulating strings |
| 04 | [Time-Based Conditions — `GotoIfTime`](./04-asterisk-time-based-conditions-gotoiftime.md) | Routing calls based on time of day/week with `GotoIfTime` |
| 05 | [Subroutines — `Gosub`/`Return`](./05-asterisk-subroutines-gosub-return.md) | Writing reusable dialplan logic with subroutines |
| 06 | [Voicemail Configuration Basics](./06-asterisk-voicemail-configuration-basics.md) | Setting up `voicemail.conf` and mailboxes from scratch |
| 07 | [Voicemail Dialplan Integration](./07-asterisk-voicemail-dialplan-integration.md) | Wiring voicemail into your dialplan for unanswered calls |
| 08 | [Voicemail Busy/Unavailable Greetings](./08-asterisk-voicemail-busy-unavailable-greetings.md) | Configuring custom busy and unavailable voicemail greetings |
| 08 | [Extension File — Busy/Unavailable Greetings](./08-extension-file-busy-unavailable-greetings.md) | Applying busy/unavailable greeting logic directly in `extensions.conf` |
| 09 | [Call Distribution — Basics & Drawbacks](./09-asterisk-call-distribution-basics-drawbacks.md) | Introduction to call distribution strategies and their trade-offs |

> 📝 **Note:** There are two `08` tutorials in this module (voicemail-focused and extensions-file-focused) — both are included above since they cover related but distinct aspects of busy/unavailable greeting handling.

---

## 🧰 Prerequisites

- Completion of [Module 01 — Asterisk Fundamentals & PBX Setup](../01-asterisk-fundamentals-and-pbx-setup)
- A working Asterisk installation with at least one registered SIP peer
- Basic familiarity with `extensions.conf` and dialplan syntax

---

## 🚀 Getting Started

Clone the repository and navigate to this module:

```bash
git clone https://github.com/saadcnx/asterisk-telephony-notes.git
cd asterisk-telephony-notes/02-dialplan-voicemail-and-call-routing
```

Work through the tutorials in numeric order — starting with outbound call contexts and ending with call distribution — for the most logical learning path.

---

## 🗺️ Series Roadmap

This is Module 2 of the **Asterisk Telephony Notes** series. Planned future modules include:

- IVR menus and advanced call queues
- Trunking and PSTN/SIP provider integration
- AMI (Asterisk Manager Interface) & AGI scripting
- Advanced call routing and failover strategies

---

## 🤝 Contributing

Found an issue, typo, or have a suggestion? Feel free to open an [issue](https://github.com/saadcnx/asterisk-telephony-notes/issues) or submit a pull request.

## 📄 License

This project is shared for educational purposes. Check the main repository for license details.

---

<p align="center">Made with ☎️ and ☕ by <a href="https://github.com/saadcnx">saadcnx</a></p>
