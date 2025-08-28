# Day 2: Detecting RDP Brute-Force Attacks in Splunk

This document outlines the steps to detect Remote Desktop Protocol (RDP) brute-force attacks using Splunk, Sysmon, and Windows Security event logs.

## 1. Install Sysmon on Target Windows Machine

### Download and Configure Sysmon
- Download Sysmon from: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Get the configuration file from: https://github.com/SwiftOnSecurity/sysmon-config

<img width="661" height="130" alt="image" src="https://github.com/user-attachments/assets/2f7f36f4-9316-486e-8993-4ba44db04d44" />
<img width="630" height="161" alt="image" src="https://github.com/user-attachments/assets/8c76f62c-f52b-4d40-97d8-52edf818abfd" />

### Install with Configuration
```powershell
sysmon.exe -accepteula -i sysmonconfig-export.xml
```
<img width="868" height="181" alt="image" src="https://github.com/user-attachments/assets/f1427a40-f4b0-42db-a598-18a326990ed6" />
<img width="887" height="517" alt="image" src="https://github.com/user-attachments/assets/d9114a1e-fdfc-4fab-901c-161c85d2e1cb" />

To confirm that Sysmon is running, go to Event Viewer / Application and Services Logs / Microsoft / Windows / Sysmon should be listed

<img width="332" height="363" alt="image" src="https://github.com/user-attachments/assets/92d91d7b-e62d-4344-8f55-c821d5b2c937" />
<img width="312" height="97" alt="image" src="https://github.com/user-attachments/assets/d5113ac1-99d2-4492-bc02-92b058165856" />

## 2. Configure Splunk Universal Forwarder on Windows

### Navigate to the inputs.conf File
```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```
<img width="647" height="172" alt="image" src="https://github.com/user-attachments/assets/bad6caab-6aa2-4c89-84ef-f5d154cecd7d" />

### Add Sysmon Event Collection
```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog:Sysmon
renderXml = false
```
<img width="466" height="422" alt="image" src="https://github.com/user-attachments/assets/b81715c2-0c6f-4b72-9f08-cfa7e8af58a1" />

### (Optional) Add Other Event Logs
```ini
[WinEventLog://Application]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Application

[WinEventLog://Security]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Security

[WinEventLog://System]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:System
```

### Restart the Forwarder
```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
./splunk.exe restart
```
<img width="820" height="22" alt="image" src="https://github.com/user-attachments/assets/37cf73df-b09a-401e-8be0-630ec8482ad7" />
<img width="657" height="26" alt="image" src="https://github.com/user-attachments/assets/89c87687-63b6-4e23-a589-2ed176807f2b" />

## 3. Configure Splunk Server

### Create Indexes
- Go to: `Settings > Indexes`
- Click **New Index**, name it `sysmon_logs`, then save.

<img width="592" height="397" alt="image" src="https://github.com/user-attachments/assets/b50ba6bb-7989-4aa7-8fa2-653f420c1e12" />
<img width="787" height="622" alt="image" src="https://github.com/user-attachments/assets/d1ae9a88-2847-49a8-b568-3f58d48e0a46" />

### Create Source Types
- Go to: `Settings > Source types`
- Add `XmlWinEventLog:Sysmon` as a new source type.

<img width="642" height="322" alt="image" src="https://github.com/user-attachments/assets/da127c97-61cb-4bf4-bb89-dfa81f520a21" />
<img width="783" height="501" alt="image" src="https://github.com/user-attachments/assets/a26af4c8-b299-4b50-b7ce-e61aed2a0af9" />

### Restart Splunk
- Navigate to `Settings > Server Controls`
- Click **Restart Splunk**

<img width="372" height="622" alt="image" src="https://github.com/user-attachments/assets/64338031-936b-4ce9-8187-bf5d5fbad360" />

## 4. Simulate a Brute-Force Attack (Optional)

### Use Hydra to Simulate
```bash
hydra -l Administrator -P passwords.txt rdp://<victim-ip>
```
<img width="520" height="60" alt="image" src="https://github.com/user-attachments/assets/30390d83-9576-4f4c-93b7-af76a84aab2a" />
<img width="1406" height="598" alt="image" src="https://github.com/user-attachments/assets/a1db4b79-699a-4e3c-b6a3-ddc166be6f92" />

### Key Event Codes
- `4625` — Failed Logon
- `4624` — Successful Logon
- `4776` — NTLM Authentication Attempt

## 5. Detect RDP Attack in Splunk

### Sample SPL Query
```spl
index=wineventlog (EventCode=4625 OR EventCode=4776) Logon_Type=10
| stats count by Account_Name, src_ip
| where count > 5
```
Reviewing logs from the index
<img width="1895" height="128" alt="image" src="https://github.com/user-attachments/assets/d79181ff-1202-41dd-9699-424ecfef71be" />
<p>Capturing the IP address of the attacker
<p><img width="398" height="177" alt="image" src="https://github.com/user-attachments/assets/261bf635-d9ce-47db-9747-3993866a91c2" />
<p></p>Filtering for the source IP address of the attacker
<p><img width="936" height="329" alt="image" src="https://github.com/user-attachments/assets/ec278e14-fc09-42f8-8534-461af84220df" />
<p>Reviewing the event that show the source IP and destination IP
<p><img width="1152" height="745" alt="image" src="https://github.com/user-attachments/assets/1f769018-3394-4ccf-a4fd-597c5dd07951" />
<p></p>Displaying the IP address of the victim's machine
<p><img width="1460" height="627" alt="image" src="https://github.com/user-attachments/assets/2ed6aebe-a58e-4ecd-9683-9fa4486065b6" />

### Interpretation
- Filters failed or NTLM logon attempts via RDP (Logon_Type=10)
- Groups by account name and source IP
- Flags accounts with > 5 failed attempts as suspicious

## 6. Incident Response
- Block offending IPs
- Reset compromised credentials
- Enforce Multi-Factor Authentication (MFA)
- Enable account lockout policies

### Steps followed to block attacker's IP in Windows Firewall
1. Once the IP has been identified, we can go to the victim server and create a firewall rule
2. Open Windows Defender Firewall with Advanced Security
   <img width="1047" height="787" alt="image" src="https://github.com/user-attachments/assets/19956624-8cb5-4d35-b310-709d86dce33a" />

4. Inbound Rules / New Rule
   <img width="1272" height="792" alt="image" src="https://github.com/user-attachments/assets/b533486d-0e31-418a-b529-df3355ffb9f5" />
   <img width="710" height="577" alt="image" src="https://github.com/user-attachments/assets/bc2110fe-9ead-46c2-97fd-b38c70aa2fd9" />
   <img width="718" height="578" alt="image" src="https://github.com/user-attachments/assets/b2497672-63df-4d6f-ad1e-7cb83fad6a2e" />
   <img width="701" height="567" alt="image" src="https://github.com/user-attachments/assets/2074974e-a700-4a6a-aa39-202fa02f256c" />
   <img width="708" height="572" alt="image" src="https://github.com/user-attachments/assets/2d2e6ea1-57c3-4685-b54c-31b1e44867b3" />
   <img width="332" height="373" alt="image" src="https://github.com/user-attachments/assets/0304377a-41a7-4b3e-b4c5-d0d6e1ec1642" />
   <img width="713" height="577" alt="image" src="https://github.com/user-attachments/assets/b378c4dd-18d4-45cb-b33a-3192b5ff3904" />
   <img width="707" height="572" alt="image" src="https://github.com/user-attachments/assets/68c81cba-9e98-4ab8-915f-73a77a94d7a4" />
   <img width="705" height="575" alt="image" src="https://github.com/user-attachments/assets/328823a7-b956-4469-ae3d-5c524bc6f33c" />
 

Blocking with PowerShell Script
<img width="857" height="121" alt="image" src="https://github.com/user-attachments/assets/8f0d18a0-6481-4680-bd6e-a494f993fc98" />



---

**Next Steps**: Extend detection by correlating with successful logins (EventCode=4624) or exploring related Sysmon process trees to uncover post-authentication activity.
