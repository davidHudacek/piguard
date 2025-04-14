# Installation Instructions for PiGuard

This guide explains how to set up PiGuard on a Raspberry Pi 4B. You will install Pi-hole, configure Tor for network-wide anonymization, and set up custom scripts for monitoring and control.

---

## 1. Prepare the Raspberry Pi

- Use Raspberry Pi Imager or Balena Etcher to flash **Raspberry Pi OS Lite** onto a microSD card (16GB or more).
- Boot your Raspberry Pi.
- Set up basic configuration:
  ```bash
  sudo raspi-config
  ```
  - Change password
  - Set hostname (e.g., `piguard`)
  - Enable SSH
  - Set timezone and locale

---

## 2. Set Static IP Address
Edit the network config:
```bash
sudo nano /etc/dhcpcd.conf
```
Add at the end:
```
interface eth0
static ip_address=192.168.178.250/24
static routers=192.168.178.1
static domain_name_servers=192.168.178.1
```
> Adjust IPs to match your local network.

Reboot:
```bash
sudo reboot
```

---

## 3. Install Pi-hole
Run the official installer:
```bash
curl -sSL https://install.pi-hole.net | bash
```
During the setup:
- Choose static IP `192.168.178.250`
- Enable web interface
- Choose upstream DNS (Cloudflare, Google...)
- Note the **admin password**

After install, Pi-hole UI is available at:
```
http://192.168.178.250/admin
```

---

## 4. Set Pi-hole DNS in Your Router
Access your router’s admin panel and set the DNS server for your LAN to:
```
192.168.178.250
```
> This makes all devices in your home use Pi-hole automatically.

---

## 5. Install Tor
```bash
sudo apt update && sudo apt install tor -y
```

Edit the configuration:
```bash
sudo nano /etc/tor/torrc
```
Add these lines:
```
AutomapHostsOnResolve 1
TransPort 9040
DNSPort 5353
VirtualAddrNetworkIPv4 10.192.0.0/10
```
Restart Tor:
```bash
sudo systemctl restart tor
```
Disable it from auto-start:
```bash
sudo systemctl disable tor
```
> (We will manage Tor manually with a toggle script.)

---

## 6. Install Required Tools
```bash
sudo apt install iptables-persistent python3-pip -y
sudo pip3 install requests
```
Install and test `speedtest-cli`:
```bash
sudo apt install speedtest-cli
speedtest-cli --simple
```
If it doesn’t work via Tor, install a compatible version manually.

---

## 7. Deploy Scripts
Place your scripts in:
```
/usr/local/bin/
```
Make them executable:
```bash
sudo chmod +x /usr/local/bin/dispinfo.py
sudo chmod +x /usr/local/bin/toggle_tor.py
```
(Optional) Test them:
```bash
sudo /usr/local/bin/toggle_tor.py
sudo /usr/local/bin/dispinfo.py
```

---

## 8. Save Persistent iptables Rule for Local Traffic
Ensure Pi-hole always works from LAN, even when Tor is active:
```bash
sudo iptables -t nat -A OUTPUT -d 192.168.178.0/24 -j RETURN
sudo netfilter-persistent save
```

---

## 9. Setup Autostart (Optional)
You can run your monitoring script automatically:
```bash
crontab -e
```
Add:
```
@reboot /usr/local/bin/dispinfo.py
```
Or use systemd for better control.

---

✅ Done! You now have PiGuard running Pi-hole and Tor with full CLI monitoring and toggle functionality.

---

> For further security upgrades (VPN, SSH hardening), see planned upgrades in the main README.
