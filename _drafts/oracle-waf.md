blah. blah.

## Single region architecture

Diagram below depicts AMH architecture in a single Azure region.

Key characteristics of this architecture are discussed below.

### Reliability

1. Azure Availability Zones inside Azure Region are used so that single zone failures can be tolerated.
2. User requests are routed through an Azure Load Balancer which can be configured in zone-redundant configuration to sustain a zone failure inside Azure region. 
3. JBoss server runs on Azure VM Scaleset (VMSS) which uses multiple VM instances. Azure VMSS is also configured in zone-redundant configuration to sustain zone failure inside Azure region.
4. Oracle Active Data Guard will provide database reliability in case of a single Azure Availability Zone failure.

### Security

1. Azure Virtual Network with multiple subnets will be used to deploy the solution. Each subnet will be configured with Network Security Group (NSG) rules to control east-west traffic.
2. Azure DdoS protection will be applied on Azure VNet mitigating DdoS attack risk.
3. Azure Network Watcher is used to monitor, diagnose, view metrics, and enable or disable logs for resources in Azure VNet.
4. Azure Active Directory (AD) will be used with Azure AD Role Based Access Control (RBAC) to limit access to application components.
5. Azure Key Vault will be used to store sensitive information such as passwords, connectionstrings, etc.
6. Azure Security Center protects server and application vulnerabilities on them. It helps to quickly identify threats, streamline threat investigation, and help automate remediation.
7. Azure Bastion will be used to connect with Application and Database VMs.

### Performance efficiency

1. Azure Managed Disks with premium SSD supports upto 20,000 IOPS and 900 MB/s of throughput.
2. Azure VMs with Accelerated Networking can provide up to 30Gbps of network throughput.
3. Azure VMSS running Web server VM instances are placed in Proximity Placement Group which co-locates VM instances and reduced inter-VM latency.
4. Estimate Oracle VM size based on CPU, memory, and I/O usage from the AWR report.
5. Configure host caching as ReadOnly for higher throughput.

### Operational excellence

1. Azure Managed Application will help SWIFT to implement the infrastructure and provide ongoing support.
2. Azure Policy provides building block to codify compliance requirements through definition and assignment.
3. Azure Monitor enables customers with collecting, analyzing, and acting on telemetry from infrastructure as well as application.
4. Azure ARM template will provide infrastructure-as-code services offering automated deployment and operations.
5. Azure Automation Update Management will be used to patching VMs running Oracle database.
6. Azure Storage (Large File Share) will be used to backup Oracle database.

### Cost optimization

1. Azure Cost Management & Billing provides governance for effective cost management. It increases accountability with budgets, cost allocation, chargebacks & continuous cost optimization.
2. Azure Reservation will provide substantial discounts on Azure VMs and Storage.
