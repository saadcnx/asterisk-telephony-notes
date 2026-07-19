# Introduction to Asterisk & Version Selection

## What is Asterisk?
Asterisk is a server that manages communication between VoIP phones (like Aastra or Yealink).

- If Phone A wants to call Phone B, the server identifies where each phone is located and connects them.
- It uses the **SIP protocol** to route calls. You don't need to know SIP in full detail, but you should know what it is.

---

## Why Asterisk vs Other Solutions?

### Alternatives
Other SIP routers/projects exist (e.g., **Kamailio** – a very fast SIP router, **FreeSWITCH**).

### Main Advantage
Asterisk has a **huge feature set**.

### Feature-Rich System
It doesn't just route calls; it provides: -
- Voicemail boxes
- Call queues (where callers wait and are dispatched to different phones)

### Replacement for PBX
Asterisk can completely replace a traditional PBX system (like your standard PSTN/landline PBX).

### Scalability Comparison
Projects like **Kamailio** can handle thousands or hundreds of thousands of calls per instance and scale better, which is needed for large enterprises or Session Border Controllers. However, Asterisk's feature set makes it a good choice if you are new to VoIP.

---

## Asterisk Versions Explained
You should understand the available versions to choose the right one for your case.

| Version | Purpose | Recommendation |
|---------|---------|----------------|
| **Asterisk 11 (LTS)** | Long-Term Support version. | Recommended for beginners. This is the stable version you get from the main download button. |
| **Asterisk 12** | Under heavy development. | Used by developers to test new features. It is not stable. |
| **Asterisk 13** | The future state. | Once features in v12 are stable and approved by the community, they go into the next LTS version (which will be v13). |

### Certified vs. Normal LTS

- **Certified:** Even more stable than the standard LTS. Only bug fixes are added very carefully. This is a "quality release."
  - **Use Case:** If you want to go productive and need the most stable version, use the Certified Asterisk LTS.

- **For Testing:** If you are trying Asterisk for the first time, just use the standard Asterisk 11 LTS.

---

## Installation Method

### Source Code (Recommended)
- Download the source, unpack it, and compile it yourself.
- The magic of Open Source is that you can change the source code for a customizable solution.

### Pre-compiled Packages
- Distributions like Debian or Ubuntu offer packages, but they are often not the newest version.

---

## Setup Demo (First Step)

### Environment
- **OS:** Ubuntu LTS 14.04
- **Installation Type:** Fresh install with only SSH server selected.

### Process
1. Get root access.
2. Download the source by copying the link for the "Asterisk 11 current version".
3. **Next steps (to be covered later):**
   - Unpack the source
   - Configure
   - Compile
