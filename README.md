# RefugeOS

> **If you are reading this, the world may already be in hard times. God bless you. May these tools help you endure with wisdom, discipline, faith, and hope. By the time you find this, I may not be around anymore — or maybe I am, just not near you. Either way, I did not want you to be left empty-handed.**

**RefugeOS** is an **Arch-based, Bible-centered, offline-first live operating system** built for crisis readiness, field use, recovery work, and low-friction deployment on x86_64 computers.

It is designed to be carried on a USB drive, booted as a live system, installed onto another machine if needed, or tested inside a virtual machine. The goal is simple: **when the network is weak, the power is unstable, or the world is in chaos, your library, tools, notes, browser, and basic working environment should still be with you.**

---

## Why this project exists

RefugeOS was built around a few practical needs:

- a **portable live system** that can boot on modern x86_64 hardware
- a **very lightweight base** that can scale from a smaller `lite` build to a fuller `full` build
- a **Bible and offline-reference layer** that still works with no connection
- a **normal desktop workflow** with browser, documents, notes, media, and file tools
- an **Arch-native build system** that is easier to maintain on Arch and SteamOS-style environments
- optional **security and recovery modules** that can be included only when they are actually needed

This repository contains the **source tree for building the ISO**, not just a wallpapered Linux remaster. It is meant to be practical, rebuildable, and understandable.

---

## Core idea

RefugeOS is built around three layers:

### 1. Survival and continuity
A bootable environment with browser access, file tools, notes, media playback, PDFs, offline reading, and document editing.

### 2. Faith and reference
Bible access through **Xiphos + SWORD**, a preloaded public-domain Bible PDF, and room for offline **Kiwix** libraries, maps, manuals, and local notes.

### 3. Technical resilience
Optional recovery, network, and security tools for troubleshooting, diagnosis, defensive assessment, and field repair.

---

## What is included

### Base environment
- lightweight X11 desktop session
- Openbox / LXDE-style workflow
- Firefox wrapped as **Refuge Browser**
- NetworkManager applet for network access
- low-memory helpers such as zram and earlyoom
- file manager, terminal, archive tools, calculator, editor, and monitoring utilities

### Reading and knowledge tools
- **Xiphos**
- **SWORD**
- **Kiwix Desktop**
- **MuPDF**
- a bundled public-domain **World English Bible** PDF
- organized `RefugeLibrary` folders for Kiwix ZIM files, maps, notes, manuals, videos, and backups

### Daily work tools
- **FeatherPad**
- **Gnumeric**
- **LibreOffice Still** in the `full` profile
- **MPV / FFmpeg**
- **Newsboat**
- **Zim** desktop wiki for field notes and local documentation
- **Qalculate** for a full calculator UI

### Mapping and field tools
- **Marble**
- **Marble Maps**
- **QMapShack** in the `full` profile
- **GParted** in the `full` profile

### Optional security modules
RefugeOS keeps heavier security tooling modular on purpose.

- `security-mini` → `nmap`, `zenmap`, `tcpdump`, `wireshark-cli`
- `security-wireless` → `aircrack-ng`, `mdk4`
- `security-heavy` → `hydra`, `john`, `hashcat`, `sqlmap`, `nikto`
- `forensics` → `sleuthkit`, `testdisk`, `binwalk`

---

## Profiles

### `lite`
The smallest practical build in this project.

Best for:
- older or weaker x86_64 systems
- emergency live USB use
- basic browsing, reading, file work, and offline reference

### `full`
Adds heavier daily-driver tools.

Best for:
- stronger laptops/desktops
- VM testing
- document work
- maps, office suite, partitioning, and a more complete recovery environment

---

## Build requirements

Use an Arch-based host with:

```bash
sudo pacman -S --needed archiso rsync
```

Recommended extras for testing:

```bash
sudo pacman -S --needed edk2-ovmf qemu
```

---

## Build examples

```bash
# inspect the host first
./build.sh doctor

# smallest normal build
sudo ./build.sh lite

# fuller everyday build
sudo ./build.sh full

# add a light security set
sudo ./build.sh full security-mini security-wireless

# maximum included modules from this repository
sudo ./build.sh full security-mini security-wireless security-heavy forensics
```

The finished ISO is written to:

```text
out/
```

---

## Boot mode default

This builder defaults to:

- **BIOS boot**
- **x86_64 UEFI boot**

IA32 UEFI is disabled by default because it causes unnecessary host-side GRUB problems on most modern systems.

If you ever need rare 32-bit UEFI support:

```bash
REFUGE_INCLUDE_IA32=1 sudo ./build.sh full
```

---

## Live system layout

Inside the live system, the main working tree is:

```text
/home/refuge/RefugeLibrary/
  Bible/
  Kiwix/
  Manuals/
  Maps/
  Medical/
  Radio/
  Local/
  Videos/
  Notes/
  Backups/
  Personal-Encrypted/
```

The intended workflow is simple:

- boot
- connect if a network exists
- read offline if it does not
- keep notes
- open manuals
- use the browser when available
- move important files into the library tree

---

## Updates, mirrors, and preloading packages

RefugeOS can be used in three different ways:

### 1. Installed RefugeOS
If RefugeOS is installed to an SSD, HDD, or USB as a real Arch system, package installs and updates work normally with `pacman`.

### 2. Live USB without persistence
If RefugeOS is only being used as a normal live USB, packages can still be installed during that session, but those changes are temporary and should not be expected to survive a reboot.

### 3. Live USB with persistence
A persistent live setup can keep changes across reboots. RefugeOS is still built with `archiso`, so future persistence workflows can be based on Archiso persistence parameters such as `cow_label`, `cow_device`, and `cow_directory`.

### Default mirror setup

A simple default mirror list is enough for most users:

```bash
sudo tee /etc/pacman.d/mirrorlist >/dev/null <<'EOF'
Server = https://geo.mirror.pkgbuild.com/$repo/os/$arch
EOF
```

Then refresh and update:

```bash
sudo pacman -Syyu
```

To install more packages later:

```bash
sudo pacman -S firefox libreoffice-still mpv kiwix-desktop xiphos
```

### Optional automatic mirror refresh

If you want a better local mirror list, install `reflector` and let it refresh `/etc/pacman.d/mirrorlist` automatically:

```bash
sudo pacman -S reflector
sudo reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
sudo systemctl enable --now reflector.timer
```

### Pre-downloading packages before hard times

You can prepare ahead of time by downloading packages now without installing them yet:

```bash
sudo mkdir -p /srv/refuge-cache
sudo pacman -Sw --cachedir /srv/refuge-cache \
  firefox libreoffice-still mpv kiwix-desktop xiphos \
  gparted qmapshack marble newsboat
```

Later, you can tell `pacman` to reuse that cache by adding another cache directory under `[options]` in `/etc/pacman.conf`:

```ini
CacheDir = /var/cache/pacman/pkg
CacheDir = /srv/refuge-cache
```

That lets you stockpile useful packages ahead of time on an internal drive, SSD, or large USB device.

### Recommendation

For this project, the safest default is to rely mostly on the official Arch repositories and keep the base system simple and reproducible. If a user wants more tools before things become worse, they can update the mirror list, refresh the databases, and pre-download packages now while the network is still available.

---

## Virtual machine use

RefugeOS can be tested in VirtualBox or QEMU before writing it to USB.

For VirtualBox, the safest first test is:

- Linux guest
- **VMSVGA** graphics controller
- **3D acceleration off** for the first boot
- **128 MB** video memory
- at least **2–4 GB RAM** for `full`

If the VM shows only a black background **but the panel, tray, clock, and launchers are visible**, the desktop session has usually booted and the issue is visual rather than a total boot failure.

For real validation, test both:

1. in a VM
2. on a real USB boot on another machine

A live USB test matters because real hardware, Wi-Fi, storage controllers, and graphics behavior can differ from a virtual machine.

---

## Steam Deck / SteamOS note

This project exists partly because SteamOS is Arch-based and building with `archiso` is much more natural in that ecosystem than trying to force a Debian live-build workflow onto the Deck.

That said, SteamOS itself is not a normal always-writable Arch install. For repeatable builds, a dedicated Arch PC, Arch VM, or external Arch install is usually the cleanest option.

---

## Responsible-use note

RefugeOS may include security and audit tools in optional modules. They are provided for:

- lawful defensive testing
- lab work
- recovery
- troubleshooting
- education
- personal infrastructure hardening

Do not use this project to attack systems you do not own or have explicit permission to assess.

---

## Project philosophy

RefugeOS is not trying to be flashy.

It is trying to be:

- understandable
- portable
- rebuildable
- useful when the network is down
- useful when you are tired
- useful when the machine is weak
- useful when you need both Scripture and practical tools in one place

If you are here because times are hard, may this system serve you well.

---

## Roadmap

Planned improvements:

- cleaner first-boot desktop polish
- optional wallpaper/theme pack
- better VirtualBox guest behavior
- optional persistence workflow for external SSD installs
- a simpler installer path for turning the live environment into a permanent system
- larger offline documentation bundles
- better hardware-specific testing notes

---

## Repository structure

```text
build.sh                     build helper
packages/                    package manifests
modules/                     optional add-on manifests
overlays/                    files copied into the live filesystem
docs/                        notes, plans, and troubleshooting
scripts/                     helper scripts
```

---

## License

This repository is released under the **MIT License** for the original project files in this repository.

Included third-party packages, fonts, applications, and modules keep their own licenses.

---

## Final note

If you found this repository when the world is breaking apart, remember this:

**do not panic, do not waste power, do not waste storage, do not waste time. Keep what matters. Keep faith. Keep records. Keep moving.**
