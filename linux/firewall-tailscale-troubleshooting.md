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

There was no UFW rule allowing TCP port 80 and no visible rule allowing the `tailscale0` interface.

Nginx was still running and listening on port 80.

## Investigation

Because the UFW configuration did not explain the behavior, I inspected the underlying Linux firewall rules:

```bash
sudo iptables -L ts-input -n -v --line-numbers
```

The Tailscale chain contained:

```text
ACCEPT all -- tailscale0 * 0.0.0.0/0 0.0.0.0/0
```

This rule accepted traffic entering through the `tailscale0` interface.

Its packet and byte counters also showed that the rule was actively matching traffic.

## Root Cause

Tailscale had installed its own netfilter/iptables rules independently of the rules displayed by UFW.

Traffic arriving through `tailscale0` was accepted by Tailscale's firewall chain, so removing UFW's explicit TCP port 80 rule did not prevent HTTP access over Tailscale.

## Verifying the Firewall Behavior

To test the behavior more directly, I inserted a temporary rule at the beginning of the `INPUT` chain:

```bash
sudo iptables -I INPUT 1 -i tailscale0 -p tcp --dport 80 -j DROP
```

This rule matched:

- Traffic entering through `tailscale0`
- TCP traffic
- Destination port 80

and dropped matching packets before they could reach Nginx.

The rule's packet and byte counters increased when I attempted to access the web server remotely, confirming that HTTP traffic was reaching and matching the firewall rule.

## Browser Cache During Verification

Initially, my MacBook browser continued displaying the Nginx page even after the new firewall rule was matching and dropping port 80 traffic.

This appeared to suggest that the firewall rule was not working.

To remove the browser from the test, I made a fresh request from the MacBook terminal:

```bash
curl -v --max-time 5 http://<TAILSCALE-IP>/
```

The connection timed out.

At the same time, I monitored the Nginx access log:

```bash
sudo tail -f /var/log/nginx/access.log
```

No new request appeared in the log.

Together, these observations showed:

```text
iptables counter increased
→ packets reached the firewall rule

curl timed out
→ the client did not receive an HTTP response

Nginx access log remained unchanged
→ the request never reached Nginx
```

The browser had been displaying previously cached content rather than successfully retrieving a new page from the server.

### Lesson

A browser is not always a reliable tool for verifying network connectivity because cached content and connection reuse can obscure the current state of the network.

For troubleshooting, a fresh `curl` request combined with server-side logs provides stronger evidence.

## Key Takeaways

- `ufw status` does not necessarily show every firewall rule affecting a Linux system.
- UFW is a frontend for Linux packet filtering; other software can install its own netfilter rules.
- Interface-specific firewall rules can be broader than port-specific rules.
- Packet and byte counters help confirm whether traffic is actually matching a firewall rule.
- A healthy service and listening socket do not guarantee remote reachability.
- Browser output alone should not be trusted as proof that a new request reached the server.
- Client-side tools such as `curl` and server-side logs can provide independent evidence during troubleshooting.
- When observed network behavior does not match a high-level firewall configuration, inspect the underlying packet-filtering rules.