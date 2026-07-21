# Module 02: Search Processing Language (SPL) Fundamentals

## 📌 Objective

The objective of this lab is to explore the fundamentals of Splunk's Search Processing Language (SPL) by performing searches, filtering events, transforming results, and visualizing Linux log data collected from the configured lab environment.

---

## 🎯 Learning Outcomes

Upon successful completion of this lab, you will be able to:

- Search indexed log data to locate specific security events.
- Filter, display, rename, and sort relevant log fields.
- Generate event counts and basic statistical summaries.
- Organize search results into tables and charts.
- Perform searches using time modifiers and wildcard characters.
- Build foundational SPL queries used in day-to-day security monitoring.

---

## 📋 Prerequisites

Before starting this lab, ensure the following requirements have been completed:

- Splunk Enterprise is installed and running.
- Splunk Universal Forwarder is configured on the Kali Linux virtual machine.
- Linux log files have been successfully indexed into Splunk.
- Access to the Splunk Search & Reporting application is available.

---

# 1. Introduction to SPL

Splunk Search Processing Language (SPL) is the primary language used to search, filter, analyze, and visualize machine-generated data. Throughout this lab, SPL is used to examine Linux authentication and system logs collected from the Kali Linux virtual machine.

Rather than manually reviewing raw log files, SPL enables analysts to efficiently search large datasets, identify relevant events, summarize log activity, and support security investigations.

---

# 2. Basic Filtering & Formatting Reference Table

The following table introduces the fundamental SPL commands used throughout this lab for filtering, organizing, and formatting search results.

| Command | Purpose | Example Query | Lab Purpose |
| :--- | :--- | :--- | :--- |
| **Index Search** | Search all data inside a specific storage bucket. | `index=linux_log` | Restricts the search to the relevant project logs instead of searching every indexed event. |
| **Host Filter** | Search events originating from a specific machine. | `index=linux_log host=kali` | Focuses the investigation on a particular system. |
| **Source Filter** | Search events from a specific log file. | `index=linux_log source="/var/log/auth.log"` | Targets authentication logs for user activity analysis. |
| **Sourcetype Filter** | Search events based on their log format. | `index=linux_log sourcetype=syslog` | Limits searches to system log events parsed using the syslog format. |
| **Table** | Display selected fields in a structured format. | `index=linux_log \| table _time, host, source` | Produces cleaner search results by displaying only the required fields. |
| **Sort** | Arrange search results chronologically or numerically. | `index=linux_log \| sort -_time` | Displays the most recent events first. |
| **Rename** | Rename a field within the search results. | `index=linux_log \| rename host AS Machine` | Improves readability when presenting search results. |
| **Wildcards** | Search using placeholder characters. | `index=linux_log source="*/auth.log"` | Matches log file paths even when directory structures vary. |

---

# 3. Core SPL Concepts & Commands Covered

## 1. Data Filtering & Formatting (`fields`, `table`, `dedup`, `sort`)

- **Objective:** Reduce unnecessary information and focus on relevant log data during analysis.
- **Hands-on Application:** Filtered raw system logs to display authentication-related events while removing unnecessary metadata.
- **Key Command:** `table` organizes selected fields into a structured format, while `dedup` removes duplicate events from the same host.

---

## 2. Statistical Transformation (`stats`, `chart`, `timechart`)

- **Objective:** Summarize raw log events into meaningful statistics.
- **Hands-on Application:** Counted authentication events and summarized activity across different users and hosts.
- **Key Command:** `stats count by user, host` converts raw events into an organized summary table.

---

## 3. Dynamic Field Manipulation (`eval`, `if`, `case`)

- **Objective:** Create calculated fields and apply conditional logic within search results.
- **Hands-on Application:** Assigned custom risk levels (Low, Medium, High) based on failed login thresholds.
- **Key Command:** `eval risk_level = if(count > 10, "High", "Low")`

---

## 4. Search Optimization & Performance (`where`, `search`)

- **Objective:** Apply search filters to improve search efficiency and reduce unnecessary results.
- **Hands-on Application:** Used post-search filtering techniques to isolate suspicious authentication activity from larger result sets.

---

# 4. Filtering Events & User Activity

After identifying the appropriate data source, SPL searches can be refined to locate specific authentication events and user activities.

### Failed SSH Logins

Displays failed SSH authentication attempts recorded in the authentication log.

```spl
index="linux_log" failed password 
```


---

### Successful Logins

Displays successful SSH authentication events.

```spl
index="linux_log" Accepted password
```


---

### Sudo Commands

Displays commands executed with elevated administrative privileges.

```spl
index="linux_log" sudo
```

---

### Search for a Specific User

Returns all events associated with the specified username.

```spl
index="linux_log" alex
```


---

# 5. Transforming & Statistical Commands

Transforming commands summarize raw events into organized statistics and visual representations that simplify log analysis.

## Event Counting & Summary

### Count Total Events

Returns the total number of events matching the search criteria.

```spl
index= "linux_log"
| stats count
```


---

### Count by Category

Groups event counts by host.

```spl
index="linux_log"
| stats count by host
```


---

### Count by Sourcetype

Groups event counts by sourcetype.

```spl
index="linux_log"
| stats count by sourcetype
```


---

### Chart Events by Sourcetype

Displays the distribution of events grouped by host and sourcetype.

```spl
index="linux_log"
| chart by host,sourcetype
```

---


# 6. Time-Based Searches

Time modifiers restrict searches to a defined time period, reducing the number of returned events and improving search performance.

- **Last 15 Minutes**

```spl
index="linux_log" earliest=-15m
```

- **Last Hour**

```spl
index="linux_log" earliest=-1h
```

- **Last 24 Hours**

```spl
index="linux_log" earliest=-24h
```

---

# 7. Practical Security Monitoring Examples

The following examples demonstrate common SPL searches used during Linux log analysis and security monitoring.

### Identify Failed Login Volume by Host

```spl
index="linux_log" Failed password
| stats count by host
```

📸 *failed_login_volume_by_host_verification.png*

---

### Identify Successful Logins by Host

```spl
index="linux_log" Accepted password
| stats count by host
```

📸 *successful_login_by_host_verification.png*

---
### Identify Failed login by User

```spl
index="linux_log" failed password
| stats count by alex
```



### Successful Login Activity Above a Threshold

```spl
index="linux_logs" Accepted password
| stats count by host
| where count > 5
```

📸 *successful_login_threshold_verification.png*

---

### Monitor Administrative Sudo Executions

```spl
index="linux_log" sudo
```

📸 *admin_sudo_execution_verification.png*

---

### Track Total Events Over Time

```spl
index="linux_log"
| timechart count
```

📸 *timechart_events_timeline_verification.png*

---

# 8. Conclusion

In this lab, the fundamental features of Splunk Search Processing Language (SPL) were explored by searching, filtering, transforming, and summarizing Linux log data. The exercises demonstrated how SPL can be used to investigate authentication events, organize search results, generate statistical summaries, and visualize system activity.

The concepts covered in this lab provide the foundation for the upcoming modules on security detection, brute-force investigations, threat hunting, and Splunk dashboard creation.
