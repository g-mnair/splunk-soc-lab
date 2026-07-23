# SSH Brute-Force Investigation Using Splunk

## 📌 Incident Investigation Overview
This lab demonstrates how Splunk can be used to investigate an SSH brute-force attack against a Linux host. Using authentication logs collected from /var/log/auth.log, I identified failed login attempts, extracted useful fields from raw events, assessed whether any authentication attempts succeeded, and documented recommended response actions for a SOC analyst.

### 💡 Why I Built This
To be an effective analyst, one must understand how adversarial tools translate into raw telemetry under the hood. By launching a controlled brute-force simulation using industry-standard penetration testing toolsets and immediately pivoting into the SIEM interface, this project provides transparent, hands-on proof of my capability to handle an inbound credential access alert from initial trigger to host-level mitigation.

---

## 1. Attack Emulation Phase (Threat Actor Perspective)
To generate realistic malicious telemetry within the environment, a high-velocity automated credential harvesting attack was launched against the Linux infrastructure.

### Step 1: Target Endpoint Configuration
The secure shell daemon (`sshd`) was initialized on the target system to expose the native network authentication interface:
```bash
sudo systemctl enable ssh --now
```

### Step 2: Automated Brute-Force Execution
An automated password-guessing loop was launched against a target profile using the standard `rockyou.txt` password directory to simulate an external threat group:
```bash
hydra -l target_user -P /usr/share/wordlists/rockyou.txt ssh://127.0.0.1 -t 4 -V
```
*The simulation was manually terminated after 45 seconds using `Ctrl + C` to restrict excessive daily indexing volume while generating hundreds of high-fidelity authentication failure events inside `/var/log/auth.log`.*

---

## 2. SIEM Triage & Scope Isolation (SOC Analyst Perspective)

### Step 1: Advanced Attacker Footprint Mapping
Because raw Linux syslogs do not parse distinct user or network fields automatically out of the box, advanced Search Processing Language (SPL) was utilized to inject regular expression (`rex`) extractions. This query isolates the source IP, measures attack velocity, tracks targeted user profiles, and sorts the results to uncover the core threat footprint:

```splunk
index="linux_log" "Failed password" 
| rex field=_raw "for (?:invalid user )?(?<user>\S+)" 
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| stats count as failed_attempts, dc(user) as unique_users_targeted by src_ip 
| sort - failed_attempts
```

*   **Analyst Evaluation:** A massive surge of sequential failures was confirmed. Because the attack loop ran locally inside the virtual architecture, the source IP aggregates as a loopback identifier (`127.0.0.1`), mimicking either an internal compromised asset or a localized privilege escalation attempt.

![Attacker Footprint Mapping Verification](assets/01_attacker_footprint_mapping.png)

---

## 3. The Ultimate SOC Question: Assessing Blast Radius

### Step 1: Verifying Authentication Success
A brute-force attack is an operational nuisance, but a *successful* brute-force attack represents a critical data breach. The malicious source network identifier was audited to ensure zero successful access tokens were generated during the attack window:

```splunk
index="linux_log" "Accepted password" 
| rex field=_raw "for (?<user>\S+)" 
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| table _time src_ip user _raw
```

*   **Analyst Evaluation:** This query returned **0 results** (empty data grid), verifying that perimeter credential harvesting attempts failed, no valid sessions were established, and the direct blast radius of the attack was effectively zero.

![Authentication Success Audit Verification](assets/02_authentication_success_audit.png)

### Step 2: Post-Auth Kill Chain Assessment Strategy
While the direct authentication check returned zero entries, a robust analytical mindset recognizes that authentication compromise is merely step one of an adversary's execution tree. If a successful login (`Accepted password`) had been identified, the immediate triage workflow would require running the following secondary inspection queries to map post-compromise actions:
1. **Audit Privileged Command Invocations:** Inspecting the system logs to identify if the compromised user account attempted to execute unauthorized root actions via `sudo`.
2. **Track Process Creation Telemetry:** Cross-referencing endpoint logs to check for spawning interactive terminal loops, netcat reverse shells, or defensive evasion commands executed during that session window.

---

## 4. Operational Velocity Charting
To map the definitive operational spike of the Hydra toolset, a timechart aggregate was mapped to isolate the exact velocity curve over the ingestion window:

```splunk
index="linux_log" "Failed password" 
| rex field=_raw "for (?:invalid user )?(?<user>\S+)" 
| timechart span=10s count by user
```

![Timechart Operational Velocity Graph](assets/03_timechart_operational_velocity.png)

---

## 5. Tier-1 (L1) Containment & Remediation Standard Operating Procedure
Once the scope was fully realized and the blast radius was confirmed as zero, the following standard L1 containment workflow was documented for incident remediation:

1. **Network Level Isolation Request:** Gathered the verified malicious source IP (`127.0.0.1`) and drafted an emergency block request ticket to the Network Security team to drop inbound traffic at the perimeter gateway. *(Simulated locally on the host using host-based firewall rules: `sudo iptables -A INPUT -s 127.0.0.1 -p tcp --dport 22 -j DROP`)*.
2. **Active Process Termination:** Executed an immediate process termination to kill any lingering background automation threads running on the endpoint (`sudo pkill -f hydra`).
3. **Identity Lockdown:** Triggered a temporary account lockout policy for the targeted user account profile within the identity management console to protect against secondary credential stuffing attempts.
4. **Escalation Hand-Off:** Documented the forensic timeline and passed the scoped investigation ticket up to the Tier-2 Incident Response team for continuous indicator of compromise (IoC) tracking.
