# Raspberry PI 5 version

Here is a clean Markdown summary of the steps we have completed to get your Raspberry Pi 5 booting from the NVMe drive in the Argon One V3 case.

---

# Raspberry Pi 5 NVMe Setup Guide (Argon One V3)

**Objective:** Configure Raspberry Pi 5 to boot from an NVMe SSD installed in an Argon One V3 case, optimize speeds, and manage case cooling.

### 1. Confirm Drive Detection

Before cloning, verify the system sees the NVMe drive.

* **Check Block Devices:**
```bash
lsblk

```


*Success Indicator:* You should see `nvme0n1` listed.
* **Check PCI Bus (if needed):**
```bash
lspci

```


*Success Indicator:* Look for "Non-Volatile memory controller".

### 2. Clone OS from SD Card to NVMe

Use `rpi-clone` to copy the running OS to the new drive.

1. **Install Git:**
```bash
sudo apt update && sudo apt install git -y

```


2. **Install `rpi-clone`:**
```bash
git clone https://github.com/geerlingguy/rpi-clone.git
cd rpi-clone
sudo cp rpi-clone rpi-clone-setup /usr/local/sbin

```


3. **Run the Clone Script:**
```bash
sudo rpi-clone nvme0n1

```


* **Prompts:** Type `yes` to initialize. Press `Enter` for label. Press `Enter` (defaults) for partition checks.



### 3. Set Boot Order

Configure the Pi firmware to prioritize the NVMe drive.

1. Open configuration:
```bash
sudo raspi-config

```


2. Navigate to: **6 Advanced Options** > **A6 Boot Order**.
3. Select: **B2 NVMe/USB Boot**.
4. Select **Finish**, but **do not reboot yet**.

### 4. Verify Boot (The "SD Card Pull" Test)

1. Shutdown the Pi:
```bash
sudo shutdown now

```


2. **Physical Step:** Remove the MicroSD card from the slot.
3. Power on the Pi.
4. SSH into the Pi.
5. Verify the root filesystem is mounted on NVMe:
```bash
lsblk

```


*Success Indicator:* The `/` mountpoint should be next to `nvme0n1p2`.

### 5. Enable PCIe Gen 3 Speed (Optional)

Unlock faster transfer speeds (Pi 5 defaults to Gen 2).

1. Check current speed:
```bash
sudo lspci -vv | grep -E "LnkCap|LnkSta"

```


2. Edit config file:
```bash
sudo nano /boot/firmware/config.txt

```


3. Add to the bottom (under `[all]`):
```text
dtparam=pciex1_gen=3

```


4. Save (`Ctrl+O`, `Enter`) and Exit (`Ctrl+X`).
5. Reboot and verify `LnkSta` shows **Speed 8GT/s**.

### 6. Install Argon One Fan Control

Ensure the fan works and the power button performs safe shutdowns.

1. **Install Script:**
```bash
curl https://download.argon40.com/argon1.sh | bash

```


2. **Configure Fan Profile:**
```bash
argonone-config

```


*Recommended:* Select Option 2 (Automatic) or set a custom curve.

### 7. Networking

* **Status:** Static IP setup **SKIPPED**.
* **Current State:** Device is using dynamic IP assignment via DHCP.

---

### Next Step

Since you held off on the Static IP, you are likely still on Wi-Fi or a dynamic ethernet connection.

**Would you like to start the installation of The Littlest JupyterHub (TLJH) now, or do you want to resolve the Ethernet/Connection question first?**
