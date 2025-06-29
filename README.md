# 🛡️ Azure Sentinel Home SOC Project

## Title
Building and Visualizing a Cloud-Based Security Operations Center Using Azure Sentinel and a Honeypot VM

---

## Introduction
This project simulates a real-world Security Operations Center (SOC) using Microsoft Azure. I set up a Windows 10 honeypot virtual machine (VM), intentionally exposed to attract attackers. Logs from this VM are sent to Microsoft Sentinel through Log Analytics Workspace, where they are queried, analyzed, and visualized. The entire setup demonstrates core SOC principles like log collection, threat detection, data enrichment, and visualization.

Led and executed by **Ibrahim Yusuf**, NACSS President, Cybersecurity and Cloud Enthusiast.

**Credit**: Inspired by the YouTube tutorial by **Josh Madakor** – *Cyber Home Lab from ZERO and Catch Attackers! (2025)*  
[Watch the original video here](https://youtu.be/g5JL2RIbThM)

---

## Project Goals

- Deploy a honeypot VM on Azure to simulate real-world threats.
- Collect and analyze security logs using Microsoft Sentinel.
- Visualize brute-force login attempts and their geolocations.
- Understand attacker behavior through enriched log data.
- Document and showcase the SOC process in a beginner-friendly way.

---

## Tools & Technologies

- Microsoft Azure (Free Subscription)
- Windows 10 Virtual Machine (Honeypot)
- Log Analytics Workspace
- Microsoft Sentinel (SIEM)
- Kusto Query Language (KQL)
- GeoIP Data (Watchlist CSV)
- Draw.io (for architecture)

---

## Step-by-Step Implementation

### 1. Azure Resource & VM Setup

- Created a resource group and virtual network.
- Deployed a Windows 10 VM named `NASA-NET-EAST-2`.
- Disabled firewall on the VM to allow open attack surface.
- Configured Network Security Group (NSG) to allow all inbound traffic.

### 2. Log Analytics Workspace & Agent

- Created a Log Analytics Workspace.
- Installed the Azure Monitor Agent (AMA) on the VM.
- Created a Data Collection Rule (DCR) to collect `SecurityEvent` logs.

### 3. Microsoft Sentinel Integration

- Enabled Microsoft Sentinel on the Workspace.
- Installed `Security Events via AMA` connector.
- Linked the workspace and configured event ingestion.

### 4. KQL Log Analysis

- Wrote queries to monitor login attempts (e.g. Event ID 4625).
- Queried and projected failed logins, usernames, IP addresses, and timestamps.

### 5. GeoIP Watchlist & Attack Map

- Uploaded `geoip-summarized.csv` file to Sentinel as a watchlist.
- Joined attacker IPs with geographic metadata (city, country, lat/long).
- Created a Sentinel workbook with a map visualization of attack sources.

### 6. Automated Incident Response

- Created a Logic App (playbook) triggered by alerts from Microsoft Sentinel.
- Logic App sends email notifications containing alert name, severity, timestamp, and compromised entity.
- Integrated `Send Data` action using Azure Log Analytics Data Collector API to log attacker metadata.
- Used a `Compose` step to format the JSON payload:
  ```json
  {
    "AlertName": "@{if(empty(triggerBody()?['AlertDisplayName']), 'UnknownAlert', triggerBody()?['AlertDisplayName'])}",
    "Entity": "@{if(empty(triggerBody()?['CompromisedEntity']), 'UnknownEntity', triggerBody()?['CompromisedEntity'])}",
    "Severity": "@{if(empty(triggerBody()?['Severity']), 'Unknown', triggerBody()?['Severity'])}",
    "Time": "@{utcNow()}"
  }
  ```
- Sent this payload to Log Analytics using `outputs('Compose')`, which created a custom log table: `TestIncidentLog_CL`.
- Verified logs via:
  ```kql
  TestIncidentLog_CL
  | sort by TimeGenerated desc
  ```
- Disabled notifications later to reduce noise, while keeping logging functional.

---

## Sample KQL Query

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
SecurityEvent
| where EventID == 4625
| order by TimeGenerated desc
| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)
| summarize FailureCount = count() by IpAddress, latitude, longitude, cityname, countryname
| project FailureCount, AttackerIp = IpAddress, latitude, longitude, city = cityname, country = countryname,
          friendly_location = strcat(cityname, " (", countryname, ")");
```

---

## Outcome

- Successfully collected and visualized attacker login attempts.
- Built a dynamic attack map showing source geolocation of failed RDP attempts.
- Demonstrated ability to ingest, query, and enrich security logs in Sentinel.
- Implemented a full incident response workflow, from alerting to automated logging.

---

## Impact & Relevance

- Developed cloud security and log analysis skills using real-world attack data.
- Built an educational SOC lab with minimal budget.
- Useful for training, resume-building, and demoing blue-team capabilities.

---

## How to Showcase on Resume

**Project**: Azure Sentinel SOC Lab  
**Tools**: Azure VM, Microsoft Sentinel, Log Analytics, AMA, KQL, GeoIP, Watchlist, Workbooks, Logic Apps  
**Highlights**:
- Built a honeypot VM to attract and study brute-force login attempts.
- Enriched logs with geolocation and visualized attacks on a live map.
- Created automated email alerts and log ingestion using Logic Apps and Azure Monitor.
- Logged attacker metadata into a custom Log Analytics table for post-incident analysis.

---

## Conclusion
This project was a deep dive into SOC workflows in the cloud. It allowed me to explore Microsoft Sentinel’s detection capabilities, log querying with KQL, and real-world attacker behavior. The experience has equipped me with the knowledge to investigate logs, automate response, and create educational content around modern blue-team practices.

---

## Author
**Ibrahim Yusuf**  
Cybersecurity Student | NACSS President | Cybersecurity and Cloud Enthusiast

