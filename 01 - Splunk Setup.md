# Splunk Server and Universal Forwarder Setup Guide

This guide documents the setup of a Splunk Enterprise server on Ubuntu Linux, and the installation and configuration of Universal Forwarders on both Windows and Ubuntu systems.

---

## Splunk Enterprise Setup on Ubuntu

**VM Specs:**
- 2 CPU cores
- 4 GB RAM
- 30 GB disk

**OS:** Ubuntu 22.04 LTS

### Installation Steps
1. **Update System**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Download Splunk Enterprise**
   ```bash
   wget -O splunk-9.3.0-51ccf43db5bd-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.3.0/linux/splunk-9.3.0-51ccf43db5bd-linux-2.6-amd64.deb"
   ```

3. **Install Package**
   ```bash
   sudo dpkg -i splunk-9.3.0-51ccf43db5bd-linux-2.6-amd64.deb
   ```

4. **Start Splunk & Create Admin Account**
   ```bash
   sudo /opt/splunk/bin/splunk start --accept-license
   ```
   Create an `admin` user and set the password when prompted.

5. **Enable Auto-Start**
   ```bash
   sudo /opt/splunk/bin/splunk enable boot-start
   ```

6. **Make Splunk Web Accessible**
   ```bash
   sudo nano /opt/splunk/etc/splunk-launch.conf
   ```
   Add the following:
   ```
   SPLUNK_BINDIP=0.0.0.0
   ```

7. **Open Required Port**
   ```bash
   sudo ufw allow 8000/tcp
   ```

### Access Splunk Web UI
- Navigate to: `http://<VM-IP>:8000`
- Login with your admin credentials

---

## Splunk Universal Forwarder Setup on Windows

**VM Specs:**
- 2 CPU cores
- 4 GB RAM
- 40 GB disk

**OS:** Windows 10 Pro or Windows Server 2019

### Installation Steps
1. **Download Installer** from [Splunk Downloads](https://www.splunk.com/en_us/download/universal-forwarder.html)

2. **Run Installer** and select:
   - `Forwarder Only`
   - Set admin credentials
   - Provide Splunk Server IP and Port `9997`

3. **Verify Service**
   - Open `services.msc`
   - Confirm `SplunkForwarder` status is **Running**

---

## Splunk Universal Forwarder Setup on Ubuntu

**VM Specs:**
- 2 CPU cores
- 4 GB RAM
- 20 GB disk

**OS:** Ubuntu 24.04 LTS

### Installation Steps
1. **Update System**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Download Universal Forwarder**
   ```bash
   wget -O splunkforwarder-10.0.0-e8eb0c4654f8-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.0.0/linux/splunkforwarder-10.0.0-e8eb0c4654f8-linux-amd64.deb"
   ```

3. **Install Package**
   ```bash
   sudo dpkg -i splunkforwarder-10.0.0-e8eb0c4654f8-linux-amd64.deb
   ```

4. **Start Forwarder**
   ```bash
   sudo /opt/splunkforwarder/bin/splunk start --accept-license
   ```
   Create `admin` credentials when prompted.

5. **Configure Splunk Server Forwarding**
   ```bash
   sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_SERVER_IP>:9997 -auth admin:<your-password>
   ```

6. **Add Log Monitor Path**
   ```bash
   sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
   ```

---

## Configure Splunk Server to Receive Logs
1. **Login to Splunk Web UI**
2. Navigate to:
   - `Settings` ➝ `Forwarding and receiving`
   - Click `+Add New` under Receive Data
   - Add port `9997` and click Save

3. **Allow TCP Port 9997 in Firewall**
   ```bash
   sudo ufw allow 9997/tcp
   ```

---

Once the forwarders are connected and monitored paths are added, log data will begin streaming into your Splunk Enterprise instance for indexing and analysis.

---

Images from installing Splunk Universal Forwarder on Windows. Unfortunately, I didn't get screenshots while installing on Linux.

<img width="1213" height="307" alt="image" src="https://github.com/user-attachments/assets/ea9206e3-3c19-4bf9-bd19-d77086468f16" />
<img width="851" height="457" alt="image" src="https://github.com/user-attachments/assets/611e5362-ee17-41d4-a416-88354632bfb2" />
<img width="518" height="406" alt="image" src="https://github.com/user-attachments/assets/d63dcc85-978e-4c0e-ade0-168e22ef41dc" />
<img width="507" height="401" alt="image" src="https://github.com/user-attachments/assets/3f337c53-99d3-4230-ae19-d48579b809c5" />
<img width="502" height="392" alt="image" src="https://github.com/user-attachments/assets/d36a2a5b-596b-4087-a3e8-b39cb58c82ed" />
<img width="506" height="396" alt="image" src="https://github.com/user-attachments/assets/49be0e84-ece5-4d41-9b67-94ef72b964a1" />
<img width="502" height="392" alt="image" src="https://github.com/user-attachments/assets/b3c69514-08fe-4a42-82c8-fe0a866ec2be" />
<img width="512" height="401" alt="image" src="https://github.com/user-attachments/assets/4528ecc0-8c43-4d0d-ab49-b244b7d6ce9d" />



