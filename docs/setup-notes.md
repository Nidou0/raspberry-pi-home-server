# Setup notes

Placeholders like `<pi-ip>` and `<connection-name>` stand for values specific to my network.

## Network discovery
```bash
nmcli connection show        # active connection name
hostname -I                  # current local IP
ip route | grep default      # gateway
```

## Fixed address (option used when the router can't be accessed)
```bash
sudo nmcli connection modify "<connection-name>" ipv4.method manual \
  ipv4.addresses <pi-ip>/24 ipv4.gateway <gateway> ipv4.dns "1.1.1.1"
sudo nmcli connection up "<connection-name>"
```
A DHCP reservation on the router is preferred, because the router then won't hand the same address to another device.

## Hardening
```bash
sudo apt install unattended-upgrades ufw fail2ban -y
sudo ufw allow OpenSSH
sudo ufw allow in on tailscale0
sudo ufw enable
sudo ufw status
```

## Pi-hole
```bash
curl -sSL https://install.pi-hole.net | bash
sudo pihole setpassword
sudo ufw allow 53
sudo ufw allow 80/tcp
pihole status
dig google.com @127.0.0.1
```
Dashboard: `http://media-drive/admin` (over Tailscale).

Settings → DNS → Interface settings → **Permit all origins**, so queries from Tailscale (`100.x`) addresses are answered.

## Testing from a client (Windows)
```
nslookup google.com <pi-ip>      # ask Pi-hole directly, without changing settings
ipconfig /flushdns
nslookup doubleclick.net         # 0.0.0.0 means it is blocked
```

## Power diagnostics
```bash
vcgencmd get_throttled           # 0x0 = healthy; 0x50005 = under-voltage now and earlier
dmesg | grep -i voltage
```
