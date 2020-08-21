---
title: "Choosing right Firewall service in Azure"
date: 2020-08-20
categories:
  - blog
tags:
  - Azure Firewall
  - Azure Application Gateway
  - Azure Front Door
  - NVA
---

Azure has multiple services providing firewall capabilities. Features provided by these services are well documented. Many enterprises have separate security and development teams. It is common for enterprise security teams to mandate an Azure service that provides firewall capability as part of architecture for an application. This post will help enterprise security teams make an informed decision on choosing right firewall service for applications running on Azure.

Below are three key Azure services that provide firewall services.

1. Azure Firewall
1. Azure Application Gateway
1. Azure Front Door

High-level features offered by Azure Firewall, Azure Application Gateway and Azure Front Door are discussed below.

## Azure Firewall

Azure firewall is a managed service. It provides high-availability. It uses built-in threat-intelligence based filtering of network traffic. It has unrestricted scalability to match growth in network traffic. It enables enterprises to author and apply policies for common tasks such as restricting access, preventing data exfiltration, etc.

## Azure Application Gateway

Azure Application Gateway is a layer 7 load balancer. It has built-in automatic failover and no downtime maintenance capabilities. Azure Application Gateway provides Web Application Firewall (WAF) services for ingress HTTP traffic. It provides SSL offloading capabilities which helps to removes decryption processing burden on web servers. Azure Application Gateway can route traffic based on URL Paths and supports cookie based session affinity.

## Azure Front Door

Azure Front Door provides secure entry point in Azure. It is a global HTTP load balancer with seamless failover capability. It helps prevent application and network layer attacks at the edge locations. Azure Front Door supports multiple configurations (e.g. round-robin, weighted round-robin, active/standby, etc.) for load balancing rules. It provides insights into the traffic through metrics such as client requests, latency, performance, etc. It provides optimized routing path between users and applications.

## Sample application

This post will cover creating a Java application with a known SQL injection vulnerability. This application will be deployed as a service controller on Azure Kubernetes Service (AKS). AKS Service controller will have a private IP address which will be reachable via Azure Firewall, Azure Application Gateway and Azure Front Door. Application architecture will look like below.

![design](/assets/aks/privateaksdevops.png)
