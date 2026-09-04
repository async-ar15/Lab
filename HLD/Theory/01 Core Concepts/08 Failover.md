
What is Failover? Definition & Meaning
Failover is the ability to switch automatically and seamlessly to a reliable backup system. When a component or primary system fails, either a standby operational mode or redundancy should achieve failover and lessen or eliminate negative impact on users.

To achieve redundancy upon the abnormal failure or termination of a formerly active version, a standby database, system, server, or other hardware component or network must always stand ready to automatically switch into action. In other words, all backup techniques including standby computer server systems must themselves be immune to failure, because failover is critical to disaster recovery (DR).

Failover automation in servers includes pulse or heartbeat conditions. That is, heartbeat cables connect two servers or multiple servers in a network with the primary server always active. As long as the heartbeat continues or it perceives the pulse, the secondary server merely rests.

However, should the secondary server perceive any change in the pulse from the primary failover server, it will initiate its instances and take over the primary server’s operations. It will also message the technician or data center requesting that they bring the primary server back online. Some systems, called automated with manual approval configuration, simply alert the technician or data center instead, requesting the change to the server take place manually.

Virtualization simulates a computer environment using a virtual machine or pseudo machine running host software. In this way, the failover process can be independent of the physical hardware components of computer server systems.

 

How does failover work?
Active-active and active-passive or active-standby are the most common configurations for high availability (HA). Each implementation technique achieves failover in a different way, although both improve reliability.

Typically, at least two nodes actively and simultaneously running the same sort of service comprise an active-active high availability cluster. The active-active cluster distributes workloads across all the nodes more evenly, preventing any single node from overloading and achieving load balancing. And because more nodes remain available, throughput and response times improve. To ensure the HA cluster operates seamlessly and achieves redundancy, the individual configurations and settings of the nodes should be identical.

In contrast, in an active-passive cluster, although there must be at least two nodes, not all of them are active. In a two node system with the first node active, the second node will remain passive or on standby as the failover server. In this standby operational mode, it can remain ready should the active, primary server stop functioning to serve as a backup. However, unless there is a failure, clients only connect to the active server.

Just as in the active-active cluster, both servers in the active-standby cluster must be configured with the very same settings. This way, clients cannot perceive any change in service, even if the failover router or server must take over.

Clearly, in an active-standby cluster although the standby node is always running, actual utilization approaches zero.

In an active-active cluster, utilization of both nodes nears half and half— although each node can handle the entire load alone. However, this also means that node failure can cause performance to degrade if one active-active configuration node handles more than half of the load consistently.

Outage time during a failure is virtually zero with an active-active HA configuration, because both paths are active. With an active-passive configuration, outage time has the potential to be greater, as the system must switch from one node to the other, which requires time.

 

Types of Failover Configurations: Active-Active vs Active-Passive
Failover configurations ensure continuous service availability by switching operations to backup systems during failures. The two most common types are Active-Active and Active-Passive. Here’s how they differ:

Active-Active Configuration
Multiple servers (nodes) run the same service simultaneously.

All nodes are active and share the workload evenly.

Benefits:

Load balancing: Distributes workloads across all nodes, preventing overload on any single server.

High availability: If one node fails, others continue operating without interruption.

Improved performance: Handles requests faster with higher throughput.

Requirements:

Servers must have identical configurations for consistency.

Considerations:

If a node fails, remaining nodes handle increased load until recovery.

Active-Passive Configuration
One server (active node) handles all operations; another server (passive node) stays on standby.

The passive node is synchronized and ready to take over if the active server fails.

Benefits:

Standby backup: Passive server is ready but not handling traffic under normal conditions.

Simpler failover: Switches to passive node when failure occurs.

Lower resource use: Passive server remains mostly idle, saving costs.

Considerations:

Failover can involve a short delay while the passive server takes control.

Overall throughput is lower since only one server is active at a time.

 

What is a failover cluster?
A failover cluster is a set of computer servers that provide fault tolerance (FT), continuous availability (CA), or high availability (HA) together. Failover cluster network configurations may use virtual machines (VMs), physical hardware only, or both.

If one of the servers in a failover cluster goes down, this triggers the failover process. Instantly sending the failed component’s workload to another node in the cluster, this prevents downtime.

Providing either HA or CA for applications and services is a failover cluster’s primary goal. Also known as fault tolerant (FT) clusters, CA clusters eliminate downtime when main or primary systems fail, enabling end users to keep using applications and services without interruptions or timeouts.

In contrast, despite a potential brief interruption in service, HA clusters offer minimal downtime, automatic recovery, and no data loss. The recovery process in HA clusters can be configured using failover cluster manager tools, which are included as part of most failover cluster solutions.

In a broader sense, a cluster is two or more nodes or servers, usually connected both physically with cables and via software. Additional clustering technologies such as parallel or concurrent processing, load balancing, and cloud storage solutions are included in some failover implementations.

Internet failover is essentially a redundant or secondary internet connection to be used as a failover link in case of a failure. This can be thought of as another piece of failover capability in servers.

 

What is an application server failover?
Application servers are simply servers that run applications. This means that application server failover is a failover strategy to protect these types of servers.

At a minimum, these application servers should have unique domain names, and ideally they should run on different servers. Failover cluster best practices typically include application server load balancing.

 

Failover Testing: Importance & How to Perform It
Failover testing validates a system’s capacity during a server failure to allocate sufficient resources toward recovery. In other words, failover testing assesses failover capability in servers.

The test will determine whether the system has the capacity in the event of any kind of abnormal termination or failure to handle necessary extra resources and move operations to backup systems. 

For instance, failover and recovery testing determines the ability of the system to manage and power an additional CPU or multiple servers once it achieves a threshold for performance — one often breached during critical failures. This highlights the important relationship between failover testing, cyber resilience, and security.

 

Failover vs Failback Explained
In computing and related technologies such as networking, failover is the process of switching operations to a backup recovery facility. The backup site in failover is generally a standby or redundant computer network, hardware component, system, or server, often in a secondary disaster recovery (DR) location. Typically, failover involves using a failover tool or failover service of some type to temporarily halt and restart operations from a remote location.

A failback operation involves returning production to its original location after a scheduled maintenance period or a disaster. It is the return from standby to fully functional.

Typically, systems designers offer failover capability in systems, servers, or networks demanding CA, HA, or a high level of reliability. Failover practices have also become less reliant on physical hardware with little or no disruption in service thanks to the use of virtualization software.

 

Does Druva offer a cloud failover strategy?
Druva offers a comprehensive cloud failover strategy designed to simplify disaster recovery for both on-premises and cloud workloads. With integrated cloud backups, Druva removes the stress and complexity of maintaining business continuity.

A simple, identical configuration of your primary and failover VMs remains the first step. Data transfer begins as soon as virtual machine disks are attached, and once transfer is complete, DNS connections are redirected, and primary VMs are rebooted with a single click failback to the primary site, ensuring fast post-event mitigation.

As threats from remote worker data and cyber attackers continue to increase, protecting your enterprise data is more critical than ever. Druva leverages the global reach and scalability of AWS to deliver a robust disaster recovery as a service (DRaaS) solution that keeps your data always on and always safe.

Watch the video below to learn more about the meaning of failover, and explore Druva DRaaS here to find out how the cloud ensures your data is always on, always safe.

Learn how you can build your DR plan: Read Whitepaper!

 

FAQs
What is failover and why is it important?
Failover is the ability to automatically switch to a backup system when the primary system fails. It ensures continuous availability and minimizes downtime, which is crucial for disaster recovery and maintaining uninterrupted service.

How does failover work in different configurations?
Failover can be implemented in active-active or active-passive clusters. Active-active clusters run multiple servers simultaneously, balancing the load, while active-passive clusters keep one server active and the other on standby, ready to take over if the active one fails.

What is a failover cluster?
A failover cluster is a group of servers connected to provide high availability or fault tolerance. If one server fails, the workload automatically shifts to another server in the cluster, preventing downtime and ensuring continuous operation.

What is failover testing and why is it necessary?
Failover testing checks if a system can successfully transfer operations to backup resources during a failure. It validates that the system has enough capacity to handle additional loads and recover automatically without data loss or significant downtime.

What is the difference between failover and failback?
Failover is switching operations to a backup system during failure, while failback is returning operations to the original system after maintenance or recovery. Failback restores normal operations once the primary system is stable and fully functional again.

