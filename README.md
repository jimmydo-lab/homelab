# Homelab

This repository documents my hands-on learning in Linux system administration, networking, virtualization, troubleshooting, and related infrastructure technologies.

## Lab Environment

* Host OS: Windows
* Hypervisor: VMware Workstation Pro
* Server OS: Ubuntu Server LTS (26.04)
* Remote administration: SSH from macOS
* Remote access: Tailscale Network

## Current Topics

### Linux Administration

* Users and groups
* File and directory ownership
* Linux permissions
* `chmod`, `chown`, and `chgrp`
* `umask`
* Shared directories
* setgid directories
* Sticky bit behavior
* SSH administration

## Labs

### Linux Permissions

Created multiple test users and used them to investigate Linux file and directory permissions.

Topics explored:

* Difference between file permissions and directory permissions
* Directory traversal using the execute (`x`) bit
* Owner, group, and others permission classes
* How `umask` affects newly created files and directories
* Shared group access
* Group inheritance using setgid
* File deletion behavior
* Sticky-bit protected directories

Detailed notes are available in [`linux/permissions.md`](linux/permissions.md).

## Goals

This homelab is being used to build practical experience with:

* Linux system administration
* Networking
* Virtualization
* Troubleshooting
* Containers
* Automation
* Infrastructure and cloud technologies

The focus is on building systems, intentionally breaking them, diagnosing failures, and documenting the troubleshooting process.

