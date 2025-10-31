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

# Creating Debian 13 LXC Container - Step by Step

Follow these detailed steps to create your Debian 13 LXC container for Stremio server deployment.

## Container Creation Steps

1. **Navigate to Storage Location**
   - Go to `local` storage (not local-lvm)

2. **Access Container Templates**
   - Click on **CT templates**
   - Click on **Templates**

3. **Download Debian 13 Template**
   - Download `debian-13-standard` template

4. **Access Proxmox Node**
   - Click on your Proxmox node

5. **Create Container**
   - Click on **Create CT**

6. **Basic Configuration**
   - **CT ID**: Assign a unique container ID (e.g., 100, 101, etc.)
   - **Hostname**: `stremio-service` (recommended)
   - **Password**: Set a secure password

7. **Template Selection**
   - **Template Storage**: Select storage where template was downloaded (`local`)
   - **Template**: Choose `debian-13-standard` template

8. **Storage Configuration**
   - **Root Disk Storage**: Select storage location for container
   - **Disk Size**: 15-20 GB recommended

9. **Hardware Resources**
   - **Cores**: 1 CPU core
   - **Memory**: 2048 MB RAM
   - **Swap**: 2048 MB swap space

10. **Network Configuration**
    - **IPv4**: DHCP
    - **IPv6**: DHCP

11. **DNS Configuration**
    - Click **Next** to use default DNS settings
    - Click **Finish** to complete container creation

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

## 🐳 Installing Docker and Docker Compose

You'll need Docker and Docker Compose to easily run the Stremio server.

### 1. Install Docker

Execute the following commands to download and run the official Docker installation script:

```bash
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sh get-docker.sh
```

## Step 1: Download Debian 13 Template
<img width="506" height="252" alt="image" src="https://github.com/user-attachments/assets/37222109-349b-48fd-97e3-66b36e91f435" />
