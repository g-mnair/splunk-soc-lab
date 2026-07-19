# Module 01: Splunk Lab Environment Setup & Cross-Platform Ingestion

## Project Overview
The objective of this lab was to build a centralized log collection environment using Splunk Enterprise and the Splunk Universal Forwarder. Splunk Enterprise was deployed on a Windows 11 system to act as the monitoring server, while the Universal Forwarder was installed on a Kali Linux virtual machine to forward Linux system and authentication logs.

## Architecture & Lab Environment Details
* **Central SIEM Monitoring System:** Splunk Enterprise (v9.2.1) hosted on Windows 11
* **Monitored Endpoint:** Kali Linux rolling distribution hosted via Oracle VirtualBox
* **Log Forwarder:** Splunk Universal Forwarder (v9.2.1) deployed on the Linux endpoint
* **Default Network Ingestion Port:** TCP/9997

---

## Deployment & Configuration Procedures

### 1. Central SIEM Platform Configuration (Windows Host)
1. Deployed Splunk Enterprise via the official Windows MSI installer package.
2. Administrator credentials were configured.
3. The Splunk Web interface was successfully launched.
4. Initialized a new data listening port mapping on **TCP Port 9997** to listen for inbound forwarder traffic. The platform was prepared to receive data from monitored endpoints.

### 2. Forwarder Deployment & Directory Setup (Kali Linux Endpoint)
1. Pulled the archived deployment package utilizing the command line download tool:

```bash
wget -O splunkforwarder-9.2.1-78803f08aabb-Linux-x86_64.tgz "https://download.splunk.com/products/universalforwarder/releases/9.2.1/linux/splunkforwarder-9.2.1 78803f08aabb-Linux-x86_64.tgz"
```

2. Unpacked the zipped repository to retrieve the system binary structure:
   ```bash
   tar -xvzf splunkforwarder-9.2.1-78803f08aabb-Linux-x86_64.tgz
   ```
3. Migrated files to the root system operating directory (/opt) and accessed the execution paths:
   ```bash
   sudo mv splunkforwarder /opt/ && cd /opt/splunkforwarder/bin
   ```
4. Accepted the software licensing policy terms and provisioned local agent admin credentials:
   ```bash
   sudo ./splunk start --accept-license
   ```

### 3. Log Ingestion File Configuration (`inputs.conf`)
To establish structured forwarding policies for specific critical Linux log streams, the local setup configuration file was updated via `nano /opt/splunkforwarder/etc/system/local/inputs.conf` to monitor core authentication paths:

```ini
[monitor:///var/log/syslog]
disabled = false
index = main
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = false
index = main
sourcetype = authlog
```

### 4. Establishing Network Forwarding Connectivity
1. Programmed the endpoint forwarder instance to actively routing traffic targeting the central Windows host IP matrix on port 9997:
   ```bash
   sudo ./splunk add forward-server <Windows_Host_IP>:9997
   ```
2. Opened the local outbound Linux Uncomplicated Firewall (UFW) mapping to guarantee communication paths:
   ```bash
   sudo ufw allow out to any port 9997 proto tcp
   ```

---

## Visual Verification & Telemetry Validation
To validate successful multi-platform integration, the central Splunk console was checked via `http://localhost:8000`. Accessing the **Search & Reporting** interface and opening **Data Summary** confirms that the Linux VM host is officially communicating and delivering live logs.

*[Insert a screenshot of your Splunk Data Summary or search page showing your Kali Linux host logs here]*

---

---

## Challenges Encountered & Problem Solving

### 1. Delayed Log Collection & Ingestion Latency
* **The Problem:** During testing, newly generated security logs from the Kali Linux endpoint occasionally required several minutes before becoming visibly indexed within the central Splunk search interface.
* **The Resolution:** After systematically verifying the local forwarder configuration settings, the latency was monitored and confirmed to be standard operational forwarding and indexing queue delays rather than a configuration error.

### 2. Network Communication & Forwarding Failures
* **The Problem:** Initial log forwarding attempts failed to reach the central Windows instance due to network communication roadblocks stemming from dynamic IP shifts, incorrect port assignments, or host-based firewall settings.
* **The Resolution:** The network pipeline was systematically troubleshooted. The `outputs.conf` destination configurations, the central TCP/9997 receiving port status, and both Windows Defender Inbound rules and Linux UFW rules were reviewed and corrected until persistent cross-platform communication was successfully established.



