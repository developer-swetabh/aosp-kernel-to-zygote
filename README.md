# AOSP Boot Sequence — ABL to Zygote

Interactive deep-dive call-flow block diagram visualizing the full Android Open Source Project boot sequence.

---

## 📦 Deliverables

| File | Format | Description |
|------|--------|-------------|
| `aosp-boot-sequence.html` | Self-contained HTML | Interactive animated overview — click any block to explore |
| `deep-dive/abl.html` | Self-contained HTML | Deep dive: Android Bootloader |
| `deep-dive/boot-image.html` | Self-contained HTML | Deep dive: Boot Image Format Evolution (legacy vs init_boot.img) |
| `deep-dive/kernel.html` | Self-contained HTML | Deep dive: Linux Kernel |
| `deep-dive/init.html` | Self-contained HTML | Deep dive: init (First Stage / Ramdisk) |
| `deep-dive/avb.html` | Self-contained HTML | Deep dive: Android Verified Boot 2.0 |
| `deep-dive/merkle-fail.html` | Self-contained HTML | Deep dive: Merkle Hash Tree & AVB Failure |
| `deep-dive/slot-selection.html` | Self-contained HTML | Deep dive: A/B Slot Selection |
| `deep-dive/mount.html` | Self-contained HTML | Deep dive: Mount Filesystem Images |
| `deep-dive/sepolicy.html` | Self-contained HTML | Deep dive: SELinux / sepolicy Setup |
| `deep-dive/services.html` | Self-contained HTML | Deep dive: Service Startup (Second-Stage init) |
| `deep-dive/zygote.html` | Self-contained HTML | Deep dive: Zygote (Android Runtime Origin) |
| `README.md` | Markdown | This file — citations, descriptions, usage notes |

## 🚀 Usage

### Opening the animation

1. **Double-click** `aosp-boot-sequence.html` in any file explorer, or
2. Run from terminal:
   ```bash
   # Linux
   xdg-open aosp-boot-sequence.html
   # macOS
   open aosp-boot-sequence.html
   # Windows
   start aosp-boot-sequence.html
   ```

### Interactive Deep Dives

- **Click any block** in the main animated flow to open its deep-dive page
- Each deep-dive page contains:
  - Overview and stepwise "How It Works" animation
  - Interactive SVG diagrams (Merkle tree, partition layout, fork model, etc.)
  - Key files & artifacts table
  - Common failure modes
  - Vendor-specific details (collapsible)
  - Code snippets with copy buttons
  - Citations and references
- **Navigation**: Use the back button, breadcrumb, or `Escape` key to return to the overview
- **Sequential reading**: Each deep-dive has Previous/Next links to read the entire sequence

### Keyboard Controls

| Key | Action |
|-----|--------|
| `Space` | Play / Pause animation (main page) |
| `→` | Next step |
| `←` | Previous step |
| `R` | Reset to beginning |
| `A` | Show all steps at once |
| `H` | Toggle high-contrast mode |
| `Escape` | Return to overview (from deep-dive pages) |
| `Enter` | Open deep-dive for focused block |

### State Preservation

When navigating to a deep-dive page, the main flow automatically saves your current position (step and speed). When you return, it restores your exact state.

### Deep Link URLs

Each block has a URL fragment for direct linking:
- `aosp-boot-sequence.html#abl` — ABL block
- `aosp-boot-sequence.html#bootimage` — Boot Image Format block
- `aosp-boot-sequence.html#kernel` — Kernel block
- `aosp-boot-sequence.html#init` — init block
- `aosp-boot-sequence.html#avb` — AVB block
- `aosp-boot-sequence.html#merkle` — Merkle tree
- `aosp-boot-sequence.html#slot` — Slot Selection
- `aosp-boot-sequence.html#mount` — Mount Images
- `aosp-boot-sequence.html#sepolicy` — SELinux
- `aosp-boot-sequence.html#services` — Service Startup
- `aosp-boot-sequence.html#zygote` — Zygote

### Exporting

- **Print to PDF**: `Ctrl+P` → "Save as PDF"
- **Offline use**: All files are fully self-contained — no internet connection required

---

## 🗂️ Block Descriptions

### 1. ABL — Android Bootloader (Green)
SoC ROM loads the Android Bootloader, which verifies the boot chain using vbmeta signatures, reads A/B slot metadata, and loads the kernel + ramdisk into memory.

### 1.5. Boot Image Format Evolution (Cyan)
Explains the evolution from the monolithic `boot.img` (kernel + ramdisk bundled) to the modern Android 13+ split: `boot.img` (kernel only) + `init_boot.img` (generic ramdisk) + `vendor_boot.img` (vendor ramdisk). Covers header versions v0–v4, build system variables, and the practical benefits of the modular approach.

### 2. Linux Kernel (Red)
Decompresses and initializes the kernel image (GKI since Android 12). Sets up core subsystems (scheduler, memory, drivers) and mounts the initial ramdisk (`initramfs`) as temporary root.

### 3. init — First Stage / Ramdisk (Orange)
The kernel executes `/init` (PID 1) from the ramdisk. Parses early `init.rc` files and reads `fstab.qcom` (⚠️ vendor-specific) to discover the partition layout.

### 4. AVB — Android Verified Boot 2.0 (Orange)
Validates the vbmeta chain, sets up `dm-linear` for logical partitions, and configures `dm-verity` for block-level integrity verification.

### 5. Merkle Hash Tree / AVB Failure Branch (Orange / Red)
- **Pass**: dm-verity lazily verifies each 4 KiB block via Merkle tree on read.
- **Fail**: Tampered partition → boot halts (locked) or shows warning (unlocked).

### 6. Slot Selection — A/B (Blue)
Reads slot metadata from `misc` partition. Evaluates `successful_boot`, `unbootable`, `retry_count` to select the healthy slot.

### 7. Mount Filesystem Images (Blue)
Dynamic partitions from `super` via dm-linear. Mounts `/system`, `/vendor`, `/odm`, `/product`, `/system_ext`.

### 8. SELinux / sepolicy Setup (Purple)
Loads compiled policy, enforces MAC, labels filesystems, re-executes init in `u:r:init:s0`.

### 9. Service Startup (Teal)
Parses full `init.rc` tree. Starts `class core` → `class main` → `class late_start`.

### 10. Zygote (Teal)
Preloads ~8000 classes, forks `system_server`, listens on socket to fork app processes.

### 11. Boot Complete
`sys.boot_completed=1` · Launcher visible · System operational.

---

## 📖 Citations

### Citation 1 — Bootloader and ABL
> The bootloader is a vendor-proprietary image responsible for bringing up the kernel on a device.

**Source**: [AOSP — Bootloader](https://source.android.com/docs/core/architecture/bootloader)

### Citation 2 — Boot Image Header and init_boot.img
> The boot image header has evolved from v0 to v4. Android 13 introduced the init_boot partition to hold the generic ramdisk, separating it from the kernel in boot.img.

**Source**: [AOSP — Boot Image Header](https://source.android.com/docs/core/architecture/bootloader/boot-image-header), [AOSP — Generic Boot Partition](https://source.android.com/docs/core/architecture/bootloader/partitions/generic-boot)

### Citation 3 — Android Verified Boot 2.0 (AVB)
> AVB 2.0 uses a hash tree (Merkle tree) to verify block integrity. The vbmeta structure contains hash descriptors, hashtree descriptors, and chained partition descriptors.

**Source**: [AOSP — Verified Boot (AVB)](https://source.android.com/docs/security/features/verifiedboot/avb)

### Citation 4 — dm-verity and Merkle hash trees
> dm-verity provides transparent integrity checking of block devices using a Merkle hash tree. Each block is verified on read; corruption returns an I/O error.

**Source**: [AOSP — Implementing dm-verity](https://source.android.com/docs/security/features/verifiedboot/dm-verity)

### Citation 5 — A/B System Updates and Slot Selection
> A/B system updates use two sets of partitions (slots). The bootloader reads boot control metadata to determine which slot is active.

**Source**: [AOSP — A/B System Updates](https://source.android.com/docs/core/ota/ab)

### Citation 6 — SELinux policy loading
> Android uses SELinux to enforce mandatory access control (MAC). During boot, init loads the compiled SELinux policy and transitions to enforcing mode.

**Source**: [AOSP — SELinux](https://source.android.com/docs/security/features/selinux)

### Citation 7 — init and service classes
> Android's init is PID 1. It parses init.rc scripts to define services organized into classes (core, main, late_start) started in order.

**Source**: [AOSP — Init](https://source.android.com/docs/core/architecture/configuration/init)

### Citation 8 — Zygote and SystemServer
> Zygote preloads classes and resources shared by all apps. It forks SystemServer, then listens for app launch requests.

**Source**: [AOSP — Android Runtime](https://source.android.com/docs/core/runtime)

### Citation 9 — Dynamic Partitions
> Logical partitions are carved from a single physical `super` partition using dm-linear.

**Source**: [AOSP — Dynamic Partitions](https://source.android.com/docs/core/ota/dynamic_partitions)

### Citation 10 — fstab (vendor-specific)
> Vendor fstab files (e.g., `fstab.qcom`) define partition-to-mount-point mapping for first-stage init.

**Source**: [AOSP — Partitions and Images](https://source.android.com/docs/core/architecture/bootloader/partitions-images)

---

## ⚠️ Vendor-Specific Notes

- **`fstab.qcom`**: Qualcomm-specific. Other SoCs use `fstab.<hardware>` (e.g., `fstab.exynos`, `fstab.tensor`).
- **ABL**: Qualcomm uses XBL→ABL; MediaTek uses preloader→LK; Tensor uses custom stages.
- **Boot control HAL**: `android.hardware.boot` implementation is vendor-specific.

All vendor-specific details are marked with collapsible sections in deep-dive pages.

---

## 🔧 Technical Details

- **Format**: Self-contained HTML with inline CSS, SVG, and JavaScript
- **Offline**: No CDN, no external fonts, no network requests
- **Accessibility**: Keyboard nav, ARIA labels, focus indicators, high-contrast (WCAG AA)
- **Browser support**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Total size**: ~220 KB (12 HTML files)
- **Interactive animations**: Merkle tree, Zygote fork, service startup, dm-verity layering

---

*Generated March 2026 · Content verified against AOSP documentation (source.android.com)*
