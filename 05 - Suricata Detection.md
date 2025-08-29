# Day 5: Detecting Network Threats with Suricata and Splunk

## Overview
This guide walks through installing and configuring Suricata as a network intrusion detection system (NIDS) with Emerging Threats rules and integrating it with Splunk to analyze network-based threats. The final outcome is to simulate attacks, capture alerts, and build dashboards in Splunk for proactive detection.

---

## Tools Used
- **Suricata**: Open-source NIDS
- **Emerging Threats (ET) Rules**: Community-driven rule set
- **Splunk Universal Forwarder**: For log forwarding
- **Splunk**: For alerting and dashboarding

---

## Step 1: Install Suricata on Ubuntu
```bash
sudo apt update
sudo apt-get install suricata -y
```

---

## Step 2: Install Emerging Threats Ruleset
```bash
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

Emerging Threats (ET) is a comprehensive rule set for detecting malware, C2 traffic, and more.

---

## Step 3: Configure Suricata
Edit the configuration:
```bash
sudo nano /etc/suricata/suricata.yaml
```

Update:
```yaml
HOME_NET: "137.184.239.33"         # Your local IP
EXTERNAL_NET: "any"
default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"
interface: eth0                     # Or the appropriate NIC
```

Enable logging and stats as needed.

<img width="860" height="617" alt="image" src="https://github.com/user-attachments/assets/4bd435a6-f4fe-429b-a475-49bc75d4eb35" />
<img width="702" height="205" alt="image" src="https://github.com/user-attachments/assets/ff3b9457-6b6b-47e9-865b-c00b545b16dc" />
<img width="426" height="100" alt="image" src="https://github.com/user-attachments/assets/2ed3d799-b92e-4aed-8dec-164fdd288ce3" />

---

## Step 4: Start and Test Suricata
```bash
sudo systemctl restart suricata
sudo systemctl enable suricata
sudo systemctl status suricata
```

Check logs:
```bash
cat /var/log/suricata/fast.log
cat /var/log/suricata/eve.json
```

---

## Step 5: Configure Splunk Universal Forwarder
```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

Add log monitor:
```ini
[monitor:///var/log/suricata/fast.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```
<img width="490" height="382" alt="image" src="https://github.com/user-attachments/assets/c85188af-0a61-44fe-852f-564d788cfc3c" />

Add the index and sourcetype to Splunk:
- Settings / Indexes / New Index and Source types
<img width="663" height="659" alt="image" src="https://github.com/user-attachments/assets/9bbe67d0-8c58-45b0-877e-14a4b42da017" />
<img width="777" height="804" alt="image" src="https://github.com/user-attachments/assets/3ffb028b-36db-467c-b24a-e1c392a51625" />
<img width="622" height="634" alt="image" src="https://github.com/user-attachments/assets/9e944899-8b02-4b28-9f35-3571aed30097" />
<img width="786" height="494" alt="image" src="https://github.com/user-attachments/assets/e586455b-c0f7-475b-9086-4225b06e805e" />

Restart the forwarder:
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

Verify ingestion in Splunk with:
```spl
index=network_security_logs sourcetype=suricata_alerts
```

---

## Step 6: Simulate a Reconnaissance Attack

- **Attacker IP**: 146.85.*.* (Public IP) or 192.168.*.* (Private IP)
- **Target IP**: 137.184.239.33

On attacker machine (e.g., Kali Linux):
```bash
nmap -sS 137.184.239.33
```

Check for triggered Suricata alerts in Splunk:
```spl
index=network_security_logs alert="ET*" src_ip=192.168.*.*
```
<img width="878" height="496" alt="image" src="https://github.com/user-attachments/assets/af3ff348-68fa-4b71-a811-54928ef7e49f" />

---

## Step 7: Analyze and Visualize in Splunk
**Dashboards to create:**
- Top triggered rules
- Top source/destination IPs
- Timechart of alerts

Sample search for destination traffic:
```spl
index=network_security_logs sourcetype=suricata dest_ip="137.184.239.33"
```

---

## Step 8: Incident Response Actions

### Block Attacker IP:
```bash
sudo ufw deny from 192.168.*.*
```

### Kill Active Connections:
```bash
sudo netstat -tulpn | grep 192.168.*.*
sudo kill <PID>
```

Document findings:
- Triggered rule names
- Affected systems
- Response actions taken

---

## Summary
By combining Suricata and Splunk, you create a real-time detection system capable of:
- Monitoring traffic with ET rules
- Alerting on malicious behavior
- Investigating threats using powerful searches
- Visualizing traffic patterns and response
