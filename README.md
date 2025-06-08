# 🌐 Secure Home Lab: WireGuard VPN Tunnel & Apache Web Server

## Project Overview

This project documents the creation and configuration of a robust home lab environment. It demonstrates the setup of a secure **WireGuard VPN tunnel** between a **Raspberry Pi** and a **Linux Virtual Machine**, followed by the deployment of a **personal website** on the Raspberry Pi, serving as a fully functional web server.

This lab showcases practical skills in:
* Network architecture and configuration
* Virtualization and remote server management
* Secure tunneling protocols (VPNs)
* Web server deployment and hosting
* System administration and troubleshooting

## Technologies Used

* **Hardware:** Raspberry Pi 4
* **Operating Systems:**
  * **Raspberry Pi OS (64-bit):** Running on the Raspberry Pi
  * **Ubuntu Server (LTS):** Running as a Virtual Machine
* **Virtualization Software:** Oracle VM VirtualBox
* **Networking:**
  * **WireGuard:** Modern, fast, and secure VPN protocol
  * **SSH (Secure Shell):** For remote command-line access
  * **VNC (Virtual Network Computing):** For remote graphical desktop access (on Pi)
  * **IPTables:** Linux firewall for network address translation (NAT) and packet filtering
* **Web Server:** Apache2 HTTP Server
* **Version Control:** Git & GitHub

## Lab Architecture

The setup establishes a segregated and secure network environment within a home LAN.

* **Physical Network (Home LAN):** All devices (Raspberry Pi, Windows Host PC, Ubuntu VM via Bridged Adapter) are connected to the home router, receiving IPs in the `10.0.0.0/24` range.
  * Raspberry Pi physical IP: `10.0.0.235`
  * Ubuntu VM physical IP: `10.0.0.85`
* **WireGuard VPN Tunnel Network:** A private, encrypted overlay network is created between the Raspberry Pi and the Ubuntu VM, using the `10.10.10.0/24` range.
  * Raspberry Pi WireGuard IP: `10.10.10.1`
  * Ubuntu VM WireGuard IP: `10.10.10.2`

## Setup & Configuration Steps

This section provides a high-level overview of the implementation process.

### 1. Raspberry Pi Initial Setup
* Flashed Raspberry Pi OS and configured headless access via SSH and VNC.
* **Screenshot: SSH Connection to Raspberry Pi**
  * `![SSH Connection to Raspberry Pi](screenshots/[YOUR_PI_SSH_SCREENSHOT_FILENAME].PNG)`
* **Screenshot: VNC Access to Raspberry Pi Desktop**
  * `![VNC Access to Raspberry Pi Desktop](screenshots/[YOUR_PI_VNC_SCREENSHOT_FILENAME].PNG)`

### 2. Ubuntu Server VM Setup
* Created a new Virtual Machine in VirtualBox with Ubuntu Server LTS.
* Crucially configured the VM's network adapter to **Bridged Mode** to allow direct communication with the physical home network and the Raspberry Pi.
* **Screenshot: VirtualBox VM Settings Summary**
  * `![Ubuntu VM Settings Summary](screenshots/Ubuntu_VM_Settings.PNG)`
* **Screenshot: VirtualBox Network Adapter Configuration (Bridged Mode)**
  * `![Ubuntu VM Network Settings - Bridged Adapter](screenshots/Ubuntu_Network_Settings.PNG)`
* Configured SSH for remote command-line access to the VM.
* **Screenshot: SSH Connection to Ubuntu Server VM**
  * `![SSH Connection to Ubuntu Server VM](screenshots/[YOUR_VM_SSH_SCREENSHOT_FILENAME].PNG)`

## WireGuard VPN Tunnel Configuration

WireGuard was installed and configured on both the Raspberry Pi and the Ubuntu VM. Each peer has its unique public/private key pair and an IP within the `10.10.10.0/24` tunnel network.

* IP forwarding was enabled on the Raspberry Pi for proper routing of tunnel traffic.
* `PostUp` and `PostDown` rules were configured on the Raspberry Pi to manage NAT for traffic leaving the tunnel onto the local wireless interface (`wlan0`).

### Raspberry Pi WireGuard Configuration (`configs/pi_wg0.conf`)
```ini
# Content of /etc/wireguard/wg0.conf on Raspberry Pi
[Interface]
PrivateKey = [HIDDEN]
Address = 10.10.10.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o wlan0 -j MASQUERADE

[Peer]
PublicKey = tzzMPnamJCUQaBZkHyRJ7LmQcl3/6bsLaTgZpPicAno= # Ubuntu VM's Public Key
AllowedIPs = 10.10.10.2/32

### Ubuntu VM WireGuard Configuration (`configs/ubuntu_vm_wg0.conf`)

* The full configuration for the Ubuntu VM's WireGuard interface can be found in `configs/ubuntu_vm_wg0.conf` within this repository.
* **Screenshot: `sudo wg show` on Ubuntu VM (showing active tunnel interface and peer)**
    * `![WireGuard Status on Ubuntu VM](screenshots/Ubuntu_wg_ping.PNG)`

## Tunnel Connectivity Verification

Successful `ping` tests were performed in both directions across the WireGuard tunnel, demonstrating full connectivity between the `10.10.10.x` tunnel IPs.

* **Screenshot: Successful Ping from Raspberry Pi to Ubuntu VM (over tunnel)**
    * `![Ping from Pi to VM (0% Packet Loss)](screenshots/Pi_ping.jpg)`
* **Screenshot: Successful Ping from Ubuntu VM to Raspberry Pi (over tunnel)**
    * `![Ping from VM to Pi (0% Packet Loss)](screenshots/Ubuntu_wg_ping.PNG)`

## Web Server Deployment

An Apache2 HTTP web server was deployed on the Raspberry Pi, and a personal website (from my GitHub repository: [**Link to your website repo if separate**]) was configured to be served from it.

* **Screenshot: Apache2 Service Status on Raspberry Pi (active/running)**
    * `![Apache2 Service Status](screenshots/PI_wg.PNG)`
* **Screenshot: Website Access from Ubuntu VM (via `curl` over tunnel)**
    * `![Website Access via Curl (Over WireGuard Tunnel)](screenshots/Ubuntu_SSH_Apache.PNG)`

## Key Learnings & Challenges

This project provided invaluable hands-on experience and presented several opportunities for in-depth learning and problem-solving:

* **Navigating Dependency Conflicts and Legacy Software:**
    * Initially, the project aimed to utilize `znail` for network tunneling. However, repeated "ModuleNotFoundError" and other compatibility issues arose due to its reliance on outdated Python dependencies, which proved challenging to resolve in a modern Python 3.11 environment.
    * **Learning:** This experience highlighted that modern software is not always backward-compatible with legacy solutions. It reinforced the importance of **adaptability** in project planning and the value of researching **alternative technologies** (like WireGuard) when an initial approach becomes an insurmountable obstacle.

* **Understanding Linux Network Forwarding and Persistence:**
    * Encountered "Destination Host Unreachable" errors when initially bringing up the WireGuard tunnel on the Raspberry Pi. This was traced back to two core issues: the `iptables` command not being found (requiring its installation) and the kernel's IP forwarding (`/proc/sys/net/ipv4/ip_forward`) being disabled (`0`).
    * **Learning:** This forced a deeper understanding of **Linux kernel networking parameters**, specifically how `ip_forward` enables packet routing between interfaces. It also emphasized the critical step of ensuring network configurations and services (`wg-quick@wg0`) are configured to **start persistently on boot** (`systemctl enable`) for a stable lab environment.

* **Precision in Configuration and Cryptographic Key Management:**
    * During WireGuard setup on both the Pi and the VM, I encountered errors such as "Key is not the correct length or format" and `wg0.conf` file not found.
    * **Learning:** These issues were resolved by meticulously reviewing the configuration files in `nano` and identifying subtle typos or incorrect formatting (e.g., missing characters, extra spaces, incorrect key formats, or improper saving of files). This underscored the paramount importance of **attention to detail** in network configuration and the **strictness of cryptographic key formats** required by secure protocols like WireGuard.

* **Practical Network Access and Troubleshooting:** This project significantly improved my comfort with remote command-line access (SSH) and graphical desktop access (VNC) for Linux systems. It also solidified my ability to distinguish between private tunnel IPs and physical network IPs, and effectively troubleshoot network connectivity issues.

## Future Enhancements

Future plans to expand this lab and further enhance skills include:

* **Full Tunnel VPN:** Configure the WireGuard tunnel to route all internet traffic from the Ubuntu VM through the Raspberry Pi, creating a secure **Browse experience**.
* **Local DNS Server:** Set up a local DNS resolver (e.g., Pi-hole or Unbound) on the Raspberry Pi for network-wide ad-blocking and custom domain resolution.
* **Additional Peers:** Expand the WireGuard network by adding more client devices (e.g., a laptop, another VM, or even a mobile phone).
* **HTTPS Implementation:** Secure the Apache web server with HTTPS (SSL/TLS) using Let's Encrypt certificates (for scenarios where public access might be considered).
* **Advanced Firewalling:** Explore more sophisticated firewall rules using `nftables` for finer-grained control over network traffic.