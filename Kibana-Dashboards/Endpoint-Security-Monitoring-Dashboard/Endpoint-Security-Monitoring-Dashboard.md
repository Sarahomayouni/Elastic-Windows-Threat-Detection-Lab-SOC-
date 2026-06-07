# Endpoint Security Monitoring Dashboard

## Purpose

Provide endpoint visibility into system activity using Sysmon telemetry, including process execution, network connections, and file activity. 

---

## Visualizations

### Total Events

Displays the total number of Sysmon-generated events collected from all monitored endpoints.

---

### Active Hosts

Shows the number of active endpoints generating Sysmon telemetry, helping to measure coverage and agent activity.

---

### Process Creation Events

Displays the total number of process creation events (Sysmon Event ID 1), used to monitor executed applications and system behavior.

---

### Network Connection Events

Shows all recorded network connection events (Sysmon Event ID 3), providing visibility into outbound and inbound communication.

---

### File Events

Displays file creation activity (Sysmon Event ID 11), helping detect suspicious file drops and modifications.

---

### Top Processes

Identifies the most frequently executed processes across endpoints to establish baseline behavior and detect anomalies.


---

### Suspicious Processes

Suspicious processes are identified using a rule-based detection approach applied to Sysmon Process Creation events (Event ID 1).

This detection targets commonly abused Windows binaries that are frequently leveraged in attack techniques, particularly Living-off-the-Land Binaries (LOLBins) and command execution tools.

These processes are often used in scripting, payload execution, and post-exploitation activities.

- Detection Logic

	event.code:1 AND process.name:(powershell.exe or cmd.exe or rundll32.exe or mshta.exe)

---

### Top Destination IPs

Displays the most contacted external IP addresses from endpoints, helping identify command-and-control (C2) communication or unusual outbound traffic.

---

### Network Connections by Host

Breaks down network activity per endpoint, showing which hosts generate the most network connections.

---

### External Network Connections

Focuses on connections leaving the internal network scope, helping detect suspicious external communications.

- Detection Logic:
	
		event.code:3 AND NOT destination.ip: ("10.0.0.0/8" OR "172.16.0.0/12" OR "192.168.0.0/16" OR "127.0.0.0/8" OR "169.254.0.0/16" OR "224.0.0.0/4" OR "::1")


---

### File Creation Volume

Shows the volume of file creation events over time, useful for identifying spikes in file activity such as malware drops or mass file generation.

---

### File Extensions Risk View

Analyzes created file types based on extensions to highlight potentially risky or uncommon file formats.

---

### Top Files Created

Displays the most frequently created files across endpoints to identify repeated artifacts or suspicious file patterns.

---

### Time Behaviour

This heatmap visualizes Sysmon events (Event IDs 1, 3, and 11) across time and the top 20 hosts, helping identify when and on which endpoints the highest activity occurs.

It is used to detect abnormal spikes in process execution, network connections, and file activity across specific machines over time.