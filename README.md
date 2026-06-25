# T-Pot Honeypot Deployment & Troubleshooting

This repository documents my hands-on journey of deploying a T-Pot Honeypot platform on a Virtual Private Server (VPS). A honeypot is a controlled security mechanism set up as a decoy to lure cyberattackers, allowing security analysts to study their methodologies and gather actionable threat intelligence.

---

## Project Overview

The objective of this project was to deploy the T-Pot Community Edition on a cloud-hosted server, configure network security to allow incoming attack traffic while securely locking down the management backend, and resolve real-world deployment hurdles related to automation and SSL certificate corruption.

### Technical Stack
* **VPS Provider:** Vultr
* **Platform:** T-Pot Community Edition
* **OS:** Debian / Ubuntu
* **Core Technology:** Docker, Nginx, Kibana, OpenSSL, Ansible

---

## Deployment Process

### Phase 1: Infrastructure & Provisioning
I provisioned a fresh Linux VPS via Vultr. To ensure the honeypot was fully exposed to the public internet to attract malicious activity—while remaining strictly secure for my own administrative access, I configured the firewall rules as follows:

| Action | Protocol | Port / Range | Source | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Accept** | TCP | `1:65535` | `0.0.0.0/0` | Attract Attackers (All Ports) |
| **Accept** | UDP | `1:65535` | `0.0.0.0/0` | Attract Attackers (All Ports) |
| **Accept** | TCP | `64294:64297` | `my personal IP` | Secure Admin & WebGUI Access |

### Phase 2: Installation
Because the automated T-Pot installation script is restricted from running directly as the `root` user, I created a dedicated deployment user with sudo privileges:

```bash
sudo adduser logan
cd /home/logan
git clone [https://github.com/telekom-security/tpotce](https://github.com/telekom-security/tpotce)
