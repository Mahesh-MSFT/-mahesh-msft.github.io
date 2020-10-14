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

## Scenario

This post will evaluate Hub and spoke network topology and Azure vWAN using a scenario with 2 workflow as depicted below.

![design](/assets/vwanhub/vwanhub.jpg)

Each workflow is described at a high-level as below.

Hub VNet workflow:

1. An User uses a Point to Site (P2S) VPN client.
1. VPN client connects to Azure VPN Gateway deployed in Hub VNet.
1. Hub VNet is peered with a spoke VNet running a web server Virtual Machine (VM).
1. Hub VNet is also peered with another spoke VNet running database server VM.
1. Web server VM will need to fetch data from database server VM.
1. User will invoke web application running on web server VM.

vWAN hub workflow:

1. Azure Virtual WAN is deployed with a hub.
1. An user uses a Point to Site (P2S) VPN client to connect with vWAN hub.
1. A VNet running web server VM is connected with vWAN hub.
1. Another VNet running database server VM is also connected with vWAN hub.
1. Web server VM will need to fetch data from database server VM.
1. User will invoke web application running on web server VM.

## Hub VNet workflow

Hub VNet workflow is now discussed in detail below.

### Create Hub VNet and configure it with VPN gateway

Use following ARM script to create Hub VNet.

```armasm
"apiVersion": "2018-04-01",
"type": "Microsoft.Network/virtualNetworks",
"name": "[parameters('hubVnetName')]",
"location": "[resourceGroup().location]",
"properties": {
"addressSpace": {
    "addressPrefixes": [
    "[parameters('hubVnetAddressPrefix')]"
    ]
}
```

Define a gateway subnet as below

```armasm
"subnets": [
    {
        "name": "GatewaySubnet",
        "properties": {
            "addressPrefix": "[parameters('gatewaySubnetPrefix')]"
                }
    }
]
```

Create a VPN client root certificate to be configured with VPN gateway as shown below.

```azurepowershell-interactive
# Create a self-signed root certificate
$rootcert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
-Subject "CN=makshP2SRootCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign

# Generate a client certificate
$clientCert = New-SelfSignedCertificate -Type Custom -DnsName makshP2SChildCert -KeySpec Signature `
-Subject "CN=makshP2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $rootcert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")

# Export client certificate
$myclientpwd = ConvertTo-SecureString -String "<super-secret-password>" -Force -AsPlainText
Export-PfxCertificate -Password $myclientpwd -Cert (get-item -Path Cert:\CurrentUser\My\$($clientCert.Thumbprint)) -FilePath myp2svpnclientCert.pfx

```

Get the Base64 encoded raw certificate data to be used in ARM Template by running follwing PS command

```azurepowershell-interactive
# Get Base64 encoded raw certificate data to be used in ARM Template
$([Convert]::ToBase64String($rootcert.Export('Cert')))
```

Set the base64 encoded raw certificate data in the parameters file as shown below. Ensure that there is no line breaks in the value.

```armasm
"clientRootCertData": {
    "value": "<base64-encoded-raw-certificate-data-obtained-above>"
}
```

### Create Web Server VM VNet



## Additional resources

* [Choose between virtual network peering and VPN gateways in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/vnet-peering)
* [Hub-spoke network topology in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
* [Azure Virtual WAN](https://azure.microsoft.com/en-gb/services/virtual-wan/)