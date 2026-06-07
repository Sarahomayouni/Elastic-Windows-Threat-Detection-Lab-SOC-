# Security Monitoring Lab - Deployment Guide

This document describes the full deployment architecture and setup process of the Security Monitoring Lab built on Elastic Stack for endpoint security monitoring and SIEM analysis.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Lab Environment

**Host:**
- Windows 11
- Intel Core i9 14900HX
- 32 GB RAM DDR5 5600
- 1 TB SSD

**Virtualization:**
- Oracle VirtualBox 7.2.8

**Guest OS:**
- Ubuntu Server 24.04 LTS
	- 4 vCPU
	- 14 GB RAM
	- 100 GB Storage
	- Docker Engine

**Elastic Stack:**
- Elasticsearch 8.13.0
- Kibana 8.13.0
- Filebeat 9.4.1
- Winlogbeat 9.4.1

**Security:**
- TLS Enabled
- Authentication Enabled

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Data Sources:

- Windows Security Logs
- Windows System Logs
- Windows Application Logs
- Sysmon Operational Logs (Event ID 1, 3, 11)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Docker:

The lab environment was deployed using Docker containers on an Ubuntu virtual machine running on VirtualBox.

Due to limited internet connectivity during deployment, Docker images were downloaded manually and containers were configured and started using Docker CLI commands.

- **Pull Images**
```bash
	docker pull elasticsearch:8.13.0
	docker pull kibana:8.13.0
```
- Elasticsearch Container

```bash
	docker run -d \
	--name elasticsearch_8 \
	--network Elastic \
	-p 9200:9200 \
	-p 9300:9300 \
	-e discovery.type=single-node \
	-e xpack.security.enabled=true \
	elasticsearch:8.13.0
```

- **Kibana Container**
```bash
	docker run -d \
	--name kibana_8 \
	--network Elastic \
	-p 5601:5601 \
	-e ELASTICSEARCH_HOSTS='["https://IP_ADDRESS:9200"]' \
	-e ELASTICSEARCH_SSL_VERIFICATIONMODE=none \
	-e ELASTICSEARCH_SERVICEACCOUNTTOKEN="YOUR_TOKEN" \
	-e XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY="YOUR_KEY" \
	-e XPACK_SECURITY_ENCRYPTIONKEY="YOUR_KEY" \
	-e XPACK_REPORTING_ENCRYPTIONKEY="YOUR_KEY" \
	kibana:8.13.0
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Sysmon Installation & Configuration

Sysmon (System Monitor) is used to collect detailed endpoint telemetry beyond standard Windows logs. It provides visibility into process creation, network connections, and file activity.

- **Installation File**

	Sysmon is installed from Microsoft Sysinternals.

	Download link:

	https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon


	Run the following command as Administrator:
```bash
	Sysmon64.exe -accepteula -i sysmonconfig.xml
```
This command Installs Sysmon as a Windows service, loads the configuration file and Starts event logging.

sysmonconfig.xml is available in : Sysmon/sysmonconfig.xml

To verify Sysmon is installed correctly:
```bash
	Sysmon64.exe -c
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Filebeat 

Filebeat was deployed on Windows endpoints to collect and forward Windows Event Logs to the Elastic Stack.

-  **Installation**

	Filebeat was downloaded manually from the official Elastic distribution package and installed on Windows hosts.

- **Configuration**

	A custom `filebeat.yml` configuration file was used to enable Windows Event Log collection and establish a secure connection with Elasticsearch.

The configuration file is available in: Beats/Filebeat/filebeat.yml

- **Data Collection**

	The following Windows Event Logs were collected:
	- Security
	- System
	- Application

- **Elasticsearch Connectivity**

Filebeat was configured to send events directly to Elasticsearch running in the lab environment.

Connectivity was validated using powershell commands: 

```bash
.\filebeat.exe test config
.\filebeat.exe test output
```

- **Verification**
  
	Successful ingestion was verified through:
	- Elasticsearch indices
	- Kibana Discover


- **Data Flow**
```bash
Windows Endpoints
      |
      |
      |-- Windows Event Logs (Security, System, Application)
      |
      v
   Filebeat
      |
      v
Elasticsearch
      |
      v
Kibana Dashboards
```

-----------------------------------------------------------------------------------------------------------------------------------------------
# Winlogbeat

Winlogbeat was deployed on Windows endpoints to collect and forward Sysmon-generated events to the Elastic Stack, providing detailed visibility into process creation, network connections, file activity, and other security-relevant endpoint telemetry.

- **Installation**

	Winlogbeat was downloaded from the official Elastic distribution package and installed on Windows hosts designated for log collection.

- **Configuration**

	A custom winlogbeat.yml configuration file was used to define the Windows Event Log sources and establish a connection with Elasticsearch.

	The configuration file is available in: Beats/Winlogbeat/winlogbeat.yml

- **Data Collection**

	in the second Phase of Project, Winlogbeat was configured to collect events from the following Windows Event Log channels:

	- Microsoft-Windows-Sysmon/Operational
	- Microsoft-Windows-PowerShell/Operational
	- Security
	- System
	- Application

	Sysmon was deployed on Windows endpoints to provide enhanced telemetry, including:

	- Process Creation Events
	- Network Connection Events
	- File Creation Events

Winlogbeat collected Sysmon-generated events directly from the Microsoft-Windows-Sysmon/Operational channel and forwarded them to Elasticsearch.

- **Elasticsearch Connectivity**

	Winlogbeat was configured to send events directly to Elasticsearch running in the lab environment.

	Connectivity was validated using the following commands:

```bash
.\winlogbeat.exe test config
.\winlogbeat.exe test output
```

- **Verification**

	Successful log ingestion was verified through:

	- Elasticsearch indices
	- Kibana Discover

```bash
	Windows Endpoints
			|
			|
			|-- Sysmon Telemetry
			|
			v
		Winlogbeat
			|
			v
	Elasticsearch
			|
			v
	Kibana Dashboards
```






