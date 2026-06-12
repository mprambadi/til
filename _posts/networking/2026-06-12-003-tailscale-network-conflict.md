---
title: "Tailscale + VPN + Docker Network Conflict"
date: 2026-06-12
category: networking
tags: [tailscale, vpn, docker, networking, openfortivpn]
summary: "Troubleshooting Tailscale, OpenFortiVPN, and Docker network conflicts with subnet routing"
---

# Tailscale + OpenFortiVPN + Docker Network Conflict Troubleshooting

## Problem

Connected to FortiVPN using `openfortivpn` and could not access internal resources:

```bash
nc -vz 10.10.0.137 443
```

The connection timed out even though:

- VPN was connected successfully.
- Routes to internal networks existed.
- DNS resolution through VPN worked.
- Traffic was leaving through the VPN interface.

---

## Initial Symptoms

### VPN Connected

```bash
ip addr show ppp0
```

Output:

```text
inet 172.21.0.2 peer 169.254.2.1/32 scope global ppp0
```

### Route Exists

```bash
ip route get 10.10.0.137
```

Output:

```text
10.10.0.137 dev ppp0 src 172.21.0.2
```

### TCP Connection Fails

```bash
nc -vz -w 5 10.10.0.137 443
```

Output:

```text
timed out
```

---

## Verifying VPN Traffic

Monitor VPN interface:

```bash
sudo tcpdump -ni ppp0 host 10.10.0.137
```

Observed:

```text
SYN  -> 10.10.0.137
SYN  -> 10.10.0.137
SYN-ACK <- 10.10.0.137
SYN  -> 10.10.0.137
```

Important observation:

- Remote host replied with SYN-ACK.
- Local machine never completed TCP handshake with ACK.

This proved:

- VPN routing worked.
- Remote server was reachable.
- Problem was local.

---

## Discovery: Docker Network Conflict

Check local networks:

```bash
docker network inspect nginx-proxy-manager_default
```

Result:

```json
{
  "Subnet": "172.21.0.0/16",
  "Gateway": "172.21.0.1"
}
```

Container:

```json
{
  "IPv4Address": "172.21.0.2/16"
}
```

VPN interface:

```text
172.21.0.2
```

Conflict:

```text
VPN Address       : 172.21.0.2
Container Address : 172.21.0.2
```

The VPN endpoint and Docker container had the same IP.

---

## Confirmation

Stop Docker:

```bash
sudo systemctl stop docker
```

Immediately:

```bash
telnet 10.10.0.137 443
```

Output:

```text
Connected to 10.10.0.137
```

Root cause confirmed.

---

## Fix

### Change Docker Network Subnet

Docker Compose:

```yaml
networks:
  default:
    ipam:
      config:
        - subnet: 172.30.10.0/24
```

Recreate network:

```bash
docker compose down
docker network rm nginx-proxy-manager_default
docker compose up -d
```

Verify:

```bash
docker network inspect nginx-proxy-manager_default
```

Expected:

```text
172.30.10.0/24
```

instead of:

```text
172.21.0.0/16
```

---

## Prevent Future Docker Conflicts

Create:

```bash
sudo nano /etc/docker/daemon.json
```

Contents:

```json
{
  "default-address-pools": [
    {
      "base": "172.30.0.0/16",
      "size": 24
    }
  ]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

This prevents Docker from automatically allocating VPN-overlapping subnets.

---

# Headscale / Tailscale Subnet Router Troubleshooting

## Problem

Headscale showed:

```text
Approved
10.10.0.0/24
```

but:

```text
Available
(empty)
```

Clients could not access VPN networks through Tailscale.

---

## Root Cause

Check:

```bash
tailscale debug prefs
```

Found:

```json
"AdvertiseRoutes": null
```

Meaning:

- Headscale had approved routes.
- Pop-OS was no longer advertising them.

---

## Fix

Run:

```bash
sudo tailscale up \
  --login-server=https://headscale.dev.sev-2.com \
  --accept-routes \
  --advertise-routes=10.10.0.0/24,10.10.108.0/24,10.10.111.0/24,10.10.121.0/24,10.10.127.0/24,192.168.10.0/24
```

---

## Verification

Headscale before:

```text
Approved         Available
10.10.0.0/24
```

Headscale after:

```text
Approved         Available         Serving
10.10.0.0/24     10.10.0.0/24     10.10.0.0/24
10.10.108.0/24   10.10.108.0/24   10.10.108.0/24
10.10.111.0/24   10.10.111.0/24   10.10.111.0/24
10.10.121.0/24   10.10.121.0/24   10.10.121.0/24
10.10.127.0/24   10.10.127.0/24   10.10.127.0/24
192.168.10.0/24  192.168.10.0/24  192.168.10.0/24
```

Subnet routing became active.

---

## Ensure Forwarding

Verify:

```bash
sysctl net.ipv4.ip_forward
```

Expected:

```text
net.ipv4.ip_forward = 1
```

Persist:

```bash
sudo tee /etc/sysctl.d/99-tailscale-router.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF

sudo sysctl --system
```

---

## Lessons Learned

1. If VPN traffic receives SYN-ACK but TCP never establishes, suspect local networking conflicts.
2. Docker bridge networks can silently overlap VPN-assigned subnets.
3. Stopping Docker is a quick way to confirm routing/IP conflicts.
4. Always inspect Docker subnets when VPN connectivity behaves strangely.
5. Headscale route approval does not mean routes are being advertised.
6. Check `AdvertiseRoutes` whenever subnet routing stops working.
7. Use explicit Docker address pools to avoid future overlap with VPN networks.
8. Verify packet flow with `tcpdump` before changing routes or firewall rules.
9. A successful VPN connection does not guarantee usable routing.
10. `headscale node list-routes` is the fastest way to verify subnet router health.
