# Nginx Troubleshooting Lab

## Objective

Practice diagnosing and recovering from a failed Linux service caused by an invalid configuration.

## Scenario

Nginx was installed and running normally on the Ubuntu server. I deliberately introduced an invalid directive into:

`/etc/nginx/nginx.conf`

The invalid directive caused Nginx configuration validation and service startup to fail.

## Investigation

Before applying the configuration, I tested it with:

```
sudo nginx -t
```

Nginx reported a syntax/configuration error and identified the affected configuration file and line.

I then attempted to restart the service:

```
sudo systemctl restart nginx
```

The restart failed.

I checked the service state with:

```
systemctl status nginx
```

and reviewed recent Nginx logs with:

```
journalctl -u nginx --since "5 minutes ago"
```

These confirmed that Nginx could not start because its configuration was invalid.

## Root Cause

An unsupported directive had been added to the Nginx configuration file.

Because Nginx validates its configuration before starting, the invalid directive prevented the service from starting successfully.

## Resolution

I removed the invalid directive and retested the configuration:

```
sudo nginx -t
```

After the configuration test succeeded, I restarted Nginx:

```
sudo systemctl restart nginx
```

I then verified the service:

```
systemctl status nginx
curl http://localhost
```

Nginx returned to an active state and successfully served HTTP requests.

## Rollback Strategy

Before modifying the configuration, I created a backup:

```
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

If the configuration could not be repaired quickly, the known-good file could be restored and tested before restarting the service.

## Key Takeaways

* Validate configuration changes before restarting a service.
* `nginx -t` tests Nginx configuration directly.
* `systemctl status` shows the service's current state and recent failure information.
* `journalctl` provides more detailed historical logs.
* Backing up known-good configuration files provides a simple rollback path.
* Troubleshooting is more effective when the root cause is identified before changes are made.

## Additional Observation — Reload vs Restart

While testing Nginx listening addresses, I changed the server from listening on all IPv4 interfaces:

```
0.0.0.0:80
```

to listening only on loopback:

```
127.0.0.1:80
```

The configuration passed:

```
sudo nginx -t
```

However, after:

```
sudo systemctl reload nginx
```

the listening socket did not immediately reflect the expected configuration change.

A full restart:

```
sudo systemctl restart nginx
```

caused Nginx to bind and listen to the expected address.

I observed the same behavior when changing the listening address back.

### Lesson

A syntactically valid configuration is not necessarily guaranteed to be applied successfully through a graceful reload.

Changes involving listening sockets can behave differently from ordinary configuration changes because the running process must acquire or replace network resources while existing workers may still be active.

I learned to verify runtime state after configuration changes using:

```
sudo nginx -t
sudo systemctl reload nginx
sudo ss -ltnp
journalctl -u nginx
```

and to use a controlled restart when a listening-address change does not apply as expected.
