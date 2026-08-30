<p align="center">
  <img src="calamares/branding/macnix/macnix_logo.png" alt="MacNix Logo" width="180"/>
</p>

<h1 align="center">MacNix</h1>

<p align="center">
  <b>Run macOS with near-native performance. One USB drive. Zero configuration.</b>
</p>

<p align="center">
  <a href="#-installation"><img src="https://img.shields.io/badge/install-guide-blue?style=for-the-badge" alt="Install Guide"/></a>
  <a href="#-gpu-compatibility"><img src="https://img.shields.io/badge/GPU-compatibility-green?style=for-the-badge" alt="GPU Compatibility"/></a>
  <a href="https://github.com/local-over/macnix/actions"><img src="https://img.shields.io/github/actions/workflow/status/local-over/macnix/build-iso.yml?style=for-the-badge&label=ISO%20Build" alt="Build Status"/></a>
</p>

---
##NOTE: The project is currently not working and will NOT be completed any soon for some personl problems 
## What is MacNix?

**MacNix** is a purpose-built Linux distribution that runs macOS as a virtual machine with near-native performance — including full GPU acceleration.

There is no dual-booting. There is no Hackintosh. The host Linux system is completely invisible. You power on your PC, you see the Apple logo, and you use macOS. That's it.

Under the hood, MacNix is a minimal Debian system that acts as a hypervisor. It automatically configures QEMU/KVM, passes your physical GPU directly to the macOS VM via VFIO, and boots OpenCore — all without you touching a single config file.

## How It Works

```
┌──────────────────────────────────────────────────┐
│                    Your Monitor                  │
│               (macOS on bare metal)              │
└──────────────────────┬───────────────────────────┘
                       │ GPU output
┌──────────────────────┴───────────────────────────┐
│              macOS Virtual Machine               │
│     ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│     │ OpenCore │  │ GPU Pass │  │ USB Pass │    │
│     │ Bootldr  │  │ through  │  │ through  │    │
│     └──────────┘  └──────────┘  └──────────┘    │
├──────────────────────────────────────────────────┤
│               QEMU/KVM Hypervisor                │
├──────────────────────────────────────────────────┤
│            MacNix (Debian Minimal)               │
│     VFIO · Hugepages · CPU Pinning · evdev       │
├──────────────────────────────────────────────────┤
│                   Hardware                       │
│     CPU (VT-x/AMD-V) · GPU · RAM · NVMe         │
└──────────────────────────────────────────────────┘
```

**Key technologies:**
- **KVM** — hardware-accelerated virtualization (near-native CPU performance)
- **VFIO/GPU Passthrough** — your physical GPU is detached from Linux and given directly to macOS
- **OpenCore** — the same bootloader used by the Hackintosh community, auto-configured for your hardware
- **Hugepages + CPU Pinning** — dedicated memory and CPU cores for macOS, eliminating jitter
- **evdev** — your physical keyboard and mouse are passed directly to the VM

## ✅ Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | Intel VT-x or AMD-V with **IOMMU** (VT-d / AMD-Vi) | AMD Ryzen 5000+ or Intel 10th Gen+ |
| **RAM** | 16 GB | 32 GB+ |
| **Disk** | 150 GB free (SSD) | 500 GB+ NVMe |
| **GPU** | Any AMD or NVIDIA (see table below) | AMD RX 6000/7000 series |
| **Motherboard** | IOMMU support in BIOS | Proper IOMMU grouping |

> [!IMPORTANT]
> **IOMMU must be enabled in your BIOS/UEFI.** This is usually listed as "VT-d" (Intel) or "AMD-Vi / IOMMU" (AMD). Without it, GPU passthrough will not work.

## 🎮 GPU Compatibility

| GPU Family | macOS Support | Passthrough | Notes |
|------------|:------------:|:-----------:|-------|
| **AMD RX 7000** (RDNA 3) | Sonoma 14.4+ | ✅ | Best option. Native support in latest macOS |
| **AMD RX 6000** (RDNA 2) | Monterey+ | ✅ | Excellent. Fully supported |
| **AMD RX 5000** (RDNA 1) | Catalina+ | ✅ | Works well, widely tested |
| **AMD RX 500** (Polaris) | Sierra+ | ✅ | Great budget option, rock-solid |
| **AMD Vega** | High Sierra+ | ✅ | Well supported |
| **NVIDIA Kepler** (GTX 600/700) | Catalina and older | ✅ | Last NVIDIA gen with native macOS drivers |
| **NVIDIA Maxwell/Pascal+** | ❌ None | ✅ | Passthrough works, but macOS has **no drivers** |
| **Intel iGPU** | Varies | ⚠️ GVT-g | Experimental. Not recommended for primary GPU |

> [!TIP]
> **Best experience:** AMD RX 6600 or newer. They're affordable, have excellent macOS support, and work flawlessly with VFIO passthrough.

> [!WARNING]
> **NVIDIA GTX 900 series and newer** can be passed through to the VM, but macOS will not recognize them. Apple dropped NVIDIA support after Kepler. If NVIDIA is your only GPU, MacNix will still run macOS — but without GPU acceleration (software rendering only).

## 🍎 macOS Version Support

MacNix can download and install the following macOS versions during setup:

| Version | Codename | Status |
|---------|----------|--------|
| macOS 15 | Sequoia | ✅ Supported |
| macOS 14 | Sonoma | ✅ Supported |
| macOS 13 | Ventura | ✅ Supported |
| macOS 12 | Monterey | ✅ Supported |

OpenCore configuration is automatically generated based on your hardware and chosen macOS version.

## 📦 Installation

### Step 1: Create a Bootable USB

Download the latest ISO from [Releases](https://github.com/local-over/macnix/releases) or build it yourself (see [Building from Source](#-building-from-source)).

```bash
# Linux
sudo dd if=macnix-*.iso of=/dev/sdX bs=4M status=progress

# macOS
sudo dd if=macnix-*.iso of=/dev/diskN bs=4m

# Windows — use Rufus (https://rufus.ie) in DD mode
```

### Step 2: Boot and Install

1. **Boot from USB** — Enter your BIOS and select the MacNix USB drive
2. **Hardware Precheck** — MacNix automatically verifies your CPU, IOMMU, and GPU compatibility
3. **Follow the Installer** — The 4-step Calamares installer walks you through:
   - **Welcome** — Language and keyboard selection
   - **GPU Detection** — Automatic identification and VFIO configuration of your GPU
   - **macOS Download** — Select your macOS version; the recovery image is fetched automatically
   - **GPU Configuration** — OpenCore config is generated for your specific hardware
4. **Reboot** — Remove the USB drive and reboot. macOS will start automatically.

### Step 3: macOS Setup

On first boot after installation, macOS Setup Assistant will launch automatically. Complete the standard macOS setup (Apple ID, user account, etc.) and you're done.

> [!NOTE]
> The first boot takes longer than usual (5-10 minutes) while MacNix configures CPU pinning, hugepages, and VFIO bindings for your specific hardware.

## ⚡ What to Expect

| Metric | Performance |
|--------|-------------|
| **Boot to desktop** | ~30 seconds (after first-boot setup) |
| **CPU performance** | 95-100% of native (KVM hardware acceleration) |
| **GPU performance** | 98-100% of native (direct passthrough, no emulation) |
| **Storage I/O** | ~90% of native (virtio-blk with NVMe backend) |
| **USB devices** | Full speed (direct passthrough) |

## 🔐 Fingerprint Setup

MacNix can bridge your Linux fingerprint reader to macOS, allowing you to use Touch ID-like authentication.

**Setup:**
```bash
# Enroll your fingerprint (run on the MacNix host)
macnix-fingerprint enroll

# Verify it works
macnix-fingerprint verify
```

The fingerprint bridge service (`macnix-fingerprint-bridge.py`) runs automatically and translates fingerprint auth requests from macOS to the host's `fprintd`.

> [!NOTE]
> Fingerprint bridging requires a Linux-compatible fingerprint reader. Most modern laptop sensors (Goodix, Synaptics, Elan) are supported via `fprintd`.

## ✅ What Works

- ✅ Full GPU acceleration (AMD recommended)
- ✅ iMessage, FaceTime, App Store (with valid SMBIOS)
- ✅ Audio output (via virtual audio device)
- ✅ USB passthrough (keyboards, mice, storage, DACs)
- ✅ Networking (bridged or NAT)
- ✅ Disk resize and management
- ✅ Sleep/Wake
- ✅ Fingerprint authentication (via bridge)
- ✅ Multiple monitors (with supported GPUs)
- ✅ macOS updates (OTA, through System Preferences)

## ❌ What Doesn't Work

- ❌ AirDrop / Handoff (requires real Apple Wi-Fi/BT hardware)
- ❌ Sidecar (requires Apple T2 chip)
- ❌ DRM content in Safari/TV+ (Apple DRM checks fail in VMs)
- ❌ Thunderbolt passthrough (controller-level limitation)
- ❌ NVIDIA GPU acceleration on macOS Mojave+ (Apple dropped support)

## 🔧 Troubleshooting

<details>
<summary><b>Black screen after boot</b></summary>

This usually means GPU passthrough failed. Check:
```bash
# Verify IOMMU is enabled
dmesg | grep -i iommu

# Check if your GPU is bound to vfio-pci
lspci -nnk | grep -A3 "VGA"

# View MacNix debug info
bash /opt/macnix/scripts/macnix-debug.sh
```
</details>

<details>
<summary><b>macOS installer hangs on Apple logo</b></summary>

- Ensure you selected the correct macOS version for your GPU
- Try regenerating OpenCore: `sudo bash /opt/macnix/scripts/phase4-qemu-config.sh`
- Check QEMU logs: `journalctl -u macnix-vm.service`
</details>

<details>
<summary><b>Poor performance / stuttering</b></summary>

- Verify hugepages are allocated: `cat /proc/meminfo | grep HugePages`
- Check CPU pinning: `cat /etc/macnix/vm.conf`
- Ensure your CPU governor is set to performance: `cpufreq-set -g performance`
</details>

<details>
<summary><b>USB devices not working in macOS</b></summary>

- USB devices must be passed through explicitly or via a USB controller
- Check connected devices: `lsusb`
- Restart the VM service: `sudo systemctl restart macnix-vm.service`
</details>

<details>
<summary><b>IOMMU grouping issues</b></summary>

If your GPU shares an IOMMU group with other devices:
```bash
# View IOMMU groups
for g in /sys/kernel/iommu_groups/*/devices/*; do
    echo "IOMMU Group $(basename $(dirname $(dirname $g))): $(lspci -nns ${g##*/})"
done
```
You may need an ACS override patch or a motherboard with better IOMMU grouping.
</details>

## 🛠 Building from Source

```bash
# Install build dependencies (Debian/Ubuntu)
sudo apt-get install live-build debootstrap xorriso squashfs-tools \
    grub-efi-amd64-bin grub-pc-bin mtools dosfstools isolinux \
    syslinux-common syslinux-utils

# Build the ISO
sudo bash scripts/phase7-build-iso.sh
```

The ISO will be output to `build/output/macnix-YYYYMMDD.iso`.

Alternatively, push to the `main` branch and GitHub Actions will build the ISO automatically.

## 🤝 Credits

MacNix stands on the shoulders of incredible open-source projects:

- **[OSX-KVM](https://github.com/kholia/OSX-KVM)** — The foundational work on running macOS under KVM/QEMU
- **[OpenCore](https://github.com/acidanthera/OpenCorePkg)** — The macOS bootloader that makes this all possible
- **[Docker-OSX](https://github.com/sickcodes/Docker-OSX)** — Inspiration and tooling for macOS virtualization
- **[Calamares](https://calamares.io/)** — The universal Linux installer framework

## 💝 Support the Project

If MacNix saved you from days of Hackintosh/VFIO headaches, consider supporting development:

**USDT (TON Network):**
```
UQBEJwLa4EGPRmUKw4O1i9d_JjJGmjkJ2myqR5lborzgceT-
```

---

<p align="center">
  Created by <b><a href="https://github.com/local-over">Hassan Elkady</a></b>
</p>

<p align="center">
  <sub><i>MacNix is an educational virtualization project. Please comply with Apple's Software License Agreement regarding macOS virtualization. macOS is a trademark of Apple Inc.</i></sub>
</p>
