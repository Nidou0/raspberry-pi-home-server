# Raspberry Pi Home Server

A self-hosted home server on a Raspberry Pi 3 Model B. It provides network-wide DNS filtering with Pi-hole, a backup file share over Samba, and secure remote access from anywhere through Tailscale.

I built this to learn Linux server administration, networking and basic hardening hands on. Most of the useful lessons came from things that broke, so the [troubleshooting log](docs/troubleshooting.md) is the most interesting part of this repo.

## What it does

| Service | Purpose |
|---|---|
| **Pi-hole** | Network-wide DNS filtering that blocks ads and trackers for every device using it |
| **Tailscale** (MagicDNS) | VPN mesh for remote SSH, file access and DNS filtering on any network, including mobile data |
| **Samba** | SMB file share used as a backup target from Windows and iPhone |
| **ufw** | Host firewall that only allows SSH, DNS (53), the Pi-hole dashboard (80) and Tailscale |
| **fail2ban** | Bans IPs after repeated failed SSH logins |
| **unattended-upgrades** | Applies security updates automatically |

## Architecture

```
            Internet
                │
      ┌─────────┴─────────┐
      │  Upstream DNS     │  Cloudflare / Quad9
      └─────────┬─────────┘
                │
   ┌────────────┴─────────────┐
   │  Raspberry Pi 3 (Linux)  │
   │  Pi-hole · Samba · ufw   │
   │  fail2ban · Tailscale    │
   └────────────┬─────────────┘
                │ Tailscale VPN (MagicDNS name: media-drive)
     ┌──────────┼──────────────┐
  Laptop     iPhone      Any network / mobile data
  (DNS, SMB, SSH)  (DNS, SMB)
```

Clients send DNS queries to the Pi's Tailscale address. Tailscale's admin DNS settings point to Pi-hole with **Override local DNS** turned on, so filtering follows the device wherever it is.

## Build steps

1. **Base setup.** Flashed Raspberry Pi OS, enabled SSH and administered the Pi headless from a Windows laptop.
2. **Networking.** Found the Pi's address and gateway with `nmcli connection show`, `hostname -I` and `ip route`. Planned a fixed address, preferring a DHCP reservation on the router to a static IP set on the Pi to avoid address clashes.
3. **Remote access.** Installed Tailscale and enabled MagicDNS so the Pi is reachable by name (`ssh <user>@media-drive`) from any network.
4. **File sharing.** Configured a Samba share for backups, including Apple compatibility (`vfs objects = fruit streams_xattr`) for access from the iPhone Files app.
5. **Hardening.** Enabled `unattended-upgrades`, set up ufw rules and installed fail2ban for SSH.
6. **Pi-hole.** Installed Pi-hole with Cloudflare/Quad9 upstream resolvers, opened ports 53 and 80/tcp in ufw, and set **Permit all origins** so it answers queries arriving over the Tailscale interface. This is safe because ufw blocks port 53 from the internet.
7. **Testing.** Checked filtering with `nslookup doubleclick.net` (returns `0.0.0.0` when blocked) and watched queries arrive in the dashboard.

See [docs/setup-notes.md](docs/setup-notes.md) for the commands.

## Troubleshooting highlights

- **Couldn't reach the Pi over SSH:** a double-router home network put the laptop and the Pi on different subnets (`192.168.31.x` and `192.168.1.x`).
- **Lost internet after switching the laptop's DNS to Pi-hole:** traced it to a network path problem rather than Pi-hole itself, using `ping`, `nslookup <domain> <pi-ip>` and `dig @127.0.0.1` on the Pi.
- **Under-voltage while running an external SSD:** `vcgencmd get_throttled` (`0x50005`) and `dmesg` logs showed voltage dips even without the SSD attached, pointing to a fault in the board's power path rather than the charger or cable.
- **Recovering ~22 GB of backups from the SD card:** Windows ext4 readers failed, so I copied the data off over the network share with `robocopy`.

Full details are in [docs/troubleshooting.md](docs/troubleshooting.md).

## Next steps

- Move backup storage to a Raspberry Pi 4 with a USB SSD, since this Pi 3 can't deliver stable power to external drives.
- Add Pi-hole as router-wide DNS through a DHCP reservation, so devices without Tailscale are covered too.
- Schedule automated backups and monitoring.

## Skills practised

Linux administration · SSH · DNS · DHCP/static addressing · VPN · SMB file sharing · firewall rules · fail2ban · patch management · log-based fault diagnosis
