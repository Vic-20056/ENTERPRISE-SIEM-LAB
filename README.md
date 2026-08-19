                                         **ENTERPRISE SIEM Home Lab: End-to-End Threat Hunting & Log Analysis**



### Project Overview
This project involves building a fully functional Security Information and Event Management (SIEM) environment from scratch. The goal was to simulate a real-world enterprise network to monitor, ingest, and analyze security alerts.

By setting up a virtualized network, I routed raw endpoint telemetry into a centralized analyst dashboard. I then launched simulated cyber attacks to test the pipeline and successfully hunted down the malicious activity. 

## LAB ARCHITECTURE
Target Endpoint: Windows 10 Virtual Machine.

SIEM Indexer: Ubuntu Linux running Splunk Enterprise.

Attacker Machine: Kali Linux.

Telemetry & Forwarding Tools: Microsoft Sysmon and Splunk Universal Forwarder.

# Phase 1: Infrastructure & Environment Setup
1. Centralized Indexer Configuration
I installed Splunk Enterprise on an Ubuntu Linux virtual machine. This served as the primary SIEM brain to receive and index all incoming network logs. 
![Splunk Ubuntu Installation](Screenshot-2026-08-18-210300.png)

2. Endpoint Telemetry Generation
On the Windows 10 target, I installed Microsoft Sysmon. I utilized the industry-standard SwiftOnSecurity configuration file to ensure we captured deep, process-level system activity.
![Sysmon Installation on Windows](Screenshot-2026-08-18-214617.png)
![Windows Event Viewer Sysmon Logs](Screenshot-2026-08-18-232505_2.png)


3. Log Forwarder Deployment
I installed the Splunk Universal Forwarder on the Windows machine. This acted as the digital bridge to automatically collect local system logs and ship them to the Ubuntu server. 

# Phase 2: Log Routing & Ingestion
To connect the systems, I manually configured the forwarder's internal files. I edited the outputs.conf file to point directly to the Ubuntu server's IP address on port 9997.

I then modified the inputs.conf file to specifically read the raw XML data coming from Sysmon. Later, I expanded this configuration to also capture standard Windows Security logs. 
![Successful Sysmon XML Ingestion](Screenshot-2026-08-19-000248.jpg)
![Successful Windows Security Log Ingestion](Screenshot-2026-08-19-012856_2.png)


# Phase 3: Attack Simulations
With the monitoring pipeline active, I shifted to the Kali Linux machine to generate malicious network noise.

1. Network Reconnaissance
I ran an aggressive Nmap scan (nmap -A -p-) against the Windows target. This simulated a hacker rattling the digital doorknobs to find vulnerable ports. 

2. Brute-Force Password Attack
I utilized Hydra to execute a dictionary attack against the Windows SMB service. I hammered the administrator account with fake passwords to trigger authentication failure alarms. 

# Phase 4: Challenges & Troubleshooting
Real-world security engineering requires intense troubleshooting. Here is a breakdown of every roadblock encountered and how it was successfully resolved. 

**Problem 1: Splunk Disk Space Safety Lock**
![Splunk Minimum Free Disk Space Error](Screenshot-2026-08-18-223348.png)

**The Issue**: Splunk paused all searching because the Ubuntu disk fell below the default 5000MB free space limit.

**The Fix**: I opened the Ubuntu terminal and edited the server.conf file using Nano. I added a [diskUsage] override to lower the minFreeSpace threshold to 500MB, then restarted Splunk. 

**Problem 2: Network Connectivity & Closed Ports**

**The Issue**: The Splunk dashboard showed zero events, indicating a broken bridge between Windows and Ubuntu.
![Empty Dashboard Zero Events](Screenshot-2026-08-18-224654.png)

**The Fix**: I used Test-NetConnection in Windows PowerShell to verify the network routing was active. I then opened port 9997 on the Ubuntu server using the UFW firewall to allow the logs inside. 
![Verifying Port 9997 is Listening on Ubuntu](Screenshot-2026-08-18-231553.png)

Problem 3: Forwarder Service Permissions

**The Issue**: The forwarder was running on a restricted user account, silently blocking it from reading the highly sensitive Sysmon logs.

**The Fix**: I opened the Windows Services menu, changed the SplunkForwarder "Log On" account to the Local System profile, and completely restarted the service. 

Problem 4: The Hidden .txt Extension Trap

**The Issue**: Windows Notepad secretly saved the configuration file as inputs.conf.txt. Splunk completely ignored it because of the wrong file extension.
![Misconfigured input.conf file](Screenshot-2026-08-18-222554.png)
![Splunkd.log showing Sysmon being ignored](Screenshot-2026-08-18-234107_2.png)

**The Fix**: Instead of relying on a graphical text editor, I wrote a custom PowerShell script. The script forcefully injected the correct text directly into a raw .conf file and rebooted the forwarder. 

Problem 5: Corrupted Fishbucket Memory

**The Issue**: Splunk's internal memory (the fishbucket) got confused during configuration changes and permanently ignored older Sysmon logs to prevent duplicates.

**The Fix**: I stopped the background service and completely deleted the hidden fishbucket folder. This wiped the forwarder's memory, forcing it to ingest all 4,700 logs from scratch! 

Problem 6: Invisible Brute-Force Attacks

**The Issue**: The Hydra brute-force attack successfully ran, but no EventCode 4625 (Failed Logon) logs appeared in Splunk.
![Empty Search Results Due to Windows Firewall](Screenshot-2026-08-19-013122_2.png)

**The Fix**: I discovered the Windows Defender Firewall was silently dropping the malicious traffic before the password check occurred. I disabled the firewall to expose the service. I also modified the Windows Local Security Policy to force the operating system to audit both successful and failed logons. 

# Phase 5: Threat Hunting Results
After successfully troubleshooting the pipeline and lowering the endpoint defenses, I pivoted to the SIEM dashboard. I utilized raw text SPL queries to hunt down the simulated attacks. 

Identified System Access: Successfully queried and visualized EventCode 4624 (Successful Logon) and EventCode 4672 (Special Privileges Assigned) to track administrative access.
![Event Code 4672 Special Privileges](Screenshot-2026-08-19-014215_3.png)
![Event Code 4624 Successful Logon](Screenshot-2026-08-19-015621_2.png)

Captured Physical Interactions: Identified EventCode 4801 the exact moment the workstation was manually locked during testing. 
![Event Code 4801 Workstation Locked](Screenshot-2026-08-19-020213_2.png)

Detected Brute-Force Activity: Captured a massive spike of EventCode 4625 (Failed Logon) alerts generated by the simulated Hydra attack and manual password failures. 
![Event Code 4625 Failed Logon Attempts](Screenshot-2026-08-19-020918_2.png)

# Conclusion
This hands-on lab successfully demonstrated the complete lifecycle of a SIEM pipeline. It required navigating complex network routing, overcoming strict operating system security policies, and writing granular data ingestion scripts. The result is a highly functional, enterprise-grade threat hunting environment! 