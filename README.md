# T-Pot Honeypot Deployment & Troubleshooting

This repository documents my journey of deploying a **T-Pot Honeypot** platform on a VPS. A honeypot is a security mechanism set up as a decoy to lure in cyberattackers, allowing defenders to study their methods and gather threat intelligence.

## Project Overview

The goal was to deploy the T-Pot Community Edition on a cloud-hosted server, configure network security to allow attack traffic while protecting the management backend, and resolve installation hurdles related to automation and SSL certificates.

## Technical Stack

- **VPS Provider:** Vultr
- **Platform:** T-Pot Community Edition
- **OS:** Debian/Ubuntu
- **Core Technologies:** Docker, Nginx, Kibana, OpenSSL

---

## Deployment Process

### Phase 1: Infrastructure & Provisioning

I started by provisioning a VPS through Vultr. To ensure the honeypot was exposed to the internet to attract attacks while still securing administrative access, I configured the firewall as follows:

| Action | Protocol | Port / Range | Source | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| Accept | TCP | `1:65535` | `0.0.0.0/0` | Attract attackers |
| Accept | UDP | `1:65535` | `0.0.0.0/0` | Attract attackers |
| Accept | TCP | `64294:64297` | `[MY IP]` | Secure admin access |

### Phase 2: Installation

Since the T-Pot installation script cannot be run directly as the `root` user, I created a dedicated user for the deployment:

```bash
sudo adduser logan
cd /home/logan
git clone https://github.com/telekom-security/tpotce
