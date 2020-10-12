---
title: "Choosing right networking hub in Azure"
date: 2020-10-12
categories:
  - blog
tags:
  - Azure Networking
  - Azure Virtual Network
  - Azure Virtual WAN
---

Azure Virtual Network provides a logical, isolated and secure network boundary inside Azure. As more and more workloads start to get deployed in Azure Virtual Networks, enabling network connectivity across them starts to become challenging. There are multiple options to address these challenges. These options include Virtual Network (VNet) Peering, VNet-to-VNet connectivity using VPN gateway, etc. To address common VNet connectivity challenges, a pattern has evolved over the years. This pattern is referred as "Hub-spoke network topology in Azure". Hub and spoke network topology pattern brings multiple benefits and simplifies network connectivity in Azure. Azure has another service called as Azure Virtual WAN (vWAN). Azure vWAN provides *managed hub and spoke topology* facilitating any-to-any connectivity. This post will compare and contrast Hub and spoke network topology and Azure vWAN to help customers make informed decision. 

## Additional resources

* [Choose between virtual network peering and VPN gateways in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/vnet-peering)
* [Hub-spoke network topology in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
* [Azure Virtual WAN](https://azure.microsoft.com/en-gb/services/virtual-wan/)