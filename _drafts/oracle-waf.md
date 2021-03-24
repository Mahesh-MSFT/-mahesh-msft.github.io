#

Oracle databases and applications run some of the mission critical workloads for customers. Microsoft Azure provides a comprehensive platform to successfully run Oracle workloads. Customers benefit from seamless support experience from Microsoft and Oracle when they need it. Customers can not only run Oracle workloads with existing skills and tools but also modernize them by using Artificial Intelligence (AI) and Machine Learning (ML) capabilities available in Azure.

Running such mission-critical applications on Azure entails following best practices and proven technical guidance. Microsoft Azure Well-Architected Framework (WAF) is a collection of assessment, actionable recommendations and deep technical guidance. Using Azure Well-Architected Framework, mission-critical workloads can be confidently developed and deployed on Azure. WAF helps improving the architecture quality of Azure workloads across five keys tenets. These five tenets are as below.

1. Reliability
1. Security
1. Performance efficiency
1. Operational Excellence
1. Cost Optimization

This article discusses developing and deploying Oracle workloads on Azure using Azure Well-Architected guidelines.

## Oracle application components

This article considers following application components to be deployed on Azure.

* JBoss EAP
* Oracle Database
* Oracle Data Guard

There are multiple deployment options available as listed below.

1. Single-region deployment
1. Multi-region deployment
1. Multi-region deployment (Far Sync)

Each deployment option is discussed in detail below.

## Single-region deployment

Diagram below depicts deployment in a single Azure region.

![single-region deployment](/assets/oraclewaf/1Region.png)

Key highlights of this architecture are as discussed below.

1. This deployment uses a single Azure region.
1. It is suitable workloads with balanced costs and reliability.

## Multi-region deployment

Diagram below depicts deployment across multiple Azure region.

![multi-region deployment](/assets/oraclewaf/2Region.png)

Key highlights of this architecture are as discussed below.

1. This deployment uses multiple Azure regions.
1. It is suitable for workloads demanding very high reliability.

## Multi-region deployment (Far Sync)

Diagram below depicts deployment across multiple Azure region using Oracle Data Guard Far Sync synchronization.

![multi-region far sync deployment](/assets/oraclewaf/2RegionFarSync.png)

Key highlights of this architecture are as discussed below.

1. This deployment uses Oracle Active Data Guard Far Sync instance and standby server in secondary Azure region.
1. It is suitable for workloads demanding very high RPO and RTO.

## Well-Architected Framework guidelines

Deployment options discussed earlier can be evaluated using Well-Architected Framework guidelines. Based on evaluation, customers can adjust Azure service configuration or deployment to improve overall architecture quality.

Each of the five tenets of well-Architected Framework is discussed in detail below.

### Reliability

Reliability of an Oracle workload refers to it's ability to continue to operate after transient or prolonged failures. Workload is expected to remain available to it's end users when they need it.

Following are key guidelines to ensure Oracle workloads are reliable.

1. A power outage or cooling failure in an Azure data center can bring down a Virtual Machine (VM) running any Oracle workload component. **Azure Availability Zones** inside an Azure Region use independent power, cooling, connectivity, etc. for data centers in them. By using Azure Services that support Availability Zones, Oracle workloads can continue to operate even if there are outages in a single data center or Availability Zone.
2. Using a single public IP address assigned to a Azure VM running web front-end can result in failed user requests if VM becomes unavailable. Azure Load Balancer can route incoming user requests on it's public IP address to multiple VMs acting as backend. By using ***zone-redundant* Azure Load Balancer**, user requests can be accepted not only when an Azure Availability Zone becomes unavailable but also when any of the multiple backend VMs behind load balancer becomes unavailable.
3. A single instance on Azure VM running an Oracle application component can easily become a single point of failure or a bottleneck. Azure Virtual Machine Scale Set (VMSS) combines multiple VM instances for easier scaling as well as higher availability. BY running JBoss server on ***zone-redundant* Azure VMSS**, Oracle workloads can not only sustain zone failures inside Azure region but also scale to cater to higher user requests.
4. Running an Oracle database on a single VM can result in data loss when that VM becomes unavailable. **Oracle Active Data Guard** runs primary and standby database instances to provide database reliability. By running primary and standby database instances across 2 Availability Zones, Oracle workloads are protected from single Azure Availability Zone failure.
5. During catastrophic natural disasters, entire Azure region may become unavailable. Also, if not carefully planned, Azure updates may run simultaneously in two Azure regions together impacting Oracle workload. **Azure Paired Region** will ensure that planned and unplanned maintenance activities are not performed at the same time improving reliability. At least one of the two paired regions will be prioritized for recovery in case of catastrophic disaster.
6. Running standby Oracle database instance in Availability Zone may not be enough in scenarios where Oracle workloads demand higher reliability. Using Oracle Active Data Guard, it's possible to run ***standby* instance in Azure Paired Region**. This regional isolation will provide protection against primary Azure region failure.
7. A regional Azure service becomes unavailable if the region in which it is deployed becomes unavailable. Global Azure services ensure that they automatically fail-over to nearest feasible Azure region. In multi-region deployment, **Azure Front Door** will provide a region redundant service to improve reliability of mission-critical Oracle workloads.
8. Some customers may choose superior performance from Oracle database over some amount of data loss due to synchronous replication between primary and standby database servers. **Oracle Active Data Guard Far Sync** makes it possible to have both, superior performance and high reliability together. Far Sync instance will coordinate with standby server(s) to promote a new primary database in case of existing primary database failure.
9. Running Primary Oracle Database and Far Sync instance in the same Availability Zone may still result in workload unavailability in case of zone failures. It is possible to deploy primary database and Far Sync instance **across Availability Zones**. Deploying in separate Availability Zone provides protection against zonal failure.

### Security

Security spans multiple areas such as infrastructure, application, data, identity, etc. of an Oracle workload. Guidelines provided below provide holistic approach to infuse security protection and detection capabilities into Oracle workload.

1. An Azure Virtual Network (VNet) can be segmented into multiple Subnets. By default, there is no network access control between these subnets. Lack of network access control can result in unsolicited network traffic arriving inside a subnet. Azure **Network Security Group (NSG)** helps is filtering incoming traffic to and from a subnet. NSGs can allow or deny network traffic based on stateful packet inspection. Any resources inside subnet can receive traffic from only allowed IP address range(s). Configuring NSGs helps in controlling East-West traffic between Oracle application components.
2. Any publically reachable Azure resource is exposed to threat of Distributed Denial of Service (DDoS) attack. A successful DDoS attack can impact the Oracle workload availability to it's intended users. A prolonged DDoS attack can exhaust all available resources and result in downtime for mission-critical Oracle application(s). **Azure DDoS Protection** service defends Azure resources against DDoS attacks. Azure DDoS Protection continuously monitors incoming traffic to identify potential indications of a DDoS attack. Oracle workloads benefit from working with Microsoft's DDoS Rapid Response (DRR) team during an active attack. Azure DdoS protection will be applied on Azure VNet that hosts all Oracle application components mitigating DdoS attack risk.
3. Without proper Azure VNet network monitoring, Oracle workloads are exposed to the risk of undesired or unknown traffic coming to Azure VNet from compromised IP addresses. **Azure Network Watcher** provides a way to monitor and if necessary repair any network issue related to Oracle application component VMs. Azure Network Watcher is used to monitor, diagnose, view metrics, and enable or disable logs for Oracle application components inside Azure VNet.
4. Uncontrolled access to Oracle application components and its underlying infrastructure can result in unexpected results. **Azure Active Directory (AAD)** helps in providing controlled access to users, groups and other security principals. Azure Active Directory (AD) will be used with Azure AD Role Based Access Control (RBAC) to limit access to Oracle application components and it's underlying Azure services.
5. Oracle application web front end uses Oracle connectionstring to connect with Oracle database. In many scenarios, this connectionstring contains sensitive details like user name and password. Malicious actors can use these credentials to potentially access database offline. **Azure Key Vault** provides a secure way to store and access Oracle credentials. Oracle web front end can securely access credentials stored in Azure Key Vault for establishing connection with Oracle database.
6. An Oracle workload on Azure contains multiple resources (VMs, Disks, etc.). These resources are exposed to risks such as malware/unwanted software installation, uncontrolled access to management ports on a VM, etc. With sophisticated security attacks, detecting security vulnerabilities and protecting Oracle workload is extremely challenging. **Azure Security Center** is Azure's native security management system which assesses Azure resources for their security posture against security best practices. Azure security center helps to detect and prevent threats against data and application services. By using Azure Security Center, Oracle workloads benefit from continuous security assessment and actionable recommendations should there be any deviation from security best practice.
7. Very often Oracle administrators will need to initiate RDP or SSH sessions on VMs. This need sometimes can result in assigning public IP addresses to VMs which is a security issue. **Azure Bastion** is an Azure service that provides private RDP and SSH access using VM's private IP address. By using Azure Bastion, Oracle administrators can safely and securely connect with Application and Database VMs.
8. While Azure AD RBAC helps in controlling access to Oracle workload infrastructure, there is still a possibility of users abusing their higher administrative rights. **Azure AD Privileged Identity Management (PIM)** helps in addressing such scenarios by providing just-in-time access and many more controls to limit and audit actions by higher privilege users. Azure AD PIM will provide just-in-time privileged access to Azure resources running Oracle workloads. It will also provide audit trail of what admin users are doing with their administrator privileges.
9. While Azure NSGs controls network layer traffic, Oracle workloads may still be exposed to unwanted application level traffic. **Azure Firewall** provides control over network and application traffic for both ingress as well as egress traffic. By using Azure Firewall, Oracle workload gains capability to control and monitor outbound internet traffic from VMs and other solution components.
10. Oracle database uses Azure Disks which if not encrypted can result in data breach. **Azure Disk Encryption** ensures that data on disks is encrypted. Oracle database can securely use Azure Disk Encryption for Managed Disk to provide encryption at rest.

### Performance efficiency

Oracle runs many most mission-critical workloads for customers. These applications are extremely performance sensitive. Guidelines discussed below enable Oracle workloads to meet performance expectations.

1. Poor disk performance has direct impact on Oracle database. User experience is adversely affected with sluggish database performance.**Azure Managed Disk** offers HDD or SSD backed storage solution to be appropriately used based on Oracle database performance requirements. Azure Managed Disks with *premium SSD* supports upto 20,000 IOPS and 900 MB/s of throughput which can be optimum performance needed for most Oracle databases.
2. Network throughput of VMs running  various Oracle application components can be an important factor driving user experience. Azure VMs with **Accelerated Networking** can provide up to 30Gbps of network throughput. 
3. Azure VMSS running Web server VM instances are placed in Proximity Placement Group which co-locates VM instances and reduced inter-VM latency.
4. Estimate Oracle VM size based on CPU, memory, and I/O usage from the AWR report.
5. Configure Azure Disk with host caching as ReadOnly for higher throughput.
6. Azure Front Door provides lower network latency to the web application.
7. Azure Ultra Disk provides upto 160K IOPS, 2K Mbps throughput at sub-millisecond disk latency.
8. Oracle primary and FAR SYNC instances are in same Azure region providing lower network latency times.

### Operational excellence

Oracle workloads need some of the most streamlined operational processes. Such processes help keeping workloads running even when operational and management operations are performed. Guidelines discussed below help in ensuring operational excellence.

1. Azure Managed Application will help to implement the infrastructure and provide ongoing support.
2. Azure Policy provides building block to codify compliance requirements through definition and assignment.
3. Azure Monitor enables customers with collecting, analyzing, and acting on telemetry from infrastructure as well as application.
4. Azure ARM template will provide infrastructure-as-code services offering automated deployment and operations.
5. Azure Automation Update Management will be used to patching VMs running Oracle database.
6. Azure Storage (Large File Share) will be used to backup Oracle database.
7. Azure Devops based CI/CD will provide zero-touch deployment experience improving overall operational management for the solution.
8. In multi-region deployment, Azure Backup can be used instead of Azure Storage based backup solution for Oracle database.
9. Azure Advisor provides additional recommendations for improving performance, availability, security, and other key metrics.

### Cost optimization

Oracle workloads can take advantage of multiple cost optimization guidelines discussed below. Customers can achieve balance between cost and quality of Oracle workloads.

1. Scale down VMSS
1. Azure Cost Management & Billing provides governance for effective cost management. It increases accountability with budgets, cost allocation, chargebacks & continuous cost optimization.
1. Azure Reservation will provide substantial discounts on Azure VMs and Storage.
1. Extend Azure Hybrid Usage benefits to Red Hat Enterprise Linux VMs.
