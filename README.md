{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vnet1Name": {
            "type": "string",
            "metadata": {
                "description": "Name of the CoreServiceVnet.",
                "displayName": "CoreServiceVnet"
            },
            "defaultValue": "CoreServiceVnet"
        },
        "vnet1subnetOneName": {
            "type": "string",
            "metadata": {
                "description": "Name of the SharedServiceSubnet.",
                "displayName": "SharedServiceSubnet"
            },
            "defaultValue": "SharedServiceSubnet"
        },
        "vnet1subnetTwoName": {
            "type": "string",
            "metadata": {
                "description": "Name of the DatabaseSubnet.",
                "displayName": "DatabaseSubnet"
            },
            "defaultValue": "DatabaseSubnet"
        },
        "vnet2Name": {
            "type": "string",
            "metadata": {
                "description": "Name of the ManufacturingVnet."
            },
            "defaultValue": "ManufacturingVnet"
        },
        "vnet2subnetOneName": {
            "type": "string",
            "metadata": {
                "description": "Name of the sensorSubnet1."
            },
            "defaultValue": "sensorSubnet1"
        },
        "vnet2subnetTwoName": {
            "type": "string",
            "metadata": {
                "description": "Name of the sensorSubnet2."
            },
            "defaultValue": "sensorSubnet2"
        },
        "vnet1SubnetNSG": {
            "type": "string",
            "metadata": {
                "description": "Name of the Network Security Group for CoreServiceVnet subnets."
            },
            "defaultValue": "SharedServicesSubnetNSG"
        },
        "asgWebName":{
            "type": "string",
            "metadata": {
                "description": "Name of the application security group."
            },
            "defaultValue": "asg-web"
        },
        "myNSGSecureName":{
            "type": "string",
            "metadata": {
                "description": "name of the network security group that allows asg traffic and denies internet access."
            },
            "defaultValue": "myNSGSecure"
        },
        "publicDNSZoneName":{
            "type": "string",
            "metadata":{
                "description":"Name of the public DNS zone"
            },
            "defaultValue": "contoso.com"
        },
        "privateDNSZoneName":{
            "type": "string",
            "metadata":{
                "description":"name of the private DNS zone"
            },
            "defaultValue":"private.contoso.com"
        }
    },
    "variables": {
        "vnet1AddressPrefix": "10.20.0.0/16",
        "vnet1subnetOneNameAddress": "10.20.10.0/24",
        "vnet1subnetTwoNameAddress": "10.20.20.0/24",
        "vnet2AddressPrefix": "10.30.0.0/16",
        "vnet2subnetOneNameAddress": "10.30.20.0/24",
        "vnet2subnetTwoNameAddress": "10.30.21.0/24",
        "wwwRecordIP": "10.1.1.4",
        "sensorvmIP": "10.1.1.4"
    },
    "resources": [
        {
            "name": "[parameters('vnet1Name')]",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2024-03-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "virtualNetwork1"
            },
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('vnet1SubnetNSG'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('vnet1AddressPrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[parameters('vnet1subnetOneName')]",
                        "properties": {
                            "addressPrefix": "[variables('vnet1subnetOneNameAddress')]",
                            "networkSecurityGroup": {
                                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('vnet1SubnetNSG'))]"
                            }
                        }
                    },
                    {
                        "name": "[parameters('vnet1subnetTwoName')]",
                        "properties": {
                            "addressPrefix": "[variables('vnet1subnetTwoNameAddress')]",
                            "networkSecurityGroup": {
                                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('vnet1SubnetNSG'))]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "name": "[parameters('vnet2Name')]",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2024-03-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "virtualNetwork2"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('vnet2AddressPrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[parameters('vnet2subnetOneName')]",
                        "properties": {
                            "addressPrefix": "[variables('vnet2subnetOneNameAddress')]"
                        }
                    },
                    {
                        "name": "[parameters('vnet2subnetTwoName')]",
                        "properties": {
                            "addressPrefix": "[variables('vnet2subnetTwoNameAddress')]"
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2024-03-01",
            "location": "[resourceGroup().location]",
            "name": "[parameters('vnet1SubnetNSG')]",
            "properties": {}
        },
        {
            "type": "Microsoft.Network/applicationSecurityGroups",
            "apiVersion": "2024-03-01",
            "location": "[resourceGroup().location]",
            "name": "[parameters('asgWebName')]",
            "properties": {}
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2024-03-01",
            "location": "[resourceGroup().location]",
            "name": "[parameters('myNSGSecureName')]",
            "properties":{
                "securityRules":[
                    {
                        "name":"AllowASG",
                        "properties":{
                            "priority":100,
                            "access":"Allow",
                            "direction":"Inbound",
                            "destinationPortRanges":[
                                "80",
                                "443"
                            ],
                            "protocol":"Tcp",
                            "sourceApplicationSecurityGroups":[
                                {
                                    "id":"[resourceId('Microsoft.Network/applicationSecurityGroups', parameters('asgWebName'))]"
                                }
                            ],
                            "sourcePortRange":"*",
                            "destinationAddressPrefix":"*"
                        }
                    },
                    {
                        "name":"DenyAnyCustom8080Outbound",
                        "properties":{
                            "priority":4096,
                            "access":"Deny",
                            "direction":"Outbound",
                            "destinationPortRanges":[
                                "8080"
                            ],
                            "protocol":"*",
                            "destinationServiceTags":[
                                "Internet"
                            ],
                            "sourceAddressPrefix":"*"
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/dnsZones",
            "apiVersion": "2018-05-01",
            "name": "[parameters('publicDNSZoneName')]",
            "location": "global",
            "properties": {}
        },
        {
            "type": "Microsoft.Network/dnsZones/A",
            "apiVersion": "2018-05-01",
            "name": "[concat(parameters('publicDNSZoneName'), '/www')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/dnsZones', parameters('publicDNSZoneName'))]"
            ],
            "properties": {
                "ttl": 3600,
                "aRecords": [
                    {
                        "ipv4Address": "[variables('wwwRecordIP')]"
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/privateDnsZones",
            "apiVersion": "2020-06-01",
            "name": "[parameters('privateDNSZoneName')]",
            "location": "global",
            "properties": {}
        },
        {
            "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
            "apiVersion": "2020-06-01",
            "name": "[concat(parameters('privateDNSZoneName'), '/manufacturing-link')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDNSZoneName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnet2Name'))]"
            ],
            "properties": {
                "registrationEnabled": false,
                "virtualNetwork": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnet2Name'))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/privateDnsZones/A",
            "apiVersion": "2020-06-01",
            "name": "[concat(parameters('privateDNSZoneName'), '/sensorvm')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDNSZoneName'))]"
            ],
            "properties": {
                "ttl": 3600,
                "aRecords": [
                    {
                        "ipv4Address": "[variables('sensorvmIP')]"
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
