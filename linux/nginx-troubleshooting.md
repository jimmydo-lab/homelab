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
