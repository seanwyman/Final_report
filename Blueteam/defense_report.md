# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
The following machines were identified on the network:

  | HOSTNAME       |     IP        |  OS          |           PURPOSE        |
  |----------------|---------------|--------------|--------------------------|
  | HOST           | 192.168.1.1   |  WINDOWS     | host the lab environment |
  | CAPSTONE       | 192.168.1.105 | UBUNTU       |                          |
  | KALI (root)    | 192.168.1.90  | KALI LINUX   | PenTesting Environmnet   |
  | ELK            | 1291.68.1.100 | LINUX        |      ELK STACK SIEM      |
  | TARGET1        | 192.168.1.110 | LINUX 3.2-4.9|     WORDPRESS WEBSITE    |
  | TARGET 2       | 192.168.1.115 | LINUX 3.2-4.9|     WORDPRESS WEBSITE    |


### Description of Targets

The target of this attack was: `Target 1: 192.168.1.110`

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. So we have implemented the alerts below:

#### Excessive HTTP Errors
  - **Beat**: packetbeat-*
  - **Metric**: WHEN count() GROUPED OVER top 5 'http.response.status_code'
  - **Threshold**:  IS ABOVE 400 FOR THE LAST 5 minutes
  - **Vulnerability Mitigated**: Flooding
  - **Reliability**: High, the only times this alert ever pinged was on the day of the attack.

#### HTTP Request Size Monitor
  - **Beat**: packetbeat-*
  - **Metric**:  WHEN sum() of http.request.bytes OVER all documents
  - **Threshold**: IS ABOVE 3500 FOR THE LAST 1 minute
  - **Vulnerability Mitigated**: Flooding
  - **Reliability**: Very low reliablity, just sitting on the host machine and looking at Kibana generates this event to go off over and over.

#### CPU Usage Monitor
  - **Beat**: metricbeat-*
  - **Metric**: WHEN max() OF system.process.cpu.total.pct OVER all documents
  - **Threshold**:  IS ABOVE 0.5 FOR THE LAST 5 minutes
  - **Vulnerabilites Mitigated**:  Botnets and Cryptojacking attacks
  - **Reliability**: This alert generated some false positive
