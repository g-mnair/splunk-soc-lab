# Module 02: Search Processing Language (SPL) Fundamentals

## 📌 Objective
The objective of this lab is to learn the fundamentals of Splunk's Search Processing Language (SPL). You will practice querying, filtering, transforming, and visualizing Linux log data collected from your lab environment.

## 🎯 Learning Outcomes
By the end of this lab, you will be able to:
* Search indexed log data and filter specific security events.
* Select, display, rename, and sort specific data fields.
* Count events and generate clear statistics.
* Aggregate data into clean tables and charts.
* Limit searches using time ranges and wildcards.
* Build basic SPL queries for daily security monitoring.

## 📋 Prerequisites
* Splunk Enterprise installed and running.
* Universal Forwarder configured on the Kali Linux VM.
* Linux logs successfully indexed into Splunk.
* Access to the default Splunk Search & Reporting app.

---

## 1. Introduction to SPL
Splunk Search Processing Language (SPL) is used to search, filter, analyze, and visualize machine-generated data. SOC analysts use SPL every day to investigate alerts, hunt threats, identify suspicious behavior, and create detections. Without SPL, logs are just raw text; SPL turns them into actionable intelligence.

---

## 2. Basic Filtering & Formatting Reference Table
The table below outlines the foundational commands used to filter and format raw logs during an investigation.

| Command | Purpose | Example Query | Why We Use It |
| :--- | :--- | :--- | :--- |
| **Index Search** | Search all data inside a specific storage bucket. | `index=linux_logs` | Limits our search to only the relevant project logs instead of searching everything. |
| **Host Filter** | Look for data coming from a specific machine. | `index=linux_logs host=kali` | Helps isolate an investigation to one specific compromised system. |
| **Source Filter** | Look for logs from a specific file path. | `index=linux_logs source="/var/log/auth.log"` | Targets the exact log file responsible for authentication events. |
| **Sourcetype Filter** | Search by the specific format of the data. | `index=linux_logs sourcetype=syslog` | Helps Splunk apply the correct parsing rules to standard system logs. |
| **Table** | Display data in clean, customized columns. | `index=linux_logs \| table _time, host, source` | Cleans up the screen by hiding messy metadata and showing only needed fields. |
| **Sort** | Order your results chronologically or numerically. | `index=linux_logs \| sort -_time` | Putting a minus (`-`) before the field sorts events from newest to oldest. |
| **Rename** | Change a column header name on the fly. | `index=linux_logs \| rename host AS Machine` | Makes presentation data and reports easier for non-technical managers to read. |
| **Wildcards** | Search using a placeholder for unknown characters. | `index=linux_logs source="*/auth.log"` | Finds files matching the pattern even if the folder path changes. |

---

## 3. Core SPL Concepts & Commands Mastered

### 1. Data Filtering & Formatting (`fields`, `table`, `dedup`, `sort`)
* **Objective:** Clean up noisy data streams and extract high-value telemetry.
* **Hands-on Application:** Narrowed down raw syslogs to view only authentication actions by removing redundant metadata fields.
* **Key Command:** `table` creates a structured grid; `dedup` eliminates repetitive event bursts from the same host.

### 2. Statistical Transformation (`stats`, `chart`, `timechart`)
* **Objective:** Aggregate raw data into actionable security metrics.
* **Hands-on Application:** Calculated event frequencies and counted anomalies across different users and hosts.
* **Key Command:** `stats count by user, host` transforms thousands of raw log lines into a clean summary table.

### 3. Dynamic Field Manipulation (`eval`, `if`, `case`)
* **Objective:** Create conditional variables and normalize text data on the fly.
* **Hands-on Application:** Built custom risk levels (Low, Medium, High) based on specific failed login thresholds.
* **Key Command:** `eval risk_level = if(count > 10, "High", "Low")`.

### 4. Search Optimization & Performance (`where`, `search`)
* **Objective:** Write highly optimized queries to reduce search execution time over massive datasets.
* **Hands-on Application:** Applied post-filtering techniques to extract specific suspicious activity from pre-aggregated results.

---

## 4. Filtering Events & User Activity
Once you know the source of your data, you can filter for specific system events or strings using basic keyword queries:

* **Failed SSH logins:** Find whenever someone inputs a wrong password.
  ```splunk
  index=linux_logs "Failed password"
  ```
  📸 *[failed_ssh_logins_verification.png]*

* **Successful logins:** Monitor who is successfully entering the environment.
  ```splunk
  index=linux_logs "Accepted password"
  ```
  📸 *[successful_logins_verification.png]*

* **Sudo commands:** Track when users run administrative or privileged actions.
  ```splunk
  index=linux_logs sudo
  ```
  📸 *[sudo_commands_verification.png]*

* **Search for a specific user:** Isolate all logs containing a specific name to see all related activity.
  ```splunk
  index=linux_logs Gayathri
  ```
  📸 *[user_search_verification.png]*

---

## 5. Transforming & Statistical Commands
Transforming commands turn raw log events into organized data summaries, counts, and visual metrics.

### Event Counting & Scarcity
* **Count Total Events:** Counts every single event returned by the search.
  ```splunk
  index=linux_logs | stats count
  ```
  📸 *[stats_count_total_verification.png]*

* **Count by Category:** Breaks down the total log count based on specific hosts or data types.
  ```splunk
  index=linux_logs | stats count by host
  ```
  📸 *[stats_count_by_host_verification.png]*

  ```splunk
  index=linux_logs | stats count by sourcetype
  ```
  📸 *[stats_count_by_sourcetype_verification.png]*

* **Top & Rare Events:** `top` finds the most common values in a dataset, while `rare` finds the least common (anomalies).
  ```splunk
  index=linux_logs | top source
  ```
  📸 *[top_source_verification.png]*

  ```splunk
  index=linux_logs | rare source
  ```
  📸 *[rare_source_verification.png]*

* **Chart Events by Sourcetype:**
  ```splunk
  index=linux_logs | chart count by sourcetype
  ```
  📸 *[chart_by_sourcetype_verification.png]*

### Advanced Statistical Aggregation
* **Aggregating Activity by Host & User:**
  ```splunk
  index=linux_logs sourcetype=syslog "sshd"
  | stats count, dc(process) as unique_processes by host, user
  | rename count as "Total Attempts"
  | sort - "Total Attempts"
  ```
  * *Why we use it:* This counts interactions and uses `dc` (distinct count) to see if an identity is spawning multiple unique system processes, which is highly unusual.
  📸 *[advanced_host_user_aggregation_verification.png]*

* **Dynamic Threat Grading with Eval:**
  ```splunk
  index=linux_logs sourcetype=syslog "sshd"
  | stats count by user
  | eval analyst_priority = case(count > 50, "CRITICAL", count > 15, "MEDIUM", 1=1, "LOW")
  | sort - count
  ```
  * *Why we use it:* The `eval` and `case` functions allow us to inject custom logic to automatically flag high-volume users for rapid analyst triage.
  📸 *[dynamic_threat_grading_verification.png]*

---

## 6. Time-Based Searches
Restricting your search to specific time windows dramatically speeds up Splunk and focuses your investigation. You can select these in the UI or type them directly into the query bar using the `earliest` command:

* **Last 15 minutes:** `index=linux_logs earliest=-15m`
* **Last hour:** `index=linux_logs earliest=-1h`
* **Last 24 hours:** `index=linux_logs earliest=-24h`

---

## 7. Practical Security Monitoring Examples
Here is a collection of production-ready baseline searches used for monitoring infrastructure:

* **Identify Failed Login Volume by Host:**
  ```splunk
  index=linux_logs "Failed password" | stats count by host
  ```
  📸 *[failed_login_volume_by_host_verification.png]*

* **Identify Successful Logins by Host:**
  ```splunk
  index=linux_logs "Accepted password" | stats count by host
  ```
  📸 *[successful_logins_by_host_verification.png]*

* **Monitor Administrative Sudo Executions:**
  ```splunk
  index=linux_logs sudo
  ```
  📸 *[admin_sudo_execution_verification.png]*

* **Track Total Events Over a Timeline:**
  ```splunk
  index=linux_logs | timechart count
  ```
  📸 *[timechart_events_timeline_verification.png]*

---

## 8. Conclusion
In this lab, we successfully mastered how to search, filter, and transform raw data streams into organized security intelligence. These fundamental SPL techniques form the absolute foundation for our upcoming modules involving active security detections, brute-force investigations, and advanced threat hunting.
