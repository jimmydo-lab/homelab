# UFW and Tailscale Firewall Troubleshooting

## Scenario

I configured UFW with a default policy of denying incoming traffic and initially allowed HTTP traffic on TCP port 80.

After confirming Nginx was remotely accessible, I removed the port 80 rule:

```bash
sudo ufw delete allow 80/tcp
```

I expected remote HTTP access to fail.

However, my MacBook could still access Nginx through the Ubuntu server's Tailscale IPv4 address.

## Initial Verification

UFW showed:

```text
Status: active
Default: deny (incoming), allow (outgoing)

22/tcp (OpenSSH)    ALLOW IN    Anywhere
```

There was no rule allowing TCP port 80 and no visible rule allowing the `tailscale0` interface.

Nginx was still running and listening on port 80.

## Investigation

Because UFW did not explain the behavior, I inspected the underlying Linux firewall rules:

```bash
sudo iptables -L ts-input -n -v --line-numbers
```

The Tailscale chain contained:

```text
ACCEPT all -- tailscale0 * 0.0.0.0/0 0.0.0.0/0
```

This rule accepted all traffic entering through the `tailscale0` interface.

Its packet and byte counters also showed that the rule was actively matching traffic.

## Root Cause

Tailscale had installed its own netfilter/iptables rules independently of the rules displayed by UFW.

Traffic arriving through `tailscale0` was accepted by Tailscale's firewall chain, so removing UFW's explicit TCP port 80 rule did not prevent HTTP access over Tailscale.

## Key Takeaways

- `ufw status` does not necessarily show every firewall rule affecting a Linux machine.
- UFW is a frontend for Linux packet filtering; other software can install its own netfilter rules.
- Interface-specific firewall rules can be broader than port-specific rules.
- Packet counters in `iptables` are useful evidence that a rule is actively matching traffic.
- When observed network behavior does not match UFW configuration, inspect the underlying firewall state before assuming the service or application is misconfigured.

This reinforced the importance of verifying the entire packet-processing path instead of relying on a single management tool.

