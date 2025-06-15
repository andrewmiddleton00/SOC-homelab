# SOC-homelab
My attempts at making a SOC home lab 
# SOC Lab Connectivity & Log Analysis Report (with Screenshots)

> **Project:** Home SOC Lab Network Troubleshooting & Apache Log Transfer
> **Date:** June 14–15, 2025
> **Author:** Andrew
> **Purpose:** Establish connectivity between Kali and Ubuntu VMs and analyze Apache logs as part of a personal SOC lab project.

---

## 1. Objective

Set up communication between a Kali Linux VM and an Ubuntu (SIEM) VM using Oracle VirtualBox, transfer Apache log files from Ubuntu to Kali, and perform basic analysis.

---

## 2. Initial Setup

* **VirtualBox Network Adapter:** Intel PRO/1000 MT Desktop
* **Ubuntu IP:** `10.10.10.10`
* **Kali IP:** `10.10.10.20`

Screenshots:

* 🖼️ Network configs (`ip a`): [See Screenshot 1](#)
* 🖼️ Ping test failures: [See Screenshot 2](#)

---

## 3. Apache Not Listening on Port 80

* Apache was running on Ubuntu, but Kali couldn't curl or scan the service.
* Verified Apache was enabled and `ss -tulnp | grep :80` showed `LISTEN`.

Troubleshooting:

* Changed `000-default.conf` and `ports.conf`
* Configtest failed due to `Listen :80` syntax error
* Corrected to `Listen 80`

🖼️ Config fix: [See Screenshot 3](#)

---

## 4. Python HTTP Server Alternative (Plan B)

Due to persistent connection issues, switched from Apache to a Python HTTP server:

```bash
cd /var/log/apache2/
sudo python3 -m http.server 8080 --bind 10.10.10.10
```

🖼️ Python server on Ubuntu: [See Screenshot 4](#)

---

## 5. Still Failing from Kali

* `curl http://10.10.10.10:8080` failed
* Attempted SCP transfer with SSH: `scp user@10.10.10.10:/var/log/apache2/access.log .`
* Also failed: Network unreachable or host down

🖼️ SCP & curl failures: [See Screenshot 5](#)

---

## 6. Final Solution: Shared Folder Transfer

Used VirtualBox shared folder to copy logs manually:

### Setup:

1. Created shared folder in VirtualBox: `/mnt/shared`
2. Mounted with:

   ```bash
   sudo mount -t vboxsf SharedLogs /mnt/shared
   sudo cp /var/log/apache2/access.log /mnt/shared
   ```
3. Accessed from Kali via `/media/sf_SharedLogs/`

🖼️ Shared folder config: [See Screenshot 6](#)
🖼️ Successful mount and copy: [See Screenshot 7](#)

---

## 7. Log Analysis on Kali

Basic command-line analysis:

### Count of Requests by IP:

```bash
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
```

### Count of Status Codes:

```bash
cat access.log | awk '{print $9}' | sort | uniq -c | sort -nr
```

### Tool Check (Curl):

```bash
grep "curl" access.log
```

### Error Check (404):

```bash
grep "404" access.log
```

🖼️ Command output: [See Screenshot 8](#)

---

## 8. Conclusion

While direct HTTP access from Kali failed due to unknown network routing issues, the fallback via shared folders allowed full log transfer and analysis. This exercise:

✅ Reinforced VM network troubleshooting
✅ Showcased fallback strategies (Python server, SCP, shared folder)
✅ Demonstrated basic log analysis skills

---

## 9. Improvements for Future

* Pre-configure NAT or Host-Only adapter properly
* Use internal DHCP or static routes for consistency
* Include Suricata or Wazuh logs next

---

**✅ Ready for GitHub commit. Next step:** Upload images and update references!
