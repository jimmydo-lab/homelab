# Homelab

This repository documents my hands-on learning and projects in Linux system administration, networking, virtualization, remote access, security, and infrastructure technologies.

My goal is to develop practical IT skills by building and configuring systems, troubleshooting unexpected behavior, and documenting both successful implementations and lessons learned.

## Lab Environment

| Component | Technology |
|---|---|
| Host OS | Windows |
| Hypervisor | VMware Workstation Pro |
| Server OS | Ubuntu Server LTS (26.04) |
| Remote Administration | SSH from macOS |
| Remote Networking | Tailscale |
| Network Hardware | GL.iNet Beryl AX (GL-MT3000) |
| Web Server | NGINX |
| Firewall | UFW, iptables |

## Projects & Labs

### 1. Remote Wake-on-LAN with Tailscale

**Technologies:** OpenWrt, GL.iNet Beryl AX, Tailscale, SSH, Wake-on-LAN, Windows

Implemented a remote power-management solution that allows my Windows desktop to be powered on from a complete shutdown (S5) over the internet.

**Key accomplishments:**
- Configured a GL.iNet Beryl AX router as an always-on Wake-on-LAN controller using a Wi-Fi-to-Ethernet connection.
- Established secure remote administration through Tailscale and SSH key authentication.
- Automated remote power-on using a macOS shell function and a one-tap iOS Shortcut.
- Diagnosed SSH authentication failures using Dropbear system logs.
- Verified that remote wake functionality persists after restarting the router.

**[View project documentation](networking/remote-wake-on-lan.md)**

### 2. Linux User and File Permissions

**Technologies:** Ubuntu Server, Linux CLI, SSH

Created multiple Linux users and groups to investigate file ownership, permission management, and shared directory behavior.

**Key accomplishments:**
- Practiced user and group management, including `chmod`, `chown`, and `chgrp`.
- Explored file and directory permissions, ownership, and directory traversal.
- Investigated how `umask` affects newly created files and directories.
- Configured shared directories using setgid for group inheritance.
- Tested sticky-bit behavior and file deletion permissions.

**[View lab documentation](linux/permissions.md)**

### 3. NGINX Configuration and Troubleshooting

**Technologies:** Ubuntu Server, NGINX, Linux CLI

Deployed an NGINX web server and investigated the effects of configuration changes on its network accessibility.

**Key accomplishments:**
- Installed and configured NGINX on Ubuntu Server.
- Modified server listening addresses to control network exposure.
- Validated configuration changes using `nginx -t`.
- Compared NGINX reload and restart behavior.
- Investigated cases where configuration changes did not immediately affect active listening sockets.

**[View troubleshooting documentation](linux/nginx-troubleshooting.md)**

### 4. Firewall and Tailscale Troubleshooting

**Technologies:** UFW, iptables, Tailscale, NGINX

Investigated unexpected network accessibility when configuring firewall policies on an Ubuntu Server VM.

**Key accomplishments:**
- Configured UFW with a default-deny incoming policy.
- Examined Tailscale-managed iptables chains and packet counters.
- Investigated why HTTP appeared accessible despite restrictive firewall rules.
- Used NGINX access logs and connection tests to validate firewall behavior.
- Identified browser caching as a factor in misleading initial testing results.

**[View troubleshooting documentation](linux/firewall-tailscale-troubleshooting.md)**

## Skills & Technologies

| Category | Technologies & Concepts |
|---|---|
| Operating Systems | Linux (Ubuntu Server), Windows, macOS |
| Virtualization | VMware Workstation Pro |
| Networking | TCP/IP, SSH, Tailscale, Wake-on-LAN, network interfaces |
| System Administration | Users, groups, permissions, services, file ownership |
| Web Services | NGINX |
| Security | SSH key authentication, UFW, iptables, access control |
| Automation | Shell commands, macOS shell functions, iOS Shortcuts |
| Version Control | Git, GitHub |
| Troubleshooting | System logs, service status, packet counters, connectivity testing |

## Learning Roadmap

This homelab is an ongoing learning environment. Future areas of exploration include:

- **Linux administration:** Deeper system management, scheduled tasks, and service configuration.
- **Networking:** DNS, DHCP, routing, subnetting, and network troubleshooting.
- **Automation:** Bash and Python scripting for system administration.
- **Containers:** Docker, container networking, and service deployment.
- **Monitoring:** System metrics, centralized logging, and alerting.
- **Infrastructure & Cloud:** Infrastructure automation, cloud computing, and deployment workflows.

## Learning Approach

My approach emphasizes hands-on experimentation rather than theoretical knowledge alone:

1. **Build:** Deploy and configure systems to understand how they operate.
2. **Experiment:** Modify configurations and observe their effects.
3. **Troubleshoot:** Investigate failures using logs, diagnostic commands, and systematic testing.
4. **Document:** Record configurations, findings, mistakes, and solutions for future reference.

The objective is to develop the technical knowledge and practical troubleshooting skills needed for IT support, systems administration, and infrastructure roles.