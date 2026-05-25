# Disaster Recovery best practices for EXPRESSCLUSTER X 5

This documentation describes recommended parameters for a disaster recovery (DR) configuration.

> **Note:** This document has been updated from the X 4.1 best practice guide to reflect EXPRESSCLUSTER X 5.3, released April 8, 2025. Please refer to the [official EXPRESSCLUSTER documentation](https://www.nec.com/en/global/prod/expresscluster/en/doc/manual.html) for the complete manuals.

## Target version for DR Best Practice

- EXPRESSCLUSTER X 5.3 for Windows (or later)
- EXPRESSCLUSTER X 5.3 for Linux (or later)

## Cluster configuration for DR

Start Cluster WebUI [Config Mode], edit cluster configuration for DR cluster as below and apply the configuration.

1. Failover Attribute  
Set server groups at each site and manual failover between server groups.
	1. Creating Server Groups  
		If you have already created server groups for hybrid disk resource, please skip to next step.
		- Click Servers [Properties]
		- Goto [Server group] tab
		- Add a server group for a primary site and add primary server(s) to the server group as follows
			- Case1) 2 nodes Mirror Disk cluster  
				| Server Group Name | Server Name |
				|:------------------|:------------|
				| site1	| server1 |
				| site2	| server2 |
			- Case2) 3 nodes Hybrid Disk cluster
				| Server Group Name | Server Name |
				|:------------------|:------------|
				| site1	| server1, server2 |
				| site2	| drserver1 |
			- Case3) 4 nodes Hybrid Disk cluster
				| Server Group Name | Server Name |
				|:------------------|:------------|
				| site1	| server1, server2 |
				| site2	| drserver1, drserver2 |
		- Add one more server group for a backup site and add backup server(s) to the server group (e.g. site2)
		- Click [OK]
	1. Changing Failover Group Parameters
		- Click failover group [Properties]
		- Goto [Info] tab and check [Use Server Groups Settings]
		- Goto [Startup Server] tab and add all server groups which you have created in the step above
		- Goto [Attribute] tab
		- Set the following parameters
			- [Manual Startup] : Check
			- [Auto Failover] : Check
			- [Prioritize failover policy in the server group] : Check
			- [Enable only manual failover among the server groups] : Check
		- Click [OK].

1. Heartbeat  
Extend Heartbeat timeout.
	- Click cluster [Properties].
	- Goto [Timeout] tab
	- [Heartbeat Timeout] : Set longer heartbeat timeout than default value.  
		**Example**
		- Windows) 90 sec
		- Linux) 120 sec

1. Mirror/Hybrid Disk Resource Parameters  
Edit Mirror/Hybrid Disk resource parameters.
	1. Mirror Connect Timeout **(For Windows only)**
		- Click md/hd resource [Properties]
		- Goto [Details] tab and click [Tuning]
		- Set [Mirror Connect Timeout]:  Heartbeat timeout – 10 seconds  
			(e.g. If heartbeat timeout is 90sec, mirror connect timeout should be 80sec.)
		- Click [OK].
	1. Mirroring Mode  
		- Click md/hd resource [Properties]
		- Goto [Details] tab and click [Tuning]
		- Goto [Mirror] tab
		- Set [Mode]:  
			Synchronous mode is recommended. But if I/O performance is slower than you expected, set asynchronous mode.
		- Click [OK].
	1. Data Compression
		- Click md/hd resource [Properties]
		- Goto [Details] tab and click [Tuning]
		- Goto [Mirror] tab
		- Set the following parameters:
			- [Compress Data] : Check (For asynchronous mode only)
			- [Compress Data When Recovering] : Check
		- Click [OK].
	1. History File **(For asynchronous mode only)**
		- Click md/hd resource [Properties]
		- Goto [Details] tab and click [Tuning]
		- Goto [Mirror] tab
		- Set the following parameters
			- [History Files Store Directory] : Set directory path for history file
			- [Limit size of History File] : Set maximum size of history file
				- **Note** We recommend to specify other than system drive (such as C: for Windows or /dev/sdaX for Linux) for History Files Store Directory or set Limit size of History File because the system may not work properly if system drive gets full.
		- Click [OK]
	1. Number of Queues **(For Linux asynchronous mode only)**
		- Click md/hd resource [Properties]
		- Goto [Details] tab and click [Tuning]
		- Goto [Mirror] tab
		- [Number of Queues] : 2048
	1. Mirror Driver **(For Linux only)**
		- Click cluster [Properties]
		- Goto [Mirror Driver] tab
		- Set the following parameters:
			- [Max. Number of Request Queues] : 2048
			- [Operation at I/O Error Detection] **(For multi-path Hybrid Disk only)**
				- [Cluster Partition] : RESET or PANIC
				- [Data Partition] : NONE

1. Mirror/Hybrid Disk Monitor Resource Parameters  
Edit Mirror/Hybrid Disk monitor resource parameter.
	1. Retry Count
		- Click mdw/hdw monitor resource [Properties]
		- Goto [Monitor(common)] tab
		- Retry Count : Set 1  

		**Note**  
		Please do NOT change Timeout parameter from 999sec (default).

1. Network Partition Resolution  
Set NP Resolution Resource.
	- Click cluster [Properties].
	- Goto [NP Resolution] tab
	- Add NP Resolution:
		- Case1: 2 nodes Mirror Disk cluster  
			No NP Resolution Resource are required because manual failover between server groups are set.
		- Case2: 3 nodes Hybrid Disk cluster
			- Windows  
				| Type | server1 | server2 | server3 |
				|:--------|:---------------|:---------------|:---------------|
				| Ping NP | pingnp target1 | pingnp target1 | - |
				| Disk NP | disknp target1 | disknp target1 | - |
			- Linux  
				| Type | server1 | server2 | server3 |
				|:--------|:---------------|:---------------|:---------------|
				| Ping NP | pingnp target1 | pingnp target1 | - |
		- Case3: 4 nodes Hybrid Disk cluster
			- Windows  
				| Type | server1 | server2 | server3 | server4 |
				|:--------|:---------------|:---------------|:---------------|:---------------|
				| Ping NP | pingnp target1 | pingnp target1 | pingnp target1 | pingnp target1 |
				| Disk NP | disknp target1 | disknp target1 | pingnp target1 | pingnp target1 |
			- Linux  
				| Type | server1 | server2 | server3 | server4 |
				|:--------|:---------------|:---------------|:---------------|:---------------|
				| Ping NP | pingnp target1 | pingnp target1 | pingnp target1 | pingnp target1 |

## Other settings

### 1. Service Startup Delay Time

When one of the servers reboots, other servers need to detect the Heartbeat timeout.
To ensure it, set up "Service Startup Delay Time".

**[Reference]**

- EXPRESSCLUSTER X 5.3 for **Windows** Installation and Configuration Guide - [2.6.3 Adjustment of time for EXPRESSCLUSTER services to start up (Required)](https://www.nec.com/en/global/prod/expresscluster/en/doc/manuals/W53_IG_EN_03.pdf#page=27)

- EXPRESSCLUSTER X 5.3 for **Linux** Installation and Configuration Guide - [2.8.5 Adjustment of time for EXPRESSCLUSTER services to start up (Required)](https://www.nec.com/en/global/prod/expresscluster/en/doc/manuals/L53_IG_EN_03.pdf#page=35)

### 2. Firewall  

TCP, UDP ports and ICMP which are used by EXPRESSCLUSTER should be opened.  

Use `clpfwctrl` command for opening the firewall ports used by EXPRESSCLUSTER.

**[Reference]**

- EXPRESSCLUSTER X 5.3 for **Windows** Reference Guide - [Adding a Firewall Rule (clpfwctrl command)](https://docs.nec.co.jp/software/clustering/expresscluster_x/x53/ecx_x53_windows_en/W53_RG_EN/W_RG_09.html#adding-a-firewall-rule-clpfwctrl-command)

- EXPRESSCLUSTER X 5.3 for **Linux** Reference Guide - [Adding a Firewall Rule (clpfwctrl command)](https://www.nec.com/en/global/prod/expresscluster/en/doc/manuals/W53_RG_EN_05.pdf#page=835)

## Version History

| Version | Date | Changes |
|:--------|:-----|:--------|
| X 5.3 update | May 2026 | Updated target versions to EXPRESSCLUSTER X 5.3 (released April 8, 2025). Updated all manual references to X 5.3 links. |
| X 4.1 original | - | Initial document for EXPRESSCLUSTER X 4.1 / X 12.10 |
