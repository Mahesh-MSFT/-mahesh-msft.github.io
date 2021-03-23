#

Oracle databases and applications run some of the mission critical workloads for customers. Microsoft Azure provides a comprehensive platform to successfully run Oracle workloads. Customers benefit from seamless support experience from Microsoft and Oracle when they need it. Customers can not only run Oracle workloads with existing skills and tools but also modernize them by using Artificial Intelligence (AI) and Machine Learning (ML) capabilities available in Azure.

Running such mission-critical applications on Azure entails following best practices and proven technical guidance. Microsoft Azure Well-Architected Framework (WAF) is a collection of assessment, actionable recommendations and deep technical guidance. Using Azure Well-Architected Framework, mission-critical workloads can be confidently developed and deployed on Azure. WAF helps improving the architecture quality of Azure workloads across five keys tenets. These five tenets are as below.

1. Cost Optimization
2. Operational Excellence
3. Performance efficiency
4. Reliability
5. Security

This article discusses developing and deploying Oracle workloads on Azure using Azure Well-Architected guidelines.

## Oracle application components

This article considers following application components to be deployed on Azure.

* JBoss EAP
* Oracle Database
* Oracle Data Guard

There are multiple deployment options available as listed below.

1. Single-region deployment
1. Multi-region deployment
1. Multi-region deployment (FAR SYNC)

Each deployment option is discussed in detail below.

## Single region deployment

Diagram below depicts deployment in a single Azure region.

Key highlights of this architecture are as discussed below.

1. This deployment uses a single Azure region.
1. It is suitable workloads with balanced costs and reliability.

## Multi region deployment

Diagram below depicts deployment across multiple Azure region.

Key highlights of this architecture are as discussed below.

1. This deployment uses multiple Azure regions.
1. It is suitable for workloads demanding very high reliability.

## Multi-region deployment (FAR SYNC)

Diagram below depicts deployment across multiple Azure region using Oracle Data Gurad FARSYNC synchronization.

Key highlights of this architecture are as discussed below.

1. This deployment uses Oracle Active Data Guard Far Sync instance and standby server in secondary Azure region.
1. It is suitable for workloads demanding very high RPO and RTO.

## Well-Architected Framework guidelines

Deployment options discussed earlier can be evaluated using Well-Architected Framework guidelines. Based on evaluation, customers can adjust Azure service configuration or deployment to improve overall architecture quality.

Each of the five tenets of well-Architected Framework is discussed in detail below.

### Reliability

Reliability of Oracle workload refers to it's ability to continue to operate after transient or prolonged failures. Workload is expected to remain available to it's end users when they need it.

Following are key guidelines to ensure Oracle workloads are reliable.

1. Azure Availability Zones inside Azure Region are used so that single zone failures can be tolerated.
2. User requests are routed through an Azure Load Balancer which can be configured in zone-redundant configuration to sustain a zone failure inside Azure region.
3. JBoss server runs on Azure VM Scaleset (VMSS) which uses multiple VM instances. Azure VMSS is also configured in zone-redundant configuration to sustain zone failure inside Azure region.
4. Oracle Active Data Guard will provide database reliability in case of a single Azure Availability Zone failure.
5. In multi-region deployment, Azure Paired region will ensure that planned and unplanned maintenance activities are not performed at the same time improving reliability of the platform.
6. In multi-region deployment, Azure Front Door will provide a region redundant service in place of Azure Application Gateway.
7. In multi-region deployment, Oracle Standby instance running in a separate region will provide protection against primary Azure region failure.
8. If used, FAR SYNC instance will ensure zero data loss and coordinate with standby server to promote a primary database server.
9. Primary Oracle Database and FARSYNC instances, if used,  are deployed in separate availability zone providing protection against regional failure.

### Security

1. Azure Virtual Network with multiple subnets will be used to deploy the solution. Each subnet will be configured with Network Security Group (NSG) rules to control east-west traffic.
2. Azure DdoS protection will be applied on Azure VNet mitigating DdoS attack risk.
3. Azure Network Watcher is used to monitor, diagnose, view metrics, and enable or disable logs for resources in Azure VNet.
4. Azure Active Directory (AD) will be used with Azure AD Role Based Access Control (RBAC) to limit access to application components.
5. Azure Key Vault will be used to store sensitive information such as passwords, connectionstrings, etc.
6. Azure Security Center protects server and application vulnerabilities on them. It helps to quickly identify threats, streamline threat investigation, and help automate remediation.
7. Azure Bastion will be used to connect with Application and Database VMs.
8. Azure AD Privileged Identity Management (PIM) solution will provide just-in-time privileged access to Azure resources and Azure AD. It will also provide audit trail of what admin users are doing with their administrator privileges.
9. Azure Firewall will provide capability to control and monitor outbound internet traffic from VMs and other solution components.
10. Azure Disk Encryption for Managed Disk will provide encryption at rest.

### Performance efficiency

1. Azure Managed Disks with premium SSD supports upto 20,000 IOPS and 900 MB/s of throughput.
2. Azure VMs with Accelerated Networking can provide up to 30Gbps of network throughput.
3. Azure VMSS running Web server VM instances are placed in Proximity Placement Group which co-locates VM instances and reduced inter-VM latency.
4. Estimate Oracle VM size based on CPU, memory, and I/O usage from the AWR report.
5. Configure Azure Disk with host caching as ReadOnly for higher throughput.
6. Azure Front Door provides lower network latency to the web application.
7. Azure Ultra Disk provides upto 160K IOPS, 2K Mbps throughput at sub-millisecond disk latency.
8. Oracle primary and FAR SYNC instances are in same Azure region providing lower network latency times.


### Operational excellence

1. Azure Managed Application will help SWIFT to implement the infrastructure and provide ongoing support.
2. Azure Policy provides building block to codify compliance requirements through definition and assignment.
3. Azure Monitor enables customers with collecting, analyzing, and acting on telemetry from infrastructure as well as application.
4. Azure ARM template will provide infrastructure-as-code services offering automated deployment and operations.
5. Azure Automation Update Management will be used to patching VMs running Oracle database.
6. Azure Storage (Large File Share) will be used to backup Oracle database.
7. Azure Devops based CI/CD will provide zero-touch deployment experience improving overall operational management for the solution.
8. In multi-region deployment, Azure Backup can be used instead of Azure Storage based backup solution for Oracle database.
9. Azure Advisor provides additional recommendations for improving performance, availability, security, and other key metrics.

### Cost optimization

1. Azure Cost Management & Billing provides governance for effective cost management. It increases accountability with budgets, cost allocation, chargebacks & continuous cost optimization.
2. Azure Reservation will provide substantial discounts on Azure VMs and Storage.
3. Extend Azure Hybrid Usage benefits to Red Hat Enterprise Linux VMs.
