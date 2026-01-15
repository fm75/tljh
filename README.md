# TLJH on Raspberry Pi 5
The Littlest Jupyter Hub (tljh) on cloud and RPi5

Installing **The Littlest JupyterHub (TLJH)** on a Raspberry Pi 5 is a great way to create a multi-user coding environment. Since the Pi 5 uses an ARM64 architecture, it is fully supported by TLJH, provided you are running a compatible OS like **Ubuntu 20.04/22.04/24.04** or **Debian 11/12**.

---

## 1. Prerequisites

Before you begin, ensure your Raspberry Pi 5:

* Is running a 64-bit version of **Ubuntu** or **Debian**.
* Is connected to your home network with a known IP address.
* Has at least **1GB of RAM** (all Pi 5 models meet this).

---

## 2. Installation Steps via SSH

### Step 1: Connect to your Raspberry Pi

Open a terminal on your computer and SSH into your Pi:

```bash
ssh <username>@<your-raspberry-pi-ip>

```

### Step 2: Update System & Install Dependencies

Ensure your package list is up to date and install the required tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-dev git curl -y

```

### Step 3: Run the TLJH Installer

The following command downloads and runs the TLJH [bootstrap script](https://tljh.jupyter.org/en/latest/install/custom-server.html). Replace `<admin-user-name>` with the username you want to use as the first administrator.

> **Note:** This user does **not** need to exist on the system yet; TLJH will create it in the web interface later.

```bash
curl -L https://tljh.jupyter.org/bootstrap.py | sudo -E python3 - --admin <admin-user-name>

```

*The installation typically takes 5–10 minutes. It will say **"Done!"** when finished.*

---

## 3. Accessing Your JupyterHub

1. Open a web browser and enter your Raspberry Pi’s IP address: `http://<your-pi-ip>`
2. Login with the **admin username** you chose in Step 3.
3. Choose any **password**. Since this is the first login, the password you enter will become the permanent password for that admin account.

---

## 4. Post-Installation Configuration

### Set Memory Limits

To prevent one user from crashing your Pi by using too much RAM, you can set a limit (e.g., 2GB per user):

```bash
sudo tljh-config set limits.memory 2G
sudo tljh-config reload

```

### Install Packages for All Users

To install libraries like NumPy or Pandas so every user can access them, use the `sudo -E conda` or `pip` command:

```bash
sudo -E /opt/tljh/user/bin/pip install numpy pandas matplotlib

```

---

### Important Considerations for Raspberry Pi

* **Static IP:** Ensure your Pi has a static IP address in your router settings so the URL doesn't change.
* **ARM Compatibility:** While TLJH works on ARM, some specific Python libraries might not have ARM wheels. If a `pip install` fails, check if there is an `apt` version of the package.
* **HTTPS:** If you plan to access this from outside your home network, you should [enable HTTPS](https://tljh.jupyter.org/en/latest/howto/admin/https.html) to secure your login credentials.

Would you like me to show you how to set up a **Public URL** for your Pi using a service like Tailscale or Cloudflare Tunnels?
