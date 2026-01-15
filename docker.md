# Why TLJH Failed on Raspberry Pi 5 (Debian Trixie)

### 1. The Systemd Paradox
**Issue:** The TLJH `bootstrap.py` script reported "Systemd is required" even though your system was running it.
**Reason:** In **Debian 13 (Trixie)** and the **6.12 kernel**, several system paths have shifted due to the "merged-usr" transition. The TLJH installer uses a legacy detection method that looks for systemd in hard-coded locations (like `/bin/systemctl` instead of `/usr/bin/systemctl`). 

### 2. Debian Trixie vs. Ubuntu 22.04
**Issue:** Missing `software-properties-common`.
**Reason:** TLJH is "opinionated" and assumes an Ubuntu-based environment. It tries to use Ubuntu-specific tools (PPAs) to fetch dependencies. On Debian Trixie (which is the "Testing" branch as of 2026), these packages are either renamed or handled differently by the modern `apt` stack, causing the installer to crash with "Exit Code 100".

### 3. PEP 668: Externally Managed Environments
**Issue:** The "Global Pip" restriction.
**Reason:** Modern Debian (12+) prevents users from running `pip install` outside of a virtual environment to protect the OS. TLJH's installer tries to perform a global installation that conflicts with these new security standards, leading to permission and environment "breakage" during the bootstrap process.

### 4. Architecture Mismatches (ARM64)
**Issue:** Binary availability.
**Reason:** TLJH installs **Traefik** and **Node.js** as fixed binaries. On the Pi 5's `aarch64` architecture, the installer sometimes fails to locate the correct ARM-specific binaries for the newer Debian Trixie repositories, leading to "404 Not Found" errors during the download phase.


---

# Raspberry Pi 5 JupyterLab AI & OpenCV Setup

This guide documents the successful Docker configuration for a Raspberry Pi 5 running Debian Trixie, featuring Python 3.12, OpenCV, and Jupyter-AI.

## 1. Directory Structure
```text
~/jupyterhub/
├── data/               # Persistent notebook storage
├── Dockerfile          # Image definition
└── docker-compose.yml  # Service orchestration

