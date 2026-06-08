# Dual Boot Kali Linux on Windows 11 (One Clean Partition)

> Step by step guide to install Kali Linux alongside Windows 11 — keeping everything in one clean partition.

---

## What You Need

- Windows 11 PC
- USB drive (8GB or more)
- At least 50GB free disk space (100GB+ recommended)
- ~1 hour

---

## Step 1 — Disable Fast Startup (Windows)

Fast Startup locks your drives and can cause corruption.

1. Open **Control Panel** → Power Options
2. Click **"Choose what the power buttons do"**
3. Click **"Change settings that are currently unavailable"**
4. **Uncheck** Turn on fast startup
5. Click Save changes

---

## Step 2 — Shrink Your Windows Partition

> ⚠️ Always do this from Windows — never resize from Linux.

1. Press `Win + X` → **Disk Management**
2. Right-click your **C: drive** → **Shrink Volume**
3. Enter how much space to shrink (in MB):
   - 100GB → type `102400`
   - 150GB → type `153600`
4. Click **Shrink**
5. You will now see a black **Unallocated** block — leave it as-is

---

## Step 3 — Disable Secure Boot (BIOS)

1. Restart your PC and enter BIOS
   - Common keys: `F2` `F12` `DEL` `ESC`
2. Find **Secure Boot** → set to **Disabled**
3. Save and exit

---

## Step 4 — Create Kali Bootable USB

1. Download Kali **Installer** ISO from [https://www.kali.org/get-kali](https://www.kali.org/get-kali)
2. Download **Rufus** from [https://rufus.ie](https://rufus.ie)
3. Open Rufus:
   - Device → select your USB
   - Boot selection → select the Kali `.iso`
   - Partition scheme → **GPT**
   - Target system → **UEFI (non CSM)**
4. Click **START** → select **ISO mode** if prompted
5. Wait for it to finish

---

## Step 5 — Boot From USB

1. Restart your PC
2. Enter your boot menu (usually `F12` or `F11`)
3. Select your USB drive
4. At the Kali menu → select **Graphical Install**

---

## Step 6 — Partition Setup (Most Important Step)

Work through language and keyboard settings until you reach **Partitioning**.

### 6.1 Select Manual

```
Partitioning method → Manual
```

### 6.2 Create EFI Partition (on the Unallocated space)

```
Size        → 512 MB
Use as      → EFI System Partition
```

### 6.3 Create Root Partition (everything else)

```
Size        → remaining space (all of it)
Use as      → Ext4 journaling file system
Mount point → /
```

> No swap partition needed — we will use a swapfile later.

### 6.4 Assign the Existing Windows EFI (CRITICAL)

> ⚠️ This is the step most people miss. If you skip this, Windows will disappear from your boot menu.

1. Find the small **260MB EFI partition** (already exists from Windows)
2. Click on it
3. Set **Use as** → `EFI System Partition`
4. Set **Mount point** → `/boot/efi`
5. **DO NOT format it** — leave Windows boot files intact

### 6.5 Finish

1. Select **Finish partitioning and write changes to disk**
2. Confirm → **Yes**
3. Complete the rest of the install (username, password, etc.)
4. When asked about GRUB → select **Yes** → point to your main disk (e.g. `/dev/nvme0n1`)

---

## Step 7 — First Boot into Kali

After install reboot into Kali and open a terminal.

### 7.1 Update system

```bash
sudo apt update && sudo apt upgrade -y
```

### 7.2 Fix GRUB so Windows shows up

```bash
sudo apt install os-prober -y
sudo nano /etc/default/grub
```

Add this line at the bottom:

```
GRUB_DISABLE_OS_PROBER=false
```

Save: `Ctrl+O` → `Enter` → `Ctrl+X`

```bash
# Verify Windows is detected
sudo os-prober
```

Expected output:
```
/dev/nvme0n1p1@/efi/Microsoft/Boot/bootmgfw.efi:Windows Boot Manager:Windows:efi
```

```bash
# Update GRUB
sudo update-grub

# Reboot
sudo reboot
```

Your boot menu will now show both **Kali** and **Windows**.

### 7.3 Create Swapfile (replaces swap partition)

```bash
# Change 16G to match your RAM size
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Final Partition Layout

This is what your disk should look like when done:

```
[ 260MB EFI ]  [ Windows C: ]  [ Your Files ]  [ 512MB EFI ]  [ Kali / ]
                                                 ↑ tiny sliver   ↑ one big partition
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Windows disappears from boot | You created a new EFI instead of reusing the 260MB one |
| GRUB only shows Kali | Set `GRUB_DISABLE_OS_PROBER=false` and run `sudo update-grub` |
| Can't shrink C: drive | Disable Fast Startup first, then try again |
| Secure Boot error on boot | Go back to BIOS and disable Secure Boot |

---

*Tested on Windows 11 + Kali Linux 2024/2025 — single NVMe drive, UEFI, GPT.*
