# Day 4: File Integrity Monitoring with Auditd and Splunk

## Objective
The objective of this guide is to detect unauthorized file or directory changes in Linux systems using Auditd and to analyze those events within Splunk. This provides hands-on experience with setting up file integrity monitoring, simulating suspicious behavior, and responding to system-level threats.

---

## Tools Used
- **Auditd**: For system-level auditing of file changes
- **Splunk Universal Forwarder**: To send logs from the Linux server to Splunk
- **Splunk**: For log ingestion, search, alerting, and dashboards

---

## Step 1: Prerequisites
Ensure the following before proceeding:
- Ubuntu Server with `sudo` access
- Auditd installed
- Splunk Universal Forwarder installed and configured
- Splunk instance accessible via web UI

---

## Step 2: Install and Configure Auditd

### Install Auditd
```bash
sudo apt update
sudo apt install auditd -y
```
<img width="497" height="25" alt="image" src="https://github.com/user-attachments/assets/6cc55997-96f8-47ee-a3dc-8443a500be5b" />

### Start and Enable Auditd
```bash
sudo systemctl start auditd
sudo systemctl enable auditd
```
<img width="985" height="104" alt="image" src="https://github.com/user-attachments/assets/956c348a-ee02-44ec-8cd2-0b95260158e2" />

### Verify Auditd Status
```bash
sudo systemctl status auditd
```
<img width="970" height="437" alt="image" src="https://github.com/user-attachments/assets/ddc96d10-c0d9-4e3f-badd-bda5bae57fbe" />

---

## Step 3: Create Auditd File Monitoring Rules

### Edit Rule Configuration
```bash
sudo nano /etc/audit/rules.d/audit.rules
```

### Add Monitoring Rule
```bash
-w /etc/ -p wa -k file_integrity
```
- `-w` — Path to monitor
- `-p` — Permissions to monitor: `w` for write, `a` for attribute changes
- `-k` — Keyword to tag log entries

<img width="509" height="280" alt="image" src="https://github.com/user-attachments/assets/99055881-c561-4f32-82ae-86f9a71dcd70" />

### Restart Auditd
```bash
sudo service auditd restart
```
<img width="692" height="37" alt="image" src="https://github.com/user-attachments/assets/33e8e3bd-23f9-4eb1-998b-a733e8afa612" />

### Confirm Rule is Active
```bash
sudo auditctl -l
```

---

## Step 4: Configure Splunk Universal Forwarder

### Edit `inputs.conf`
```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

### Add Monitor for Auditd Logs
```ini
[monitor:///var/log/audit/audit.log]
disabled = false
index = linux_file_integrity
sourcetype = auditd
```
<img width="372" height="276" alt="image" src="https://github.com/user-attachments/assets/b615b21c-7341-451b-85e8-d963755d3b5c" />

### Restart Splunk Forwarder
```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
<img width="1235" height="503" alt="image" src="https://github.com/user-attachments/assets/6b56f96b-16b5-4a60-a82c-cc633c711523" />

---

## Step 5: Install and Use Auditd Splunk App

1. Download the Auditd app from [Splunkbase](https://splunkbase.splunk.com/app/2642)
2. In Splunk UI: Go to `Apps > Manage Apps > Install App from File`
3. Upload the `.tgz` package and install

<img width="341" height="386" alt="image" src="https://github.com/user-attachments/assets/038338aa-e517-4483-ba0c-3f678fa48499" />
<img width="672" height="112" alt="image" src="https://github.com/user-attachments/assets/beaac286-0841-4183-9d33-e229e36174e6" />
<img width="685" height="328" alt="image" src="https://github.com/user-attachments/assets/b11b84ec-6e5a-4af3-917a-ecb33b68648e" />

---

## Step 6: Simulate Unauthorized Changes

### Verify Rule is Added to Auditctl
```bash
auditctl -l
```

<img width="532" height="83" alt="image" src="https://github.com/user-attachments/assets/3299ce1e-c81a-4492-a3f7-ab55302b9007" />

### Add a File
```bash
sudo nano /etc/firstfileintegritycheck.txt
```
<img width="830" height="42" alt="image" src="https://github.com/user-attachments/assets/8417e33d-2095-424b-8144-bca142b137de" />

Added a line to text file

<img width="657" height="93" alt="image" src="https://github.com/user-attachments/assets/64bf04c2-cd36-4738-aaaa-b837a9597991" />

### Search with `ausearch`
```bash
sudo ausearch -k file_integrity
```
<img width="692" height="37" alt="image" src="https://github.com/user-attachments/assets/ba4b4120-88e9-4fda-bb76-f64ab5421523" />
<img width="1910" height="493" alt="image" src="https://github.com/user-attachments/assets/dda9944c-d8b8-40ce-888c-e752bd46c974" />

I searched for the file name specifically since several logs were coming through
<img width="1907" height="373" alt="image" src="https://github.com/user-attachments/assets/8434886f-c576-4093-9511-1846a867a97f" />

---

## Step 7: Analyze Logs in Splunk

### Search by Index
```spl
index=linux_file_integrity source="/var/log/audit/audit.log" sourcetype=auditd nametype=CREATE
```
<img width="1907" height="620" alt="image" src="https://github.com/user-attachments/assets/86e59808-7e9d-4b73-a991-74905c5a04cf" />

### or Search by Keyword
```spl
index=linux_file_integrity sourcetype=auditd key="file_integrity"
```
<img width="902" height="296" alt="image" src="https://github.com/user-attachments/assets/1aba3212-4e49-4e1a-aa29-0b57768b8169" />

### Sample Log Analysis
```text
type=SYSCALL msg=audit(1667829100.123:123): arch=c000003e syscall=2 success=yes exit=0 a0=7ffc12345 a1=80400 ...
```

---

## Step 8: Visualizations and Dashboard Ideas

- **Top Modified Files** by frequency
- **Users with Most Changes** using `auid`
- **Time Series** of write/attribute events

---

## Step 9: Incident Response Workflow

### Identify the User
Use `ausearch` or extracted Splunk fields.

### Restore Files
Recover from backup if `/etc/` files are corrupted.

### Add Access Controls
Restrict write access to `/etc/` to root-only.

---

## Summary
This setup offers a lightweight and effective way to:
- Detect file tampering in critical directories
- Investigate changes with full log context
- Correlate logs across Linux endpoints

Ensure rules are reviewed and tuned to match the organization's security requirements. This approach can be extended to monitor other directories such as `/var/log`, `/home`, or custom application paths.
