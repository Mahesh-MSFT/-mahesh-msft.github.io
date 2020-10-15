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

Create public IP address to be assigned for VPN gateway as shown below.

```armasm
{
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[parameters('gatewayPublicIPName')]",
    "location": "[resourceGroup().location]",
    "properties": {
    "publicIPAllocationMethod": "Dynamic"
    }
}
```

Create VPN Gateway as shown below.

```armasm
{
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Network/virtualNetworkGateways",
    "name": "[parameters('gatewayName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', parameters('gatewayPublicIPName'))]",
        "[concat('Microsoft.Network/virtualNetworks/', parameters('hubVnetName'))]"
    ]
}
```

Assign the public IP and subnet as shown below.

```armasm
"ipConfigurations": [
    {
    "properties": {
            "privateIPAllocationMethod": "Dynamic",
            "subnet": {
            "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
            "id": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('gatewayPublicIPName'))]"
            }
        },
        "name": "vnetGatewayConfig"
    }
]
```

Configure VPN certificate as below.

```armasm
"vpnClientConfiguration": {
    "vpnClientAddressPool": {
        "addressPrefixes": [
            "[parameters('vpnClientAddressPoolPrefix')]"
        ]
    },
        "vpnClientRootCertificates": [
        {
            "name": "[parameters('clientRootCertName')]",
            "properties": {
                "PublicCertData": "[parameters('clientRootCertData')]"
            }
        }
    ]
}
```

### Create Web Server VNet and VM

Define the Web Server VNet with following template.

```armasm
 {
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('spoke1VnetName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', parameters('hubVnetName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', 'spoke-vnet1-subnet-nsg')]"
    ],
    "properties": {
    "addressSpace": {
        "addressPrefixes": [
        "[parameters('spoke1VnetAddressPrefix')]"
        ]
    },
}
``` 

Define a default subnet as shown below.

```armasm
"subnets": [
    {
        "name": "default",
        "properties": {
            "addressPrefix": "[parameters('spoke1SubnetAddressPrefix')]",
            "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', 'spoke-vnet1-subnet-nsg')]"
            }
        }   
    }
]
```

Create a Web Server VM as shown below.

```armasm
{
    "apiVersion": "2017-03-30",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[parameters('spoke1Vm1Name')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', parameters('vm1-spoke1-nic'))]"
    ],
    "properties": {
        "hardwareProfile": {
        "vmSize": "Standard_A2_v2"
        },
    "osProfile": {
        "computerName": "[parameters('spoke1Vm1Name')]",
        "adminUsername": "spoke1vm1-uid",
        "adminPassword": "P@ssw0rd1234"
    },
    "storageProfile": {
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2012-R2-Datacenter",
            "version": "latest"
        },
        "osDisk": {
            "name": "[concat(parameters('spoke1Vm1Name'),'_OSDisk')]", 
            "caching": "ReadWrite",
            "createOption": "FromImage"
            }
    },
    "networkProfile": {
        "networkInterfaces": [
                {
                "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('vm1-spoke1-nic'))]"
                }
            ]
        }
    }
}
```

### Create Network Peering between Hub VNet and Web Server VNet

Configure the peering as shown below in Web Server VNet.

```armasm
{
    "apiVersion": "2020-05-01",
    "type": "virtualNetworkPeerings",
    "name": "[variables('SpokevNet1toHubvNetPeeringName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks/', parameters('hubVnetName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks/', parameters('spoke1VnetName'))]"
    ],
    "comments": "Peering from Spoke vNet 1 to Hub vNet",
    "properties": {
        "allowVirtualNetworkAccess": true,
        "allowForwardedTraffic": false,
        "allowGatewayTransit": false,
        "useRemoteGateways": true,
        "remoteVirtualNetwork": {
            "id": "[resourceId('Microsoft.Network/virtualNetworks',parameters('hubVnetName'))]"
        }
    }
}
```

Similarly, configure reciprocal peering in Hub VNet as shown below.

```armasm
{
    "apiVersion": "2020-05-01",
    "type": "virtualNetworkPeerings",
    "name": "[variables('HubvNettoSpokevNet1PeeringName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks/', parameters('hubVnetName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks/', parameters('spoke1VnetName'))]"
    ],
    "comments": "Peering from Hub vNet to Spoke vNet 1",
    "properties": {
        "allowVirtualNetworkAccess": true,
        "allowForwardedTraffic": false,
        "allowGatewayTransit": true,
        "useRemoteGateways": false,
        "remoteVirtualNetwork": {
            "id": "[resourceId('Microsoft.Network/virtualNetworks',parameters('spoke1VnetName'))]"
        }
    }
}
```

### Create Database Server VNet and VM

Follow the same steps as followed in creating Web Sever VNet and VM. Change the configuration values as appropriate.

### Create Network Peering between Hub VNet and Database Server VNet

Configure the network peering using same process as followed for Web Server peering. Ensure that configuration values are replaced as appropriate.  

### Validate connectivity between Web and Database Servers

To validate connectivity between Web and Database servers, connect with the Hub VNet using VPN first. Then using the VPN connectivity, RDP session with either Web or Database server will be established.

To use VPN client, download the client by running following command.

```azurecli-interactive
az network vnet-gateway vpn-client generate -n hubvnetGateway `
    --processor-architecture Amd64 `
    -g $rgName
```

After downloading, launch the appropriate client (Windows (32, 64) or Mac).

![launchvpn](/assets/vwanhub/hubvnetvpn.jpg)

Ensure that connection is established successfully after clicking "Connect" button.
 
![connectvpn](/assets/vwanhub/vnetvpn.jpg)


## Additional resources

* [Choose between virtual network peering and VPN gateways in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/vnet-peering)
* [Hub-spoke network topology in Azure](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
* [Azure Virtual WAN](https://azure.microsoft.com/en-gb/services/virtual-wan/)