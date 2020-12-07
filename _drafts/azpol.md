# Introduction

Following are the Azure Policies configured by Enterprise Scale Landing Zone.

## Deny-Public-Endpoints-for-PaaS-Services PolicySet

Most of the Azure Platform-as-a-service (PaaS) services are created with a public IP address assigned to them. This is a good option for developers who want to quickly get started with these services. Public endpoint accelerates learning curve and is ideal when developing pilots and small-scales Proof Of Concept (PoC) implementations.

However, when these pilots/PoCs make transition to production-ready enterprise applications, their use of public IP addresses is sometimes overlooked.

Production workloads using public IPs without proper security measures in place can increase security risks. Malicious actors can potentially use public IP as a gateway to launch an attack. Many enterprise compliance policies do not allow use of public IP just to avoid exposure to such security risks.

*Deny-Public-Endpoints-for-PaaS-Services* PolicySet helps enterprises prevent prevents Azure services getting created with a public IP address in the first place. *Deny-Public-Endpoints-for-PaaS-Services* PolicySet consists of following policies.

    Deny-PublicEndpoint-CosmosDB
    Deny-PublicEndpoint-MariaDB
    Deny-PublicEndpoint-MySQL
    Deny-PublicEndpoint-PostgreSql
    Deny-PublicEndpoint-KeyVault
    Deny-PublicEndpoint-Sql
    Deny-PublicEndpoint-Storage
    Deny-PublicEndpoint-Aks

Together, *Deny-Public-Endpoints-for-PaaS-Services* policies above, prevent major Azure services such as CosmosDB, SQL, AKS, etc. being exposed over a public IP.

## Deploy-Diag-LogAnalytics

    Deploy-Diagnostics-NIC
    Deploy-Diagnostics-AA
    Deploy-Diagnostics-PublicIP
    Deploy-Diagnostics-LoadBalancer
    Deploy-Diagnostics-NetworkSecurityGroups
    Deploy-Diagnostics-KeyVault
    Deploy-Diagnostics-CognitiveServices
    Deploy-Diagnostics-DLAnalytics
    Deploy-Diagnostics-DataLakeStore
    Deploy-Diagnostics-EventHub
    Deploy-Diagnostics-iotHub
    Deploy-Diagnostics-LogicAppsWF
    Deploy-Diagnostics-LogicAppsISE
    Deploy-Diagnostics-RecoveryVault
    Deploy-Diagnostics-SearchServices
    Deploy-Diagnostics-ServiceBus
    Deploy-Diagnostics-SQLDBs
    Deploy-Diagnostics-SQLElasticPools
    Deploy-Diagnostics-APIMgmt
    Deploy-Diagnostics-ApplicationGateway
    Deploy-Diagnostics-Batch
    Deploy-Diagnostics-MySQL
    Deploy-Diagnostics-PostgreSQL
    Deploy-Diagnostics-CDNEndpoints
    Deploy-Diagnostics-CosmosDB
    Deploy-Diagnostics-DataFactory
    Deploy-Diagnostics-Firewall
    Deploy-Diagnostics-PowerBIEmbedded
    Deploy-Diagnostics-StreamAnalytics
    Deploy-Diagnostics-ExpressRoute
    Deploy-Diagnostics-ACI
    Deploy-Diagnostics-ACR
    Deploy-Diagnostics-VirtualNetwork
    Deploy-Diagnostics-VM
    Deploy-Diagnostics-VMSS
    Deploy-Diagnostics-VNetGW
    Deploy-Diagnostics-AKS
    Deploy-Diagnostics-Website
    Deploy-Diagnostics-AnalysisService
    Deploy-Diagnostics-EventGridTopic
    Deploy-Diagnostics-EventGridSub
    Deploy-Diagnostics-HDInsight
    Deploy-Diagnostics-RedisCache
    Deploy-Diagnostics-Relay
    Deploy-Diagnostics-SignalR
    Deploy-Diagnostics-TrafficManager
    Deploy-Diagnostics-WebServerFarm
    Deploy-Diagnostics-SQLMI
    Deploy-Diagnostics-TimeSeriesInsights
    Deploy-Diagnostics-MlWorkspace

Deploy-Sql-Security
    Deploy-Sql-Tde
    Deploy-Sql-SecurityAlertPolicies
    Deploy-Sql-AuditingSettings
    Deploy-Sql-vulnerabilityAssessmentsAppend-KV-SoftDelete

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