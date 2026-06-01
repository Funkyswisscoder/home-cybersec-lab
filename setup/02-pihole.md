# 02 — Pi-hole

**Host:** Raspberry Pi 4B — 192.168.128.10  
**Version:** Pi-hole v6 (FTL integrated web server)

## Rationale

DNS-layer filtering stops threats before a TCP connection is established. By advertising Pi-hole as the DNS server via DHCP, all devices on the network are covered without any per-device agent or configuration.

## Installation

```bash
curl -sSL https://install.pi-hole.net | bash
```

**Key installer choices:**

| Setting | Value |
|---|---|
| Interface | eth0 / wlan0 |
| Upstream DNS | Cloudflare (1.1.1.1) |
| Admin web interface | Enabled |
| Query logging | Enabled |

## Network-wide Coverage

Pi-hole is advertised to all DHCP clients by configuring the ISP router's DNS field:

```
DHCP DNS Server: 192.168.128.10
```

All devices receive Pi-hole as their DNS resolver on lease renewal.

## Port Note (Pi-hole v6)

Pi-hole v6 uses an integrated FTL web server instead of lighttpd. Default ports:
- `80` — HTTP admin
- `443` — HTTPS admin

The HTTPS port was remapped to `8443` to free port 443 for Wazuh:

```toml
# /etc/pihole/pihole.toml
[webserver]
port = "80o,8443s"
```

## Dashboard

```
http://192.168.128.10/admin/
```

## Verification

```bash
dig google.com @192.168.128.10
# Expected: SERVER: 192.168.128.10#53
```

## Security Controls

| Control | Framework Reference |
|---|---|
| DNS-layer threat blocking | CIS Control 9.2 · NIST SC-20 |
| Query logging | CIS Control 8.2 · NIST AU-12 |
