# 03 — WireGuard

**Host:** Raspberry Pi 4B — 192.168.128.10  
**VPN Subnet:** 10.8.0.0/24  
**Listen Port:** 51820/UDP

## Rationale

WireGuard provides encrypted tunnel access to the home network. Key advantages over OpenVPN:
- ~4,000 lines of code vs ~400,000 (smaller attack surface, easier to audit)
- Significantly better performance on ARM architecture
- Modern cryptography: ChaCha20, Poly1305, Curve25519, BLAKE2

When the tunnel is active, all DNS queries from the client are routed through Pi-hole, ensuring filtering and logging apply even when connected remotely.

## Installation

```bash
sudo apt install wireguard -y
```

## Server Configuration

### Generate Keys

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

### `/etc/wireguard/wg0.conf`

```ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.8.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o wlan0 -j MASQUERADE

[Peer]
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32
```

### Enable IP Forwarding

```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p
```

### Start Service

```bash
sudo systemctl enable wg-quick@wg0 --now
```

## Client Configuration

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.8.0.2/32
DNS = 192.168.128.10

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = 192.168.128.10:51820
AllowedIPs = 192.168.128.0/23, 10.8.0.0/24
PersistentKeepalive = 25
```

`DNS = 192.168.128.10` ensures all DNS queries from the client route through Pi-hole when the tunnel is active.

## Verification

```bash
# Server side
sudo wg show

# Client side
ping 10.8.0.1
```

Expected output includes an active peer with recent handshake and transfer stats.

## Security Controls

| Control | Framework Reference |
|---|---|
| Encrypted tunnel | CIS Control 12.6 · NIST SC-8 |
| DNS through VPN | CIS Control 9.2 · NIST SC-20 |
| Minimal codebase | NIST SA-11 (Attack Surface Reduction) |
