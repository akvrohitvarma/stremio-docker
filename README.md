# 🚀 Ultimate Stremio Server Setup with NordVPN & Cloudflare Tunnel

A complete, production-ready guide to deploying a private Stremio streaming server with enterprise-level privacy and remote access capabilities using Proxmox, Docker, NordVPN, and Cloudflare Tunnel.

## 🎯 What This Repository Provides

This comprehensive guide helps you build a **secure, private streaming server** that you can access from anywhere while maintaining complete anonymity through NordVPN protection.

# Technologies Used

## Core Technologies
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

## Step 1: Download Debian 13 Template
<img width="506" height="252" alt="image" src="https://github.com/user-attachments/assets/37222109-349b-48fd-97e3-66b36e91f435" />
