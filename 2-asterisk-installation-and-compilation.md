# Asterisk Installation & Compilation

## Pre-Requisites (Vanilla Ubuntu)

Before compiling, you need to set up your environment. The assumption here is a fresh, vanilla Ubuntu installation. You must be **root** to avoid write permission problems.

### 1. Install Essential Packages

First, install the core compiling tools and specific libraries needed by Asterisk:

- **build-essential:** Contains the necessary tools, such as GCC and `make`, for compiling code from source.
- **SSL development libraries:** Needed to compile SSL-related features.
- **ncurses development libraries:** Needed to display terminal-based menus, such as the module selection interface.
- **Linux headers:** Required to compile driver-related components.

### Trick for Linux Headers

You must install headers that exactly match your current kernel version. Instead of looking up the kernel version manually, use:

```bash
apt-get install linux-headers-$(uname -r)
```

> `$(uname -r)` runs the `uname -r` command and inserts the current kernel version automatically.

---

## The Compilation Process

Once the required packages are installed, navigate to the downloaded Asterisk source code directory.

### 1. Unpack

Extract the Asterisk source code archive that you downloaded previously.

### 2. Configure

Run the configuration script:

```bash
./configure
```

This standard step checks your environment to confirm that all required libraries are present and the system is ready to compile.

- If it succeeds, you should see the Asterisk ASCII-art logo.
- If it fails, the output will usually identify the missing package. Install that package and run `./configure` again.

### 3. Make Menuconfig (Optional)

Run:

```bash
make menuconfig
```

This opens a menu interface, similar to the Linux kernel configuration menu, where you can enable or disable specific Asterisk features and modules.

#### Why Use It?

- **Stability:** Removing unnecessary modules may improve stability by reducing the number of components that can fail.
- **Experimental or risky features:** Some experimental or potentially unsafe features are disabled by default. Enable them only when required.

> **Beginner recommendation:** Skip this step during your first installation. The default configuration already includes most commonly required modules. `make menuconfig` is mainly useful for advanced installations or special requirements.

### 4. Compile Asterisk

Run:

```bash
make
```

This compiles the Asterisk server according to the selected configuration.

> Compilation commonly takes several minutes, depending on the system.

### Additional Note on Hardware

Digium, one of the original developers of Asterisk, provides telephony hardware commonly used for connecting analog landlines.

- If you use compatible telephony hardware, you may need to compile the required hardware drivers.
- In a SIP-only environment, where Asterisk connects to a SIP carrier, trunk, or gateway, hardware-specific drivers are generally unnecessary.

---

## Installation & Initial Setup

### 1. Install Asterisk

After compilation, Asterisk exists only inside the source directory. Run:

```bash
make install
```

This copies the compiled binaries and supporting files into the appropriate Linux system directories.

### 2. Install Sample Configuration Files

Asterisk configuration files are stored in:

```text
/etc/asterisk
```

After a fresh source installation, this directory may be empty.

Instead of manually creating more than 100 configuration files, run:

```bash
make samples
```

This creates sample configuration files so that Asterisk can start.

> **Security warning:** Do not use the sample configuration unchanged in a production environment. It may contain guest access and enabled features that should be disabled or hardened before deployment.

---

## Starting Asterisk Manually

Start Asterisk in console mode with:

```bash
asterisk -cvvv
```

### Command Options

- `-c`: Runs Asterisk in console mode and displays server activity.
- `-vvv`: Sets verbosity to level 3. Adding more `v` characters increases the verbosity level.

Once Asterisk starts, it loads its modules and opens the Asterisk command-line interface.

To check registered SIP phones or peers, run:

```asterisk
sip show peers
```

The list will be empty until SIP endpoints are configured and registered.

## Exiting Asterisk

Pressing `Ctrl+C` stops the complete Asterisk process.

> This is acceptable for testing, but it is not the recommended way to operate Asterisk on a production server. In production, Asterisk should normally run as a managed system service.
