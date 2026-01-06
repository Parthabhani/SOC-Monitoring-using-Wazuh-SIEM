# SOC-Monitoring-using-Wazuh-SIEM
The goal of this lab was to build a mini SOC (Security Operations Center) environment to:  Collect authentication logs from a Linux system  Detect suspicious SSH login activity  Generate and analyze security alerts using Wazuh SIEM  Understand real-world SOC workflows including alert triage and troubleshooting

2️⃣ Environment Details
Operating System: Linux (Kali / Debian-based)
SIEM Tool: Wazuh (All-in-One deployment)
Logging Mechanism: rsyslog (file-based logging)
Attack Simulation: SSH invalid login attempts
Monitoring Interface: Wazuh Dashboard (Web UI)

3️⃣ Installing Wazuh (All-in-One)
We installed Wazuh using the official all-in-one installer, which includes:
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
This setup simulates a SOC server in a real environment.

After installation:
Dashboard was accessible via https://localhost
Manager services were verified using systemctl

4️⃣ Understanding Agent Behavior (Important Learning)
In an all-in-one Wazuh deployment, the manager can monitor the local system directly.
The dashboard may show 1 agent with status “Never connected”.
This is normal behavior and does not indicate a failure.
Local log ingestion still works correctly without a separate agent handshake.

🔑 Key SOC Insight:
Agent connection status is less important than actual alert generation.

5️⃣ Preparing SSH for Attack Simulation
Initially, SSH connections failed because the SSH service was not running.
Actions taken:
Installed SSH server
Started and enabled SSH service
Verified port 22 was listening
This ensured SSH authentication events could be generated.

6️⃣ Problem Encountered: No Authentication Logs Detected
Even after SSH failures:
No alerts appeared in Wazuh
/var/log/auth.log did not exist
Logs were only available via journalctl
Root Cause:
The Wazuh build on this system does not support journald ingestion
Authentication logs were not being written to files

7️⃣ Installing and Configuring rsyslog
To resolve this, we configured file-based logging using rsyslog.
Steps:
Installed rsyslog manually
Enabled and started rsyslog service
Configured rsyslog to write authentication logs to a file
Configuration added:
auth,authpriv.*    /var/log/auth.log
After restarting rsyslog:
/var/log/auth.log was successfully created
SSH login failures were written to the file

8️⃣ Verifying Authentication Logs
We generated SSH failures using:
ssh wronguser@localhost
Then verified logs:
tail -n 20 /var/log/auth.log
Observed entries such as:
Invalid user attempts
Failed password messages
This confirmed the logging pipeline was working.

9️⃣ Configuring Wazuh to Read Authentication Logs
Next, we configured Wazuh to ingest the authentication logs.
Configuration added to /var/ossec/etc/ossec.conf:
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

Important Notes:
syslog format is required for auth logs
journald format was removed due to incompatibility
Configuration was validated before restarting services

🔄 Restarting and Validating Wazuh
Wazuh Manager was restarted successfully
No logcollector or configuration errors occurred
Services entered the active (running) state

🔥 Generating and Detecting Security Alerts
Attack Simulation:
Repeated SSH login attempts with an invalid user.
Detection:
Wazuh generated authentication failure alerts
Alerts were visible in:
Security Events → Authentication
Raw alerts (alerts.json)
This confirmed end-to-end SOC detection.

🔍 Alert Analysis (SOC Perspective)
Each alert was analyzed for:
Source IP
Username used
Frequency of attempts
Severity level
The activity was classified as:
SSH brute-force / invalid login attempt
No successful compromise detected

🧠 Key SOC Learnings from This Lab
SIEM setup requires troubleshooting, not just installation
Log ingestion is more important than agent status
File-based logging is widely used in production SOCs
Alerts must be validated using both dashboard and raw logs
SOC work focuses on detection and analysis, not exploitation

📌 Project Outcome
This lab successfully demonstrated:
Centralized log monitoring
Real-time security alert generation
Practical SOC troubleshooting
Incident-level alert analysis
