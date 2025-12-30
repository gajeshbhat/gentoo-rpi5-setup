# Docker & Boot Optimization for Pi 5 (Gentoo)

This document outlines the steps taken to reduce the system boot time from **14.2s** to **9.4s** by decoupling Docker from unnecessary network dependencies and streamlining the `systemd` startup chain.

## Performance Gains

| Component | Baseline (Stock) | Optimized | Difference |
| --- | --- | --- | --- |
| **Kernel** | 2.77s | 2.78s | +0.01s |
| **Userspace** | 11.50s | 6.62s | **-4.88s** |
| **Total Boot** | **14.27s** | **9.41s** | **-4.86s** |

---

## Optimization Steps

### 1. Remove Network-Online Wait

By default, `systemd` waits for a full DHCP/IP handshake before starting Docker. For a local MongoDB setup, this is unnecessary.

```bash
# Mask the wait service to prevent it from blocking the boot chain
sudo systemctl mask systemd-networkd-wait-online.service

```

### 2. Decouple Docker from Network Targets

Ensure the Docker daemon starts as soon as the network stack is initialized, rather than waiting for an active internet connection.
**Command:** `sudo systemctl edit docker.service`

```ini
[Unit]
After=network.target
Wants=network.target

```

### 3. Streamline Docker Daemon (`daemon.json`)
Disabled unnecessary networking features (iptables, bridges) that add latency during the `dockerd` initialization phase.
**File:** `/etc/docker/daemon.json`

```json
{
  "storage-driver": "overlay2",
  "iptables": false,
  "ip-forward": false,
  "userland-proxy": false
}

```

### 4. Optimize Container Network Mode (MongoDB)
Switched the `mongodb` service from `bridge` mode to `host` mode. This eliminates the overhead of creating virtual interfaces (`docker0`) and performing NAT translation. Kindly refer to docker documentation for more details

```yaml
services:
  mongodb:
    network_mode: "host"  # Direct access to localhost:27017
    # ... rest of config

```

---

## Diagnostic Commands
Use these to verify that the optimizations remain effective after system updates:

* `systemd-analyze blame` — Identify slow individual services.
* `systemd-analyze critical-chain` — Visualize the boot bottleneck.
* `pi-status` — Custom script for high-level telemetry. Check the root of this directory to setup this telemetry script

---

### Future Improvements

* **Kernel Built-ins:** Transition `overlay`, `bridge`, and `veth` from modules (`<M>`) to built-in (`<*>`) to shave ~1s off `containerd` initialization.
* **VConsole Disable:** If running headless, `systemctl disable systemd-vconsole-setup.service` can save another ~300ms.
