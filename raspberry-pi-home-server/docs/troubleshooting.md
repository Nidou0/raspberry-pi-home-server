# Troubleshooting log

## 1. SSH to the Pi failed
**Symptom:** the laptop couldn't connect to the Pi, and a later attempt showed a host key mismatch warning.
**Cause:** the house has two routers. The laptop was on one subnet (`192.168.31.x`) and the Pi on the other (`192.168.1.x`), so the laptop couldn't route to the Pi. The host key warning meant the laptop had a stored key for that address that no longer matched the Pi.
**Fix:** connected the laptop to the same network as the Pi and removed the stale key with `ssh-keygen -R <host>` after confirming it was my Pi. Tailscale later removed the dependency on the local network entirely.

## 2. Internet dropped after pointing the laptop's DNS at Pi-hole
**Symptom:** with the laptop's DNS set to the Pi's IP, nothing resolved, and the Pi couldn't even be pinged.
**Diagnosis:**
- Switched DNS back to automatic first to restore connectivity.
- `ipconfig` showed the laptop was on the other router's subnet again.
- On the Pi, `dig google.com @127.0.0.1` showed Pi-hole itself was answering.
- `nslookup google.com <pi-ip>` from the laptop tests Pi-hole without changing any settings.
**Lesson:** separate "is the service working" from "can the client reach it" before changing anything.

## 3. Pi-hole ignored queries over Tailscale
**Cause:** by default Pi-hole only answers queries from the local network, and Tailscale clients arrive from `100.x` addresses.
**Fix:** set **Permit all origins**. It's safe here because ufw doesn't expose port 53 to the internet.

## 4. Under-voltage with an external NVMe SSD
**Symptom:** `vcgencmd get_throttled` returned `0x50005` (under-voltage now and since boot) after an SSD in a USB enclosure was attached.
**Tests:** swapped to a verified 5V/3A supply and a short cable. `dmesg | grep -i voltage` still showed repeated "Undervoltage detected!" / "Voltage normalised" pairs under load, even with the SSD unplugged.
**Conclusion:** the fault is in the board's own power path, most likely around the micro USB input, and not in the supply. This Pi stays on light duties (Pi-hole, Tailscale), and storage moves to a Raspberry Pi 4.

## 5. Recovering ~22 GB of backups from the SD card
**Problem:** DiskInternals Linux Reader listed the ext4 partition but couldn't open files, probably because of newer ext4 features. WSL's `--mount` also failed because the laptop's SD reader doesn't present cards as fixed disks.
**Fix:** booted the Pi normally and copied everything over the Samba share with `robocopy`, which retries and resumes cleanly on large transfers.
