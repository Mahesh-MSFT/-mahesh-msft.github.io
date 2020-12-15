# Introduction

Following are the Azure Policies configured by Enterprise Scale Landing Zone.

## Deny-Public-Endpoints-for-PaaS-Services Policy Initiative

Most of the Azure Platform-as-a-service (PaaS) services are created with a public IP address assigned to them. This is a good option for developers who want to quickly get started with these services. Public endpoint accelerates learning curve and is ideal when developing pilots and small-scales Proof Of Concept (PoC) implementations.

However, when these pilots/PoCs make transition to production-ready enterprise applications, their use of public IP addresses is sometimes overlooked.

Production workloads using public IPs without proper security measures in place can increase security risks. Malicious actors can potentially use public IP as a gateway to launch an attack. Many enterprise compliance policies do not allow use of public IP just to avoid exposure to such security risks.

*Deny-Public-Endpoints-for-PaaS-Services* Policy Initiative (also referred as PolicySet) helps enterprises prevent prevents Azure services getting created with a public IP address in the first place. *Deny-Public-Endpoints-for-PaaS-Services* Policy Initiative consists of following policies.

    Deny-PublicEndpoint-CosmosDB
    Deny-PublicEndpoint-MariaDB
    Deny-PublicEndpoint-MySQL
    Deny-PublicEndpoint-PostgreSql
    Deny-PublicEndpoint-KeyVault
    Deny-PublicEndpoint-Sql
    Deny-PublicEndpoint-Storage
    Deny-PublicEndpoint-Aks

Together, *Deny-Public-Endpoints-for-PaaS-Services* policies above, prevent major Azure services such as CosmosDB, SQL, AKS, etc. being exposed over a public IP.

## Deploy-Diag-LogAnalytics Policy Initiative

Lack of auditing and diagnostics information at granular level can impact operational practices. Incomplete audit information makes it difficult to correlate logs from multiple Azure services and form a coherent debugging experience.

It is desirable that once Azure services are provisioned, they provide detailed information about Azure platform they interact with.
Such information can be broadly divided into logs and metrics. Each Azure service can be further categorized into its sub-components (e.g. An Azure Public IP resource has `DDoSProtectionNotifications`, `DDoSMitigationReports` and `DDoSMitigationFlowLogs` as its sub-components. Collecting diagnostic information at these sub-categories can greatly enhance auditing and debugging experience.

 *Deploy-Diag-LogAnalytics Policy Initiative* provides an easy option to enforce logs and metrics collection at a deeper level.Azure Policies which are part *Deploy-Diag-LogAnalytics Policy Initiative* can help enterprises gather logs and metrics as shown below.

| Policy Name  | Log Categories           |Metrics        |
|:-------------|:-----------------------:|----------------:|
Deploy-Diagnostics-AA|JobLogs JobStreams DscNodeStatus|AllMetrics
Deploy-Diagnostics-ACI||AllMetrics
Deploy-Diagnostics-ACR||AllMetrics
Deploy-Diagnostics-ActivityLog|Administrative Security ServiceHealth Alert Recommendation Policy Autoscale ResourceHealth|
Deploy-Diagnostics-AKS|kube-audit kube-apiserver kube-controller-manager kube-scheduler cluster-autoscaler|AllMetrics
Deploy-Diagnostics-AnalysisService|Engine Service|AllMetrics
Deploy-Diagnostics-APIMgmt|GatewayLogs|Gateway Requests Capacity EventHub Events
Deploy-Diagnostics-ApplicationGateway|ApplicationGatewayAccessLog ApplicationGatewayPerformanceLog ApplicationGatewayFirewallLog|AllMetrics
Deploy-Diagnostics-Batch|ServiceLog|AllMetrics
Deploy-Diagnostics-CDNEndpoints|CoreAnalytics|
Deploy-Diagnostics-CognitiveServices|Audit RequestResponse|AllMetrics
Deploy-Diagnostics-CosmosDB|DataPlaneRequests MongoRequests QueryRuntimeStatistics|Requests
Deploy-Diagnostics-DataFactory|ActivityRuns PipelineRuns TriggerRuns|AllMetrics
Deploy-Diagnostics-DataLakeStore|Audit Requests|AllMetrics
Deploy-Diagnostics-DLAnalytics|Audit Requests|AllMetrics
Deploy-Diagnostics-EventGridSub||AllMetrics
Deploy-Diagnostics-EventGridTopic||AllMetrics
Deploy-Diagnostics-EventHub|ArchiveLogs OperationalLogs AutoScaleLogs|AllMetrics
Deploy-Diagnostics-ExpressRoute|PeeringRouteLog|AllMetrics
Deploy-Diagnostics-Firewall|AzureFirewallApplicationRule AzureFirewallNetworkRule AzureFirewallDnsProxy|AllMetrics
Deploy-Diagnostics-HDInsight||AllMetrics
Deploy-Diagnostics-iotHub|Connections DeviceTelemetry C2DCommands DeviceIdentityOperations FileUploadOperations Routes D2CTwinOperations C2DTwinOperations TwinQueries JobsOperations DirectMethods E2EDiagnostics Configurations|AllMetrics
Deploy-Diagnostics-KeyVault|AuditEvent|AllMetrics
Deploy-Diagnostics-LoadBalancer|LoadBalancerAlertEvent LoadBalancerProbeHealthStatus|AllMetrics
Deploy-Diagnostics-LogicAppsISE|IntegrationAccountTrackingEvents|
Deploy-Diagnostics-LogicAppsWF|WorkflowRuntime|AllMetrics
Deploy-Diagnostics-MlWorkspace|AmlComputeClusterEvent AmlComputeClusterNodeEvent AmlComputeJobEvent AmlComputeCpuGpuUtilization AmlRunStatusChangedEvent|Run Model Quota Resource
Deploy-Diagnostics-MySQL|MySqlSlowLogs|AllMetrics
Deploy-Diagnostics-NetworkSecurityGroups|NetworkSecurityGroupEvent NetworkSecurityGroupRuleCounter|
Deploy-Diagnostics-NIC||AllMetrics
Deploy-Diagnostics-PostgreSQL|PostgreSQLLogs|AllMetrics
Deploy-Diagnostics-PowerBIEmbedded|Engine|AllMetrics
Deploy-Diagnostics-PublicIP|DDoSProtectionNotifications DDoSMitigationFlowLogs DDoSMitigationReports|AllMetrics
Deploy-Diagnostics-RecoveryVault|CoreAzureBackup AddonAzureBackupAlerts AddonAzureBackupJobs AddonAzureBackupPolicy AddonAzureBackupProtectedInstance AddonAzureBackupStorage|
Deploy-Diagnostics-RedisCache||AllMetrics
Deploy-Diagnostics-Relay||AllMetrics
Deploy-Diagnostics-SearchServices|OperationLogs|AllMetrics
Deploy-Diagnostics-ServiceBus|OperationalLogs|AllMetrics
Deploy-Diagnostics-SignalR||AllMetrics
Deploy-Diagnostics-SQLDBs|SQLInsights AutomaticTuning QueryStoreRuntimeStatistics QueryStoreWaitStatistics Errors DatabaseWaitStatistics Timeouts Blocks Deadlocks SQLSecurityAuditEvents|AllMetrics
Deploy-Diagnostics-SQLElasticPools||AllMetrics
Deploy-Diagnostics-SQLMI|ResourceUsageStats SQLSecurityAuditEvents|
Deploy-Diagnostics-StreamAnalytics|Execution Authoring|AllMetrics
Deploy-Diagnostics-TimeSeriesInsights||AllMetrics
Deploy-Diagnostics-TrafficManager|ProbeHealthStatusEvents|AllMetrics
Deploy-Diagnostics-VirtualNetwork|VMProtectionAlerts|AllMetrics
Deploy-Diagnostics-VM||AllMetrics
Deploy-Diagnostics-VMSS||AllMetrics
Deploy-Diagnostics-VNetGW|GatewayDiagnosticLog IKEDiagnosticLog P2SDiagnosticLog RouteDiagnosticLog RouteDiagnosticLog TunnelDiagnosticLog|AllMetrics
Deploy-Diagnostics-WebServerFarm||AllMetrics
Deploy-Diagnostics-Website||AllMetrics
|<img width=250/>|<img width=50/>|<img width=40/>|

## Deploy-Sql-Security Policy Initiative

SQL databases are prevalent Azure service in most Azure deployments. Unfortunately, they are also prime target for malicious activities from within and outside of an enterprise.

*Deploy-Sql-Security* Policy Initiative helps protect Azure SQL databases using a set of following policies.

### Deploy-Sql-Tde

SQL database and its backups are prone to risks of getting into hands of malicious actors. It's very easy to restore SQL database from either database files or backup. Without proper defence system in place, malicious actors can have access to all the data.

Ensuring that SQL database is encrypted at rest is one of the first steps towards building SQL database defence strategy. Azure SQL database Transparent Data Encryption (TDE) ensures that data is encrypted at rest without needing any application code level change.

A SQL database with TDE enabled makes it hard for malicious actors to get access to data it holds even if its compromised.

As Azure SQL database deployments within an enterprise increases, it is very important that governance around ensuring Azure SQL databases are created with TDE enabled is in place.

*Deploy-Sql-Tde* policy ensures that Azure SQL databases have TDE enabled.

### Deploy-Sql-SecurityAlertPolicies

Bad actors are on the constant lookout to access and exploit business-critical Azure SQL databases. Risk of such attempts going unnoticed can reduce an enterprise's ability to detect and respond to them. In worst case scenario, an enterprise may never know if its SQL database has been compromised.

Azure SQL database provides way to set up security alerts that can report suspicious activities on SQL server. Such alert sends email a to pre-configured email addresses and optionally to Azure subscription admins and owners.

*Deploy-Sql-SecurityAlertPolicies* helps to enforce enabling of security alerts on Azure SQL databases. Enterprise can benefit from identifying malicious activities such as SQL injection attack, brute force attack, etc. though these alert. Security alerts provide detailed information about every incident. This detailed information is surfaced in Azure portal as well as email message triggered.

### Deploy-Sql-AuditingSettings

A business-critical Azure SQL database can be subject to a large number of DML (Data Manipulation Language), DCL (Data Control Language) and DDL (Data Definition Language) commands as part of day to day operations. Without a clear control and insight into these operational activities, its challenging to distinguish between legitimate and suspicious operations.

Enabling SQL Auditing can help in gathering important information about all database activities. Its also a requirement for many industry/regional regulatory compliance requirements. SQL Auditing helps generating and reporting audit trail of database events.

*Deploy-Sql-AuditingSettings* helps in enforcing Azure SQL Database Auditing. This policy audits and reports key database events such as ownership changes, successful/failed logins, role membership changes, schema changes, etc. Enterprises can use this policy and audit trail it generates to gain rich insights into database operations and comply with industry/regional regulatory requirements.

### Deploy-Sql-vulnerabilityAssessments

Throughout it's lifecycle, Azure SQL database undergoes very large number of schema, permission and configuration changes. There is always a risk of such changes resulting in deviation from best practices. Excessive permissions, orphaned roles and many such configurational drifts can be exploited by malicious actors.

Azure SQL database has built-in vulnerability assessment service.  State of Azure SQL database through the lense of Microsoft's best practices for SQL database can be evaluated using vulnerability assessment. A vulnerability assessment scan identifies database and server level security risks. A remediation task in applicable scenario may be also generated to fix the vulnerability.

*Deploy-Sql-vulnerabilityAssessments* policy ensures that Azure SQL databases are configured with vulnerability assessment. The assessment scans are performed periodically and reports are stored in Azure storage account. Pre-defined email address is used to share the results of periodic scan results for reporting purposes. 


    Append-KV-SoftDelete
    DataProtectionSecurityCenter
    Deny-AppGW-Without-WAF
    Deny-ERPeering
    Deny-IP-forwarding
    Deny-Private-DNS-Zones
    Deny-PublicIP
    Deny-Storage-http
    Deny-Subnet-Without-Nsg
    Deny-Subnet-Without-Nsg
    Deploy-ASC-Monitoring
    Deploy-ASC-Security
    Deploy-ASC-Standard
    Deploy-AzActivity-Log
    Deploy-AzureBackup-on-VM
    Deploy-DDoSProtection
    Deploy-DNSZoneGroup-For-Blob-PrivateEndpoint
    Deploy-DNSZoneGroup-For-File-PrivateEndpoint
    Deploy-DNSZoneGroup-For-KeyVault-PrivateEndpoint
    Deploy-DNSZoneGroup-For-Queue-PrivateEndpoint
    Deploy-DNSZoneGroup-For-Sql-PrivateEndpoint
    Deploy-DNSZoneGroup-For-Table-PrivateEndpoint
    Deploy-FirewallPolicy
    Deploy-HUB
    Deploy-LA-Config
    Deploy-Log-Analytics
    Deploy-LX-Arc-Monitoring
    Deploy-Nsg-FlowLogs
    Deploy-Resource-Diag
    Deploy-Sql-AuditingSettings
    Deploy-SQL-DB-Auditing
    Deploy-Sql-Security
    Deploy-Sql-SecurityAlertPolicies
    Deploy-Sql-Tde
    Deploy-Sql-vulnerabilityAssessments
    Deploy-vHUB
    Deploy-VM-Backup
    Deploy-VM-Monitoring
    Deploy-VMSS-Monitoring
    Deploy-vNet
    Deploy-vWAN
    Deploy-Windows-DomainJoin
    Deploy-WS-Arc-Monitoring
    Enforce-SQL-Encryption