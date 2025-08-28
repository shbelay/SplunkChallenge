# Day 3: Investigating PowerShell Abuse on Windows with Splunk and Sysmon

This documentation outlines how to detect, investigate, and respond to suspicious PowerShell activity on Windows systems using Sysmon logs ingested into Splunk.

## Objective

Detect and investigate malicious or unauthorized PowerShell activity using:
- Sysmon for process monitoring
- Splunk for log aggregation and alerting
- Simulated suspicious activity using the EICAR test file

## Tools Used
- **Windows Machine** for running PowerShell commands and generating Sysmon logs
- **Sysmon** to monitor PowerShell activity, including process creation and script execution
- **Splunk** for ingesting and analyzing Sysmon logs
- **Splunk Universal Forwarder** to send logs from Windows machine
- **Simulated Attack Tools**: Malicious PowerShell commands

---

## Step 1: Install and Configure Sysmon

### Download Sysmon
From Microsoft's Sysinternals page: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

### Install Sysmon
```powershell
sysmon.exe -accepteula -i sysmonconfig.xml
```

### Verify Installation
```powershell
sc query sysmon64
```

### Add PowerShell Events to Sysmon
- Navigate to Event Viewer
- Click Applications and Services Logs / Microsoft / Windows / PowerShell

  <img width="325" height="287" alt="image" src="https://github.com/user-attachments/assets/1ff7c8b1-1f89-40f1-9423-e319b1781229" />

  <img width="276" height="98" alt="image" src="https://github.com/user-attachments/assets/1e081919-bc5c-4ffa-ace0-0f0816e71fea" />

- Double-click Operational

  <img width="477" height="115" alt="image" src="https://github.com/user-attachments/assets/045794b5-316b-4e84-b864-60348346e554" />

- Copy the log path: Microsoft-Windows-PowerShell/Operational

  <img width="365" height="35" alt="image" src="https://github.com/user-attachments/assets/adf0241c-113b-48d9-957e-9597be6b1d77" />


---

## Step 2: Configure Splunk Universal Forwarder

### Inputs Configuration
Edit the following file:
```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

Add PowerShell logs to be captured by Sysmon and forwarded to Splunk:
```ini
[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = powershell_logs
sourcetype = WinEventLog:PowerShell
renderXml = false
```
<img width="521" height="540" alt="image" src="https://github.com/user-attachments/assets/f1afb8a9-a750-4eae-b9d5-c01202ecb80d" />

### Restart Universal Forwarder
```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
./splunk.exe restart
```
<img width="850" height="438" alt="image" src="https://github.com/user-attachments/assets/38e76b61-d0f9-4625-a697-fec706700087" />

---

## Step 3: Configure Outputs and Verify Connection

### Verify Forwarding
```powershell
splunk list forward-server
```

Ensure the indexer is listed as `active`.

---

## Step 4: Create Indexes and Sourcetypes in Splunk

- Go to **Settings > Indexes**

<img width="592" height="328" alt="image" src="https://github.com/user-attachments/assets/500531fb-192a-4c08-8d22-c657233e9931" />

- Create `powershell_logs`. sysmon_logs and windows_logs were created on previous days. 

<img width="793" height="252" alt="image" src="https://github.com/user-attachments/assets/302dec00-99b7-4bf2-b624-dcca30266f34" />

- Go to **Settings > Source types**
- Create source types: `WinEventLog:PowerShell`
  - WinEventLog:Sysmon and WinEventLog:Security were created on previous days.

<img width="580" height="346" alt="image" src="https://github.com/user-attachments/assets/0f6542c0-c236-4d9f-b427-c130c2cf05fb" />

Click New Source Type. Add 'WinEventLog:PowerShell' and save

<img width="780" height="496" alt="image" src="https://github.com/user-attachments/assets/4c7b285e-a12e-4678-b1f8-ef58f243bc46" />

Restart Splunk from **Settings > Server Controls > Restart**

<img width="640" height="647" alt="image" src="https://github.com/user-attachments/assets/a6040b48-24a7-4217-97b0-74c4182e1fb4" />

Verify PowerShell logs are being ingested

<img width="1510" height="553" alt="image" src="https://github.com/user-attachments/assets/baf10e17-243a-42ec-96a5-97bbcd18c5d4" />

---

## Step 5: Simulate Suspicious PowerShell Activity

Download the [EICAR test malware file](https://www.eicar.org/download-anti-malware-testfile/):
```powershell
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

This simulates malware download behavior.

<img width="857" height="107" alt="image" src="https://github.com/user-attachments/assets/31345fbb-677c-4f53-aeb8-966bb6840b18" />

---

## Step 6: Analyze Sysmon Logs in Splunk

### Verify Logs in Splunk Web UI
Example SPL:
```spl
index=sysmon_logs sourcetype="XmlWinEventLog:Sysmon"
```

### Sample Event Matches
- **EventCode=3/4** – Network connection
- **EventCode=11** – File created

**Details:**
- Source IP: `139.84.139.20`
- Destination Host: `ngcobalt397.manitu.net`
- File Created: `C:\Users\shbelay\Downloads\eicar.com.txt`

<img width="1158" height="516" alt="image" src="https://github.com/user-attachments/assets/81b94cf6-b2e2-4796-8cbb-4971bee38228" />

---

## Step 7: Visualization in Splunk

- **Bar chart** of PowerShell command frequency
- **Pie chart** of users running PowerShell
- **Line graph** showing activity over time

---

## Step 8: Incident Response Actions

### Kill Malicious PowerShell Process
```powershell
Stop-Process -Name powershell -Force
```

### Block Outbound Connections
```powershell
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}
New-NetFirewallRule -DisplayName "Block Malicious IP" -Direction Outbound -Action Block -RemoteAddress <malicious-ip>
```

### Block Malicious Activity via Firewall Based on Port, Program, or Creating Custom Rule 
- Go to Firewall / Outbound Rules / New Rule
  
<img width="864" height="778" alt="image" src="https://github.com/user-attachments/assets/70c803f7-0bb1-4dee-bd7c-77cfdf268e82" />
<img width="696" height="573" alt="image" src="https://github.com/user-attachments/assets/03546126-2393-432c-803c-a7a9dcb12397" />


---

## Step 9: Create Alerts in Splunk

Example alert query:
```spl
index=sysmon EventCode=1 CommandLine="*Invoke-WebRequest*" OR CommandLine="*EncodedCommand*"
```

Configure thresholds and actions from the Splunk Alert Manager.

---

## Appendix: Enabling PowerShell Logs in Event Viewer

1. Go to `Applications and Services Logs > Microsoft > Windows > PowerShell`
2. Double-click `Operational`
3. Copy the log path: `Microsoft-Windows-PowerShell/Operational`
4. Add it to your `inputs.conf` and restart the forwarder

Now PowerShell logs will be collected and available in Splunk for correlation with Sysmon and Security logs.
