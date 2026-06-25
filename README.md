T-Pot Honeypot Deployment & Troubleshooting

This repository documents my journey of deploying a T-Pot Honeypot platform on a VPS. A honeypot is a security mechanism set up to be a decoy to lure in cyberattackers, allowing us to study their methods and gather threat intelligence.
Project Overview

The goal was to deploy the T-Pot Community Edition on a cloud-hosted server, configure network security to allow attack traffic while protecting the management backend, and resolve installation hurdles related to automation and SSL certificates.
Technical Stack

    VPS Provider: Vultr
    Platform: T-Pot Community Edition
    OS: Debian/Ubuntu
    Core Tech: Docker, Nginx, Kibana, OpenSSL

Deployment Process
Phase 1: Infrastructure & Provisioning

I started by provisioning a VPS through Vultr. To ensure the honeypot was exposed to the internet (to attract attacks) but secure for my own management, I configured the firewall as follows:
Action	Protocol	Port / Range	Source	Purpose
Accept	TCP	1:65535	0.0.0.0/0	Attract Attackers
Accept	UDP	1:65535	0.0.0.0/0	Attract Attackers
Accept	TCP	64294:64297	[YOUR_MANAGEMENT_IP]	Secure Admin Access
Phase 2: Installation

Since the T-Pot installation script cannot be run directly as the root user, I created a dedicated user for the deployment:

sudo adduser logan
cd /home/logan
git clone https://github.com/telekom-security/tpotce

Troubleshooting & Solutions

During the setup, I encountered two major roadblocks that required deep diving into the logs and system configuration.
Issue 1: Ansible Playbook "Premature End of Stream"

While running the installation, the playbook failed with the following error:
sudo: interactive authentication is required

The Problem:
The automated installer attempted to execute tasks via sudo, but the system was prompting for a password. Because the playbook is non-interactive, it couldn't provide the password, causing the process to crash.

The Fix:
I modified the sudoers file to allow my user to execute commands without a password prompt, which allowed the Ansible playbook to complete its tasks seamlessly.
Issue 2: WebGUI Access & SSL Corruption

Even after the services showed as active via systemctl status tpot, I was unable to access the WebGUI. I spent over an hour restarting servers and re-configuring the firewall, only to discover that the SSL certificates Nginx was expecting had become corrupted during the initial download.

The Fix:
To restore access, I manually generated a fresh set of self-signed SSL certificates.

    Create the certificate directory:

    sudo mkdir -p /home/logan/tpotce/data/nginx/cert

    Generate the certificates using OpenSSL:
    The following command created a self-signed certificate valid for 365 days:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /home/logan/tpotce/data/nginx/cert/nginx.key \
    -out /home/logan/tpotce/data/nginx/cert/nginx.crt \
    -subj "/C=US/ST=State/L=City/O=Organization/OU=Unit/CN=localhost"

    Set permissions and restart:
    I updated permissions to ensure the Nginx container could read the files, then restarted the T-Pot service:

    sudo chmod -R 777 /home/logan/tpotce/data/nginx/cert
    sudo systemctl restart tpot

Final Results & Observations

After applying the SSL fix, the WebGUI became fully operational. I now have access to several powerful monitoring dashboards:

    Attack Map: Real-time visualization of where attacks are originating globally.
    Kibana: Deep-dive analysis of the logs and attack vectors.
    Spiderfoot: OSINT automation to investigate the attackers.

Initial Findings

Within minutes of going live, the honeypot began capturing traffic. As seen in the captured logs, I observed multiple "Known Attackers" and "Mass Scanners" hitting various ports, primarily originating from global botnets. This highlights the constant background noise of the internet and the frequency of automated scanning.
