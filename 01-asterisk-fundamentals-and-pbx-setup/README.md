# 📞 01 — Asterisk Fundamentals & PBX Setup

> Part of the [**Asterisk Telephony Notes**](https://github.com/saadcnx/asterisk-telephony-notes) series — a hands-on, from-scratch guide to building, understanding, and running an Asterisk PBX.

This module takes you from zero to a fully working Asterisk PBX: installing and compiling Asterisk, understanding its core architecture, configuring SIP endpoints, writing your first dialplan, and handling real incoming calls.

---

## 📚 What You'll Learn

By the end of this module, you'll be able to:

- Understand what Asterisk is and choose the right version for your setup
- Compile, install, and securely configure Asterisk from source
- Understand Asterisk's file system, core concepts, and networking requirements
- Configure `sip.conf` and add SIP peers / softphones
- Write dialplans using contexts, extensions, and applications
- Play audio files and handle incoming calls end-to-end

---

## 🗂️ Tutorials in This Module

| # | Tutorial | Description |
|---|----------|--------------|
| 01 | [Introduction & Version Selection](./1-asterisk-intro-and-version-selection.md) | What Asterisk is, its architecture, and how to pick the right version |
| 02 | [Installation & Compilation](./2-asterisk-installation-and-compilation.md) | Compiling and installing Asterisk from source |
| 03 | [`make menuselect` Modules Guide](./3-asterisk-make-menuselect-modules-guide.md) | Selecting and configuring build modules with `menuselect` |
| 04 | [Init Script & Security Setup](./4-asterisk-init-script-and-security-setup.md) | Setting up the init/service script and hardening your install |
| 05 | [File System & PBX Layout](./5-asterisk-file-system-and-pbx.md) | Understanding Asterisk's directory structure and config file layout |
| 06 | [Network Config: DHCP & NTP](./6-asterisk-network-config-dhcp-ntp.md) | Preparing your network — DHCP and NTP configuration for VoIP |
| 07 | [Core Concepts: Extensions, Peers, Trunks & Tunnels](./7-asterisk-core-concepts-extensions-peers-trunks-tunnels.md) | Key telephony concepts every Asterisk admin needs to know |
| 08 | [`sip.conf` — Pro-Level Deep Dive](./8-asterisk-sip-conf-pro-level-deep-dive.md) | Advanced, in-depth guide to configuring `sip.conf` |
| 09 | [Adding SIP Peers & Softphones](./9-asterisk-adding-sip-peers-and-softphones.md) | Registering SIP peers and connecting softphones to your PBX |
| 10 | [Dialplan Basics: Contexts & Extensions](./10-asterisk-dialplan-basics-contexts-extensions.md) | Introduction to `extensions.conf` — contexts and extension logic |
| 11 | [Dialplan Applications & Your First Call](./11-asterisk-dialplan-applications-and-first-call.md) | Using dialplan applications to complete your first test call |
| 12 | [Playback() & Audio Files](./12-asterisk-applications-playback-and-audio-files.md) | Playing audio prompts with the `Playback()` application |
| 13 | [Incoming Calls & Context Mapping](./13-asterisk-incoming-calls-context-mapping.md) | Routing and handling incoming calls via context mapping |

---

## 🧰 Prerequisites

- A Linux server or VM (Ubuntu/Debian/CentOS recommended)
- Basic familiarity with the Linux command line
- Root or sudo access
- Basic understanding of networking (IP, DNS, ports) is helpful but not required

---

## 🚀 Getting Started

Clone the repository and start with Tutorial 01:

```bash
git clone https://github.com/saadcnx/asterisk-telephony-notes.git
cd asterisk-telephony-notes/01-asterisk-fundamentals-and-pbx-setup
```

Then open `1-asterisk-intro-and-version-selection.md` and work through the tutorials in numeric order — each one builds directly on the previous, from installation all the way to handling live incoming calls.

---

## 🗺️ Series Roadmap

This is Module 1 of the **Asterisk Telephony Notes** series. Planned future modules include:

- Voicemail, IVR menus, and call queues
- Trunking and PSTN/SIP provider integration
- AMI (Asterisk Manager Interface) & AGI scripting
- Advanced call routing and failover strategies

---

## 🤝 Contributing

Found an issue, typo, or have a suggestion? Feel free to open an [issue](https://github.com/saadcnx/asterisk-telephony-notes/issues) or submit a pull request.

## 📄 License

This project is shared for educational purposes. Check the main repository for license details.

---

<p align="center">Made with 📞 and ☕ by <a href="https://github.com/saadcnx">saadcnx</a></p>
