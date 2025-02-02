
# Raspberry Pi 3 Hotspot Setup

This tutorial will guide you through setting up your Raspberry Pi 3 (running Raspberry Pi OS Lite - Bullseye) as a Wi-Fi hotspot (Access Point).

## Prerequisites

- Raspberry Pi 3 with Raspberry Pi OS Lite (Bullseye)
- A working internet connection (Ethernet or Wi-Fi)
- Basic knowledge of using the terminal

## Steps

### 1. Update the system
First, update your system to make sure everything is up-to-date.

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install required software
Install `hostapd` (for the Access Point) and `dnsmasq` (for DHCP server).

```bash
sudo apt install hostapd dnsmasq
```

### 3. Stop the services
Stop both services temporarily so that we can configure them.

```bash
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

### 4. Disable the services from starting automatically

```bash
sudo systemctl disable hostapd
sudo systemctl disable dnsmasq
```

### 5. Configure `hostapd`
Create a configuration file for `hostapd` to set up the access point.

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Add the following configuration:

```plaintext
interface=wlan0
driver=nl80211
ssid=NamaHotspot
hw_mode=g
channel=7
wmm_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=PasswordHotspot
rsn_pairwise=CCMP
```

Replace `NamaHotspot` with your desired SSID (hotspot name) and `PasswordHotspot` with your desired Wi-Fi password.

### 6. Set `hostapd` to use the configuration
Configure the system to use the `hostapd.conf` file.

```bash
sudo nano /etc/default/hostapd
```

Add the following line:

```plaintext
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### 7. Configure `dnsmasq`
Set up the DHCP server by editing `dnsmasq.conf`.

```bash
sudo nano /etc/dnsmasq.conf
```

Add the following configuration:

```plaintext
interface=wlan0
dhcp-range=192.168.100.50,192.168.100.150,255.255.255.0,24h
```

### 8. Set static IP for wlan0
Configure a static IP address for the Wi-Fi interface.

```bash
sudo nano /etc/dhcpcd.conf
```

Add the following configuration at the bottom:

```plaintext
interface wlan0
static ip_address=192.168.100.1/24
nohook wpa_supplicant
```

### 9. Unmask `hostapd` service
Unmask the `hostapd` service so it can be started.

```bash
sudo systemctl unmask hostapd
```

### 10. Enable services
Enable the `hostapd` and `dnsmasq` services to start automatically on boot.

```bash
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
```

### 11. Start the services
Start both services.

```bash
sudo systemctl start hostapd
sudo systemctl start dnsmasq
```

### 12. Reboot the Raspberry Pi
Finally, reboot your Raspberry Pi to ensure everything starts properly.

```bash
sudo reboot
```

## Optional: Configure IP Forwarding and NAT for Internet Sharing

To share your internet connection over the Wi-Fi hotspot, follow these optional steps:

1. **Enable IP Forwarding**:
   Edit the sysctl configuration file to enable IP forwarding.

   ```bash
   sudo nano /etc/sysctl.conf
   ```

   Add or uncomment the following line:

   ```plaintext
   net.ipv4.ip_forward=1
   ```

   Apply the changes:

   ```bash
   sudo sysctl -p
   ```

2. **Configure NAT (Network Address Translation)**:
   Set up iptables to route traffic from your Wi-Fi network (wlan0) to your Ethernet connection (eth0).

   ```bash
   sudo iptables --table nat -A POSTROUTING -o eth0 -j MASQUERADE
   ```

3. **Save iptables rules**:
   Save the iptables configuration to ensure the changes persist after reboot.

   ```bash
   sudo sh -c 'iptables-save > /etc/iptables/rules.v4'
   ```

   If the `/etc/iptables` directory does not exist, create it first:

   ```bash
   sudo mkdir -p /etc/iptables
   ```

4. **Install iptables-persistent**:
   To ensure iptables rules are automatically applied at boot, install the `iptables-persistent` package.

   ```bash
   sudo apt install iptables-persistent
   ```

   During the installation, you'll be asked if you want to save the current iptables rules. Select `Yes`.

5. **Reboot the Raspberry Pi**:
   Reboot your Raspberry Pi to apply all the configurations.

   ```bash
   sudo reboot
   ```

## Testing

Your Raspberry Pi should now be acting as a Wi-Fi hotspot. Try connecting another device (smartphone, laptop, etc.) to the hotspot using the SSID and password you configured.

---

## Troubleshooting

If the services do not start correctly, check the logs for more information:

```bash
sudo journalctl -u hostapd
sudo journalctl -u dnsmasq
```

---

## License

This tutorial is provided under the MIT License. Feel free to use and modify it as needed.
