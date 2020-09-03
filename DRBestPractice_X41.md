# Introduction
This documentation describes recommended parameters for a disaster recovery (DR) configuration.

# Target version for DR Best Practice
- Windows  
	EXPRESSCLUSTER X 12.10 or later	
- Linux  
	EXPRESSCLUSTER X 4.1.0-1 or later

# Cluster configuration for DR
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
			- Case2) 4 nodes Hybrid Disk cluster
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
		- Windwos) 90 sec
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
	- Goto [NP Resoution] tab
	- Add NP Reolution:
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

# Other settings
1. OS startup time  
OS startup time should be longer than Heartbeat timeout to detect Heartbeat timeout when the OS is rebooted.  
  e.g.) OS startup time 100sec > Heatbeat timeput 90sec
    - Change OS startup time with following way:
    	- Windows  
    		- bcdedit  
      **Reference**  
      [EXPRESSCLUSTER X 4.1 for Windows Installation and Configuration Guide](https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html)  
      \- Chapter 1 Determining a system configuration  
      \- Adjustment of the operating system startup time (Required)
    		- armdelay command  
        **Reference**  
       [EXPRESSCLUSTER X 4.1 for Windows Legacy feature guide](https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html)  
      		Chapter 3 Compatible command reference  
      		 \- Setting or displaying the start delay time (armdelay command)    
    	- Linux
    		- Adjust timeout of boot loader (GRUB).  
      **Reference**  
      [EXPRESSCLUSTER X 4.1 for Linux Installation and Configuration Guide](https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html)  
      \- Chapter 1 Determining a system configuration  
       \- Adjustment of the operating system startup time (Required)

1. Firewall  
TCP, UDP and ICMP ports which are used by EXPRESSCLUSTER shuold be opend.  
**Note**  
Windows will change the profile depends on the state of the network connection.  
**Reference**
    - [EXPRESSCLUSTER X 4.1 for Windows/Linux Getting Started Guide](https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html)  
  		\- Chapter 5 Notes and Restrictions  
  		\- Communication port number
    - [A batch file to open cluster ports](https://github.com/EXPRESSCLUSTER/Tools/blob/master/OpenPorts.bat)
