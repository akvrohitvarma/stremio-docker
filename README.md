# 🚀 Ultimate Stremio Server Setup with NordVPN & Cloudflare Tunnel

A complete, production-ready guide to deploying a private Stremio streaming server with enterprise-level privacy and remote access capabilities using Proxmox, Docker, NordVPN, and Cloudflare Tunnel.

## 🎯 What This Repository Provides

This comprehensive guide helps you build a **secure, private streaming server** that you can access from anywhere while maintaining complete anonymity through NordVPN protection.

# Technologies Used

- **Proxmox VE** - Virtualization platform for container management
- **Debian 13 (Trixie)** - Base operating system for LXC container
- **Docker** - Containerization platform for Stremio server
- **Docker Compose** - Multi-container Docker application management
- **NordVPN** - Privacy and security through VPN tunneling
- **Cloudflare Tunnel** - Secure remote access without port forwarding

## 🚀 Proxmox LXC Container Creation

Follow these steps within your Proxmox web interface to prepare the environment.

### 1. Template Download

1.  Navigate to the storage location where you want to download the template (e.g., **local**, but **not** `local-lvm`).
2.  Click on **CT Templates**.
3.  Click on **Templates**.
4.  Download the **`debian-12-standard`** template (or `debian-13-standard` if available and preferred).

### 2. Create the LXC Container

1.  Click on your **Proxmox node** name.
2.  Click on **Create CT**.
3.  **General Settings:**
    * Set **CT ID** (e.g., `101`).
    * Set **Hostname** (e.g., `stremio-service`).
    * Set a strong **password**.
4.  **Template:**
    * Select the storage where the template was downloaded (e.g., `local`).
    * Choose the **`debian-12-standard`** template.
5.  **Disk:**
    * Select the storage where the LXC container data will be saved.
6.  **CPU:**
    * Set **Cores** to **1** (minimum, more is better for performance).
7.  **Memory:**
    * Set **Memory** to **2048 MB**.
    * Set **Swap** to **2048 MB**.
8.  **Network:**
    * Set **IPv4** to **DHCP**.
    * Set **IPv6** to **DHCP** (or your preferred settings).
9.  **DNS:**
    * Click **Next** (default settings are usually fine).
10. Click **Finish** to create the LXC container.

---

## 🔒 First Boot, System Update, and NordVPN Setup

This section covers basic setup, installing necessary tools, and configuring NordVPN.

### 1. Initial Login and Updates

1.  Click on your new container (**`stremio-service`**) and press **Start**.
2.  Go to the **Console** and wait for the container to boot.
3.  Login with username `root` and the password you set during creation.
4.  Execute the initial system updates:

    ```bash
    apt update && apt upgrade -y && apt autoremove -y
    ```

5.  Install `curl`, which is needed to download the NordVPN installer:

    ```bash
    apt install curl -y
    ```

### 2. Install and Configure NordVPN

1.  Install the NordVPN tool:

    ```bash
    sh <(curl -sSf [https://downloads.nordcdn.com/apps/linux/install.sh](https://downloads.nordcdn.com/apps/linux/install.sh))
    ```

    > **Note:** The command provided in the input, `chmod +x install.sh` followed by `bash install.sh`, is often an alternative or older method. The single-line `sh <(...)` command is the official and more streamlined installation method for the NordVPN repository and tool.

2.  **Generate a NordVPN Access Token:**
    * In your web browser, log in to your **NordVPN account**.
    * Navigate to the section to get an access token (usually under the main account settings).
    * Follow the verification steps and **copy the access token**.

3.  Log in to NordVPN in the LXC console using the token:

    ```bash
    nordvpn connect --token <your-token>
    ```

4.  Configure NordVPN settings. Replace `<country name>` with your desired location (e.g., `Zurich`):

    ```bash
    nordvpn set analytics off
    nordvpn set autoconnect enabled <country name>
    nordvpn whitelist add port 80
    nordvpn whitelist add port 443
    nordvpn whitelist add port 8080
    nordvpn whitelist add subnet 0.0.0.0/24
    ```

5.  Check the status. It might show disconnected after the settings change:

    ```bash
    nordvpn status
    ```

6.  **Reboot** the container to apply autoconnect:

    * From the Proxmox GUI, click the **Reboot** button, or
    * Execute the command in the shell: `reboot -f`

7.  After the container successfully reboots, log back in and check the status again. It should show a connection:

    ```bash
    nordvpn status
    ```

    ✅ **VPN is now working and will automatically connect on boot.**

---

## ⚡ Enabling Hardware Acceleration (iGPU Passthrough)

This is crucial for transcoding video streams efficiently, preventing high CPU load on the Proxmox host. This guide assumes you have an **Intel iGPU** (Quick Sync) in your Proxmox host.

### 1. Proxmox Host Preparation

The steps below are executed on your **Proxmox Host** console (the main PVE shell, not inside the LXC container).

#### A. Identify GPU Device Nodes

First, confirm the presence and major/minor numbers of your Intel GPU devices.

1.  List the device files to get the major and minor numbers:
    ```bash
    ls -l /dev/dri
    ```
    You will likely see output similar to this (note the numbers **226, 0** and **226, 128**):
    ```
    crw-rw---- 1 root video 226, 0   # <-- card0 (Major 226, Minor 0)
    crw-rw---- 1 root render 226, 128 # <-- renderD128 (Major 226, Minor 128)
    ```

2.  If the numbers differ from `226:0` and `226:128`, **use your numbers** in the next step.
3.  LXC Container Preparation (Find Group ID)

You need to find the specific Group ID (GID) that the `render` group uses inside your Debian LXC container, as this number must be mapped accurately.

-  Log into your Stremio LXC container's console (via Proxmox or SSH).
-  Execute the following command to retrieve the GID for the `render` group:

    ```bash
    getent group render | cut -d: -f3
    ```
    *Note: This command usually returns a number like **104** or **108**. Write this number down.*

#### B. Configure LXC Passthrough

Edit the container configuration file. Replace `<CT_ID>` with your Stremio container's ID (e.g., `101`).

1.  Edit the configuration file:
    ```bash
    nano /etc/pve/lxc/<CT_ID>.conf
    ```

2.  Add the following lines to the very end of the file. These lines allow the container to access the device and mount it to the correct location (using the common device numbers):

    ```ini
    # Passthrough iGPU device nodes for hardware acceleration (VAAPI)
    lxc.cgroup2.devices.allow: c 226:0 rwm
    lxc.cgroup2.devices.allow: c 226:128 rwm
    lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
    lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
    ```
    *(If your numbers were different, ensure you update `226:0` and `226:128` accordingly).*

3.  Save the file (`Ctrl+S`) and exit (`Ctrl+X`).

4.  **Reboot the LXC container** from the Proxmox UI to apply the configuration changes.

### 2. Debian LXC Container Driver Installation

Now, log back into your Stremio LXC container's shell (as root) to install the necessary decoding libraries.

1.  **Install VAAPI Drivers:**
    ```bash
    # Ensure non-free component is enabled in sources list (mandatory for Intel media drivers)
    # Install the necessary VAAPI driver package
    apt update
    apt install -y intel-media-va-driver-non-free
    ```
### 3. Configure Device Passthrough in Proxmox GUI

1.  In the Proxmox web interface, click on your **LXC container** (`stremio-service`).
2.  Navigate to the **Resources** panel.
3.  Click the **Add** button, and select **Device Passthrough**.
4.  In the configuration window:
    * **Device Path:** Enter `/dev/dri/renderD128`.
    * Check the **Advanced Options** box.
    * **GID in CT:** Enter the GID you found in step 1 (e.g., `104`).
    * **UID in CT:** Leave this empty.
    * **Access Mode:** Select `0666` (This provides the container with necessary read/write permissions for transcoding).
5.  Click **Add**.---

Your container now has hardware acceleration!

## 🐳 Installing Docker and Docker Compose

You'll need Docker and Docker Compose to easily run the Stremio server.

### 1. Install Docker

Execute the following commands to download and run the official Docker installation script:

```bash
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sh get-docker.sh
```
### 2. Install Docker Compose

Install Docker Compose v2.22.0 (or the latest stable version) by downloading the binary and making it executable:

```bash
# Download Docker Compose v2.22.0
curl -L "[https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-$(uname](https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-$(uname) -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make the binary executable
chmod +x /usr/local/bin/docker-compose
```

### 3. Verification

Verify that both Docker and Docker Compose are installed correctly:

```bash
docker --version
docker-compose --help
```

## 💻 Setting Up Docker Container to Run Stremio Server

With Docker and NordVPN configured, you can now deploy the Stremio server using a `docker-compose.yml` file. This setup ensures that the Stremio server traffic is routed through the VPN connection established in the LXC container.

### 1. Create the `docker-compose.yml` File

Execute the following command in the container's root directory to create and open the `docker-compose.yml` file using the `nano` text editor:

```bash
nano docker-compose.yml
```

### 2. Paste Configuration

Paste the following content into the nano editor:

```yml
services:
  stremio:
    image: tsaridas/stremio-docker:latest
    container_name: stremio-server
    restart: unless-stopped
    ports:
      - "80:8080"
    volumes:
      - ./stremio-data:/root/.stremio-server
    devices:
      - "/dev/dri/renderD128:/dev/dri/renderD128"  # Enable hardware acceleration (if supported by Proxmox/LXC)
    environment:
      - DOMAIN=<your purchased domain>  # **IMPORTANT: CHANGE THIS**
      - NO_CORS=1
      - AUTO_SERVER_URL=1
      - CASTING_DISABLED=1
    networks:
      - stremio-network

networks:
  stremio-network:
    driver: bridge
```

### 3. Customize and Save

* **Crucial Step:** Make sure to replace `<your purchased domain>` with your actual domain name in the `DOMAIN` environment variable.
    * *Optional:* For more details on other environment variables, refer to the image's documentation: `https://hub.docker.com/r/tsaridas/stremio-docker`
* Press **`Ctrl+S`** to save the file.
* Press **`Ctrl+X`** to exit the `nano` editor.

### 4. Deploy the Container

Execute the following command to start the Stremio server container in the background (`-d` for detached mode):

```bash
docker-compose up -d
```

Docker will now pull the tsaridas/stremio-docker:latest image, and then it will successfully set up the stremio-server container.

### 5. Access the Stremio Server

1.  Find the IP address of your LXC container by executing:

    ```bash
    ip a
    ```
    Look for the IP address listed under the **`eth0`** interface.

2.  On any device (laptop, mobile, etc.) on the same local network, open a web browser and navigate to the IP address you found (e.g., `http://192.168.1.100`).

You should now see the Stremio server interface running!

> **Note:** Because we used `restart: unless-stopped` in the configuration, the Stremio server container will automatically start whenever the LXC container reboots.

## 🌎 Setting Up Cloudflare Tunnelling for Remote HTTPS Access

Cloudflare Tunnels (formerly Argo Tunnel) provides a secure, HTTPS-encrypted connection from your LXC container to the Cloudflare network, allowing worldwide access without exposing any ports on your home router.

### 1. Cloudflare Zero Trust Configuration

Follow these steps in your web browser:

1.  Go to the **Cloudflare Dashboard** at `dash.cloudflare.com`.
2.  Navigate to **Zero Trust**.
3.  Click on **Networks**, then click on **Tunnels**.
4.  Click on **Create a new tunnel**.
5.  Select the **Cloudflared** connector option.
6.  **Name your tunnel** (e.g., `stremio-service`) and click **Save**.

### 2. Install and Run `cloudflared` on LXC

1.  On the next screen, follow the on-screen instructions to **Install and run `cloudflared`** on your Debian 64-bit LXC container.
2.  After installing the binary on your LXC, the Cloudflare website will provide a specific command to install the service. **Copy this entire command** (it includes your unique token) and paste it into your LXC terminal:

    ```bash
    # Example command (token will be unique, copy it exactly from the Cloudflare page)
    sudo cloudflared service install <token-provided-by-cloudflare>
    ```
    Click **Next** on the Cloudflare website after executing the command.

### 3. Configure Public Hostname

This step links your custom domain to the internal Stremio service:

1.  In the **Public Hostname** section of the Cloudflare UI, set up the following:
    * **Subdomain (Optional):** Add a subdomain (e.g., `movies`).
    * **Domain:** Select the domain you have purchased and added to Cloudflare (e.g., `yourdomain.com`).
    * **Path:** Leave this empty.
2.  Under **Service** configuration:
    * **Type:** Select **HTTP**.
    * **URL:** Type the internal address of your LXC container running Stremio: `http://<lxc ip>:80` (Replace `<lxc ip>` with the IP address you found earlier using `ip a`).
3.  Click **Save Hostname** and then **Complete setup**.

Now if you go to `movies.<yourdomain>.com`, you will see the Stremio web UI, and you can access this from anywhere in the world to your self-hosted LXC container via HTTPS, secured by Cloudflare.

---
**End of Process: Enjoy your self-hosted movie streaming journey!**
