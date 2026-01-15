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
It is **technically possible**, but there are some important distinctions in how The Littlest JupyterHub (TLJH) operates that you should consider before proceeding.

On a Raspberry Pi 5 running the 6.12 kernel, you have plenty of power to compile and run Python 3.14, but TLJH is "opinionated" about its Python environment.

---

## 1. The Core Conflict: Hub vs. User Environments

TLJH manages two distinct Python environments:

* **The Hub Environment (`/opt/tljh/hub`):** This runs the JupyterHub server itself. It is usually installed using the system’s default `python3` or a version specified by the installer.
* **The User Environment (`/opt/tljh/user`):** This is where your notebooks run and where you install libraries like NumPy or OpenCV.

### Should you change the Hub Python to 3.14?

**Probably not.** The Hub environment needs to be extremely stable. Since Python 3.14 is the "latest and greatest" as of early 2026, some TLJH plugins or internal dependencies (like `traefik-proxy` or specific JupyterHub authenticators) might not be fully tested for 3.14 yet.

### The Recommendation

The best approach is to **install TLJH normally** (letting it use the stable 3.12 for the Hub) and then **add Python 3.14 as a separate Kernel**. This allows you to write code in 3.14 while keeping the server infrastructure on a verified stable version.

---

## 2. How to Install Python 3.14 on your RPi5

Since Debian (the base of Raspberry Pi OS) moves slowly, 3.14 might not be in the `apt` repositories yet. You will likely need to build it from source.

### Step 1: Install Build Dependencies

```bash
sudo apt update
sudo apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev \
libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget curl bzip2

```

### Step 2: Download and Compile

```bash
wget https://www.python.org/ftp/python/3.14.0/Python-3.14.0.tar.xz
tar -xf Python-3.14.0.tar.xz
cd Python-3.14.0
./configure --enable-optimizations
make -j4  # Using 4 cores on your RPi5
sudo make altinstall

```

> **Note:** Use `altinstall` to prevent overwriting the system's default `python3` binary, which would likely break your OS. This will install it as `python3.14`.

---

## 3. Registering 3.14 as a TLJH Kernel

Once 3.14 is installed, you can make it available to all users in your JupyterHub:

1. **Create a virtual environment for the 3.14 kernel:**
```bash
sudo /usr/local/bin/python3.14 -m venv /opt/tljh/user/envs/python314

```


2. **Install the kernel into that environment:**
```bash
sudo /opt/tljh/user/envs/python314/bin/pip install ipykernel

```


3. **Register the kernel so it appears in the Jupyter interface:**
```bash
sudo /opt/tljh/user/envs/python314/bin/python -m ipykernel install --name 'python314' --display-name 'Python 3.14' --prefix /opt/tljh/user

```



---

## 4. If you still want 3.14 as the "Default"

If you absolutely want the entire TLJH setup to bootstrap using 3.14, you would run the bootstrap script using that specific version:

```bash
curl -L https://tljh.jupyter.org/bootstrap.py | sudo python3.14 - --admin <admin-username>

```

**Risk Factor:** If any of the TLJH core dependencies (like `jupyterhub` or `tornado`) have not been patched for 3.14's specific changes (such as deprecated C APIs or syntax), the installation will fail midway, leaving you with a broken `/opt/tljh` directory.

### Summary Table

| Goal | Method | Risk |
| --- | --- | --- |
| **Stability** | Use 3.12 for TLJH; Add 3.14 as Kernel | **Low** (Recommended) |
| **Bleeding Edge** | Bootstrap TLJH with `python3.14` | **High** (Likely dependency errors) |
| **OS Integrity** | Use `make altinstall` for 3.14 | **Safe** |
| **OS Integrity** | Overwrite system `python3` with 3.14 | **Fatal** (Will break `apt` and boot) |

**Would you like me to provide the specific commands to set up that Python 3.14 kernel with common data science libraries after you finish the TLJH installation?**
---

### Important Considerations for Raspberry Pi

* **Static IP:** Ensure your Pi has a static IP address in your router settings so the URL doesn't change.
* **ARM Compatibility:** While TLJH works on ARM, some specific Python libraries might not have ARM wheels. If a `pip install` fails, check if there is an `apt` version of the package.
* **HTTPS:** If you plan to access this from outside your home network, you should [enable HTTPS](https://tljh.jupyter.org/en/latest/howto/admin/https.html) to secure your login credentials.

Would you like me to show you how to set up a **Public URL** for your Pi using a service like Tailscale or Cloudflare Tunnels?
