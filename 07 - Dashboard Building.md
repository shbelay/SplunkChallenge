# Day 7: Building a Custom Splunk Dashboard

## Objective
Create a custom Splunk dashboard using Palo Alto firewall Data Filtering logs to visualize traffic patterns and threat information.

## Prerequisites
- Download the sample log file: `PA_HaxLab-01-DF-logs.log`
- Open two Splunk tabs:
  - One to run and test SPL queries
  - One to build the dashboard

<img width="1917" height="936" alt="SplunkDashboard" src="https://github.com/user-attachments/assets/96b9d3d0-f0bd-40f1-93c8-63b1574f650a" />


---

## Guide

### Step 1: Create a Time Filter
1. Click **Add Input**
2. Select **Time**
3. Set **Label** to `Time Range`
4. Set **Token** to `token_for_time`

---

### Step 2: Add Blocked Traffic Count
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log" action_taken=blocked
| stats count by action_taken
| table count
```

#### Dashboard Panel Settings
- Panel Type: **Single Value**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `Blocked`
- Use same SPL query in the **Search String** field

---

### Step 3: Add Allowed Traffic Count
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log" action_taken=allowed
| stats count by action_taken
| table count
```

#### Dashboard Panel Settings
- Panel Type: **Single Value**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `Allowed`

---

### Step 4: Add Total Traffic Count
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log"
| stats count
```

#### Dashboard Panel Settings
- Panel Type: **Single Value**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `Total`

---

### Step 5: Visualize Based on File Types
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log"
| table file_type
| stats count by file_type
```

#### Dashboard Panel Settings
- Panel Type: **Bar Chart**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `File Types`

---

### Step 6: Visualize Blocked Destination IP in Geo-Location Style
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log" action="block"
| table dst_ip
| iplocation dst_ip
| stats count by Country
| geom geo_countries featureIdField="Country"
```

#### Dashboard Panel Settings
- Panel Type: **Statistics Table** or **World Map**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `Destination IP Geo-location`

---

### Step 7: Create a Top Talker Panel
#### SPL Query
```spl
sourcetype="Panos-df" source="PA_HaxLab-01-DF-logs.log"
| stats count by src_ip, dst_ip
| sort - count
| head 20
```

#### Dashboard Panel Settings
- Panel Type: **Statistics Table**
- Time Picker: **Shared time** (`token_for_time`)
- Content Title: `Top Talkers`

---

## Summary
This guide walks through building an interactive Splunk dashboard using Palo Alto firewall logs. It includes visual panels for blocked/allowed/total traffic, file types, geo-location of blocked IPs, and top talkers.

These dashboards can assist security teams in identifying patterns, threats, and anomalous activity at a glance.
