
# 🗣️ 03 — Call Queues, Database & IVR

> Part of the [**Asterisk Telephony Notes**](https://github.com/saadcnx/asterisk-telephony-notes) series — a hands-on, from-scratch guide to building, understanding, and running an Asterisk PBX.

This module builds on [Module 01](../01-asterisk-fundamentals-and-pbx-setup) and [Module 02](../02-dialplan-voicemail-and-call-routing) and covers call center–style features: queues, agent management, Music on Hold, the Asterisk database (AstDB), custom prompt recording, and IVR design best practices.

---

## 📚 What You'll Learn

By the end of this module, you'll be able to:

- Set up basic call queues and configure agents
- Choose the right queue strategy and configure timeouts
- Add and manage dynamic queue members
- Configure Music on Hold (MoH) for waiting callers
- Implement dynamic queue login/logout for agents
- Use the Asterisk database (AstDB) and its core functions
- Build a single-extension toggle for quick agent login/logout
- Record and produce custom, production-quality voice prompts
- Export and integrate audio files into your dialplan
- Apply IVR design best practices — and know what to avoid

---

## 🗂️ Tutorials in This Module

| # | Tutorial | Description |
|---|----------|--------------|
| 01 | [Queues — Basic Setup](./01-asterisk-queues-basic-setup.md) | Setting up your first call queue in `queues.conf` |
| 02 | [Queue Strategies & Timeouts](./02-asterisk-queue-strategies-and-timeouts.md) | Choosing ring strategies and configuring timeout behavior |
| 03 | [Queue Dynamic Members](./03-asterisk-queue-dynamic-members.md) | Adding and removing queue members dynamically |
| 04 | [Music on Hold Configuration](./04-asterisk-music-on-hold-configuration.md) | Configuring MoH classes for callers waiting in queue |
| 05 | [Dynamic Queue Login/Logout — Basics](./05-asterisk-dynamic-queue-login-logout-basic.md) | Letting agents log in/out of queues dynamically |
| 06 | [Database Basics & Functions](./06-asterisk-database-basics-and-functions.md) | Using the Asterisk database (AstDB) and its core functions |
| 07 | [Dynamic Queue — Single Extension Toggle](./07-asterisk-dynamic-queue-single-extension-toggle.md) | Building a single extension to toggle agent login/logout |
| 08 | [Custom Prompt Recording — Production](./08-asterisk-custom-prompt-recording-production.md) | Recording production-quality custom voice prompts |
| 09 | [Audio Export & Integration](./09-asterisk-audio-export-and-integration.md) | Exporting and integrating audio files into your dialplan |
| 10 | [IVR Best Practices — Do's and Don'ts](./10-asterisk-ivr-best-practices-dos-and-donts.md) | Designing effective IVR menus and avoiding common pitfalls |

---

## 🧰 Prerequisites

- Completion of [Module 01 — Asterisk Fundamentals & PBX Setup](../01-asterisk-fundamentals-and-pbx-setup)
- Completion of [Module 02 — Dialplan, Voicemail & Call Routing](../02-dialplan-voicemail-and-call-routing)
- A working Asterisk installation with at least one registered SIP peer
- Basic familiarity with `extensions.conf` and dialplan syntax

---

## 🚀 Getting Started

Clone the repository and navigate to this module:

```bash
git clone https://github.com/saadcnx/asterisk-telephony-notes.git
cd asterisk-telephony-notes/03-call-queues-database-and-ivr
```

Work through the tutorials in numeric order — starting with basic queue setup and ending with IVR design best practices — for the most logical learning path.

---

## 🗺️ Series Roadmap

This is Module 3 of the **Asterisk Telephony Notes** series. Planned future modules include:

- Trunking and PSTN/SIP provider integration
- AMI (Asterisk Manager Interface) & AGI scripting
- Advanced call routing and failover strategies

---

## 🤝 Contributing

Found an issue, typo, or have a suggestion? Feel free to open an [issue](https://github.com/saadcnx/asterisk-telephony-notes/issues) or submit a pull request.

## 📄 License

This project is shared for educational purposes. Check the main repository for license details.

---

<p align="center">Made with 🗣️ and ☕ by <a href="https://github.com/saadcnx">saadcnx</a></p>
