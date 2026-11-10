# Remote Wake-on-LAN with GL.iNet Beryl AX and Tailscale

## Project Overview

Configured a GL.iNet Beryl AX (GL-MT3000) travel router to power on a Windows desktop remotely from a complete shutdown (S5) using Tailscale and Wake-on-LAN (WoL). Added a macOS one-command wake function and an iOS Shortcut for one-tap wake requests. Confirmed the iOS remote wake workflow and functionality after rebooting the Beryl AX.

The desktop hosts a VMware-based Ubuntu Server homelab. The Beryl AX remains on while the desktop is shut down, allowing a remote device to reach the router through Tailscale and request a local Ethernet WoL packet.

## Hardware and Software

- **Desktop motherboard:** ASRock Z690M-ITX/ax
- **Desktop OS:** Windows
- **Home gateway:** ISP-provided Xfinity gateway
- **Always-on router:** GL.iNet Beryl AX (GL-MT3000), OpenWrt-based firmware
- **Remote connectivity:** Tailscale (manually updated on Beryl AX)
- **Wake utility:** `wakeonlan` 0.41-1 (installed via **Applications → Plug-ins**)
- **Clients:** MacBook (macOS SSH) and iPhone (Apple Shortcuts)

## Network Architecture

```text
MacBook / iPhone (away from home)
          |
      Tailscale
          |
GL.iNet Beryl AX (always on)
          |   ^
          |   | Wi-Fi repeater link to Xfinity gateway
          |
  Local Ethernet LAN (WoL magic packet)
          |
ASRock Z690M-ITX desktop (powered off / S5)
```

The Beryl AX connects wirelessly to the Xfinity gateway and by Ethernet to the desktop's WoL-capable network adapter. A remote SSH command is sent to the Beryl AX through Tailscale. The Beryl AX emits the magic packet **locally** on its Ethernet LAN; the magic packet itself does not need to traverse Tailscale.

No inbound public internet port forwarding is required. Normal desktop browsing does not need to use a VPN or a Tailscale exit node.

> **Privacy:** Use placeholders for Tailscale IP addresses, Ethernet MAC addresses and public keys in public documentation. Never commit private SSH keys or credentials.

## 1. Configure the motherboard and Windows

### UEFI/BIOS

On the ASRock Z690M-ITX/ax, under **Advanced → ACPI Configuration**, enable applicable wake settings:

- `I219 LAN Power On` — enabled when using the Intel I219 Ethernet port
- `PCIE Devices Power On` — enabled
- `Deep Sleep` — disabled, if present

Save the BIOS settings and reboot.

### Windows Ethernet adapter

In **Device Manager → Network adapters**, open the properties for the wired Ethernet adapter. In the **Advanced** and **Power Management** tabs, enable supported wake options such as **Wake on Magic Packet**, shutdown wake/PME (if offered), and the permission to wake the computer. Driver options vary.

Disable **Fast Startup** under **Control Panel → Power Options → Choose what the power buttons do** to make shutdown-state troubleshooting more predictable.

Find the **Ethernet** MAC address (not the Wi-Fi MAC address):

```powershell
Get-NetAdapter | Format-Table Name, InterfaceDescription, MacAddress, Status
```

## 2. Configure the Beryl AX

1. Connect the Beryl AX to the Xfinity gateway in **Repeater** mode.
2. Connect the Beryl AX LAN port to the desktop's Ethernet port using a short cable.
3. Enable Tailscale on the Beryl AX. In this project, Tailscale was also manually updated through SSH.
4. Install the OpenWrt `wakeonlan` package (version `0.41-1` used in this project).
5. Confirm the router's LAN interface and broadcast address:

```bash
ip -4 addr show br-lan
command -v wakeonlan
tailscale ip -4
```

For a default `192.168.8.1/24` GL.iNet LAN, the broadcast address is often `192.168.8.255`. **Use the broadcast address that actually works on your router.**

Send a packet from the Beryl AX:

```bash
wakeonlan -i <LAN_BROADCAST_IP> <DESKTOP_ETHERNET_MAC>
```

First test from sleep, then from a full Windows shutdown. In this project, a remote S5 wake was successfully verified.

## 3. Set up SSH public-key authentication for macOS

Create a separate key pair on the MacBook:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/beryl_ax -C "macbook-beryl"
```

Only the **public** key (`~/.ssh/beryl_ax.pub`) is transferred to the router. The private key remains on the MacBook.

To append the public key without overwriting other authorized keys:

```bash
cat ~/.ssh/beryl_ax.pub | ssh root@<BERYL_TAILSCALE_IP> \
  'umask 077; mkdir -p /etc/dropbear; cat >> /etc/dropbear/authorized_keys'
```

Notes:

- `>>` appends; `>` overwrites.
- `/etc/dropbear/authorized_keys` is a **file**, so it must not have a trailing `/`.
- `umask 077` restricts newly created files/directories; it does not rewrite permissions on existing files.
- Ensure each public key is on its **own line**.
- Keep an existing working SSH session open until new key authentication is verified.

Check and, if needed, correct permissions on the router:

```bash
ls -ld /etc/dropbear
ls -l /etc/dropbear/authorized_keys
chmod 600 /etc/dropbear/authorized_keys
```

Test the key from macOS without password fallback:

```bash
ssh -i ~/.ssh/beryl_ax -o IdentitiesOnly=yes \
  -o PreferredAuthentications=publickey -o PasswordAuthentication=no \
  root@<BERYL_TAILSCALE_IP>
```

A key **passphrase** prompt is expected if one was set; it is different from the router account password.

## 4. Create a one-command macOS wake function

Create or edit `~/.ssh/config` on the MacBook:

```sshconfig
Host beryl
    HostName <BERYL_TAILSCALE_IP>
    User root
    IdentityFile ~/.ssh/beryl_ax
    IdentitiesOnly yes
```

Test the SSH host alias:

```bash
ssh beryl
```

In `~/.zshrc`, add:

```bash
wakepc() {
    ssh beryl 'wakeonlan -i <LAN_BROADCAST_IP> <DESKTOP_ETHERNET_MAC>'
}
```

Load the change:

```bash
source ~/.zshrc
```

From then on, while Tailscale is connected on the MacBook:

```bash
wakepc
```

The command uses SSH over Tailscale to tell the Beryl AX to send a local magic packet. Sending a packet is **not** the same as verifying that Windows has finished booting.

## 5. Create a one-tap iOS Shortcut

1. Install and connect **Tailscale** on the iPhone.
2. In Apple **Shortcuts**, create a shortcut named **Wake Desktop**.
3. Add the **Run Script over SSH** action.
4. Configure the action:

   | Field | Value |
   | --- | --- |
   | Host | `<BERYL_TAILSCALE_IP>` |
   | Port | `22` (unless SSH was changed) |
   | **User** | **`root`** |
   | Authentication | **SSH Key** |
   | Script | `wakeonlan -i <LAN_BROADCAST_IP> <DESKTOP_ETHERNET_MAC>` |

5. In the action's SSH key settings, copy the iPhone-generated **public** key.
6. Add that public key as a **separate line** in `/etc/dropbear/authorized_keys` on the router. Preserve existing keys and do not transfer the iPhone's private key.
7. Temporarily set the script to `whoami` and optionally add **Show Result** to confirm the output is `root`.
8. Restore the `wakeonlan` command, test it over Tailscale, and add the shortcut to the iPhone Home Screen or Control Center.

**Verified:** After correcting the missing SSH username, the iPhone Shortcut successfully performed the remote wake workflow.

## 6. Troubleshooting: iPhone Shortcut SSH authentication

**Observed symptom:** Apple Shortcuts displayed an error instructing the user to ensure the SSH server had the device's public key in its authorized keys, despite the public key already being present on a separate line.

To inspect Dropbear logs on the Beryl AX:

```bash
logread -f | grep -i dropbear
```

During the failed iPhone connection, the router logged a message of the form:

```text
Child connection from <TAILSCALE_CLIENT_IP>:<SOURCE_PORT>
Login attempt for nonexistent user from <TAILSCALE_CLIENT_IP>:<SOURCE_PORT>
Exit before auth ... Disconnect received
```

**Diagnosis:** The Tailscale connection and SSH transport were working. Authentication did not proceed because the **User** field was missing from the iOS Shortcuts **Run Script over SSH** action.

**Resolution:** Entered `root` as the SSH username. The Shortcut then worked as intended, without needing to regenerate the keys.

**Lesson:** When key authentication appears to fail, use server logs to distinguish invalid usernames from genuinely rejected SSH keys. Do not assume every client-facing SSH key error is a key-format problem.

## 7. Persistence and reboot validation

The Beryl AX was restarted after configuration, and the user **verified that all required functions continued to work**, including the remote wake workflow. This is an important validation because Tailscale had been updated manually and a separate package was installed.

Useful checks after future firmware upgrades or major configuration changes:

```bash
tailscale version
tailscale status
command -v wakeonlan
```

Also confirm that SSH remains accessible over Tailscale, the authorized keys remain available, and a remote wake test works. A successful router reboot test does **not** guarantee persistence across a firmware reflash or reset, so revisit these checks after firmware updates.

## 8. Security considerations

- Do **not** forward SSH (port 22) or WoL UDP port 9 from the public internet.
- Use Tailscale access controls and keep the router's software updated.
- Prefer SSH public-key authentication and protect private keys.
- Avoid publishing actual MAC/IP addresses and public keys in repository examples.
- SSH as `root` grants broad access to the router; a narrower wake-only interface is a possible future improvement.

## 9. Verification summary

| Test | Status |
| --- | --- |
| Beryl AX configured with Tailscale | Verified |
| `wakeonlan` package installed | Verified |
| Windows desktop wakes from shutdown (S5) | Verified |
| Remote wake over Tailscale | Verified |
| iOS Shortcut using SSH | Verified |
| iOS SSH issue traced to missing `root` username | Resolved |
| Functions persist after Beryl AX reboot | Verified |
| macOS `wakepc` function | Setup documented; independently confirm if desired |

## Lessons learned

This project practiced UEFI/ACPI power configuration, Ethernet WoL, wireless-to-wired networking, Tailscale, SSH public-key authentication, Dropbear logs, shell pipes and append redirection, `umask`, macOS shell functions, iOS automation, and reboot persistence testing.

## Future enhancements

- Confirm and document a successful `wakepc` test from the macOS shell.
- Add an optional PC-online check after the wake request.
- Review opportunities for least-privilege remote wake rather than full root SSH access.
- Test again after Beryl AX firmware upgrades.
