# 01 — OS Hardening

**Host:** Raspberry Pi 4B  
**OS:** Ubuntu Server 22.04.5 LTS (64-bit)

## Rationale

Ubuntu Server 22.04 LTS was chosen over Raspberry Pi OS for better compatibility with security tooling (Zeek, Wazuh) and longer support lifecycle (2027).

## Steps

### 1. System Update

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

### 2. UFW — Host Firewall

Default deny incoming enforces the principle of least privilege. Only explicitly required ports are opened.

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### 3. Fail2ban — Brute Force Protection

Monitors SSH logs and bans IPs after repeated failed login attempts.

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban --now
```

### 4. Disable Root SSH Login

Prevents direct root access via SSH, enforcing account separation.

```bash
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart ssh
```

### 5. Static IP via Netplan

DNS and VPN services require a stable address. Configured via `/etc/netplan/50-cloud-init.yaml`:

```yaml
network:
  version: 2
  wifis:
    wlan0:
      dhcp4: no
      addresses:
        - 192.168.128.10/23
      routes:
        - to: default
          via: 192.168.128.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
      access-points:
        "SSID":
          password: "REDACTED"
```

## Security Controls

| Control | Framework Reference |
|---|---|
| UFW default deny | CIS Control 4.4 · NIST SC-7 |
| Fail2ban | CIS Control 4.3 · NIST AC-7 |
| PermitRootLogin no | CIS Control 5.3 · NIST AC-6 |
