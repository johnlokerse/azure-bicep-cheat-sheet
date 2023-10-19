# Azure Bicep Cheat Sheet

Azure Bicep has become my preferred tool for writing infrastructure-as-code for Azure, thanks to its modularity and effortless adaptability. This repository serves as a handy cheat sheet, offering code snippets and references for Azure Bicep.

> Community contributions are appreciated :-)! If you have content to add, please add it to the sheet! Make sure to check the [contribution guidelines](CONTRIBUTING.md) before contributing.

<p align="center">
  <img src="images/AzureBicepCheatSheetHeader.png">
</p>

## What is a cheat sheet?

A cheat sheet is a concise set of notes or a reference guide used for quick retrieval of essential information. It's often a single page that contains summaries, commands, formulas, or procedures that someone might need to reference frequently, especially when learning a new topic or skill.

## What is Azure Bicep?

Azure Bicep is a domain-specific language (also known as DSL) designed by Microsoft for defining and deploying Azure resources in a declarative manner. It's the next generation of Azure Resource Manager (ARM) templates, offering a cleaner syntax, improved type safety, and better support for modularization. While ARM templates use JSON syntax, Bicep uses a more concise syntax that aims to make it easier for developers to author and maintain Azure deployments.

## Topics

> Click the arrow next to a topic to expand its content.

<details>
  <summary>
    <h2>Basics</h2> <br>
    <i>Declarations of new and existing resources, variables, parameters and outputs.</i>
  </summary>

### Create a resource

```bicep
  resource resourceName 'ResourceType@version' = {
    name: 'exampleResourceName'
    properties: {
      // resource properties here
    }
  }
```

### Create a child resource

#### Via name

```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' = {
    name: 'my-vnet'
  }

  resource resChildSubnet 'Microsoft.Network/virtualNetworks/subnets@2022-01-01' = {
    name: '${resVnet}/my-subnet'
  }
```

#### Via parent property

```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' = {
    name: 'my-vnet'
  }

  resource resChildSubnet 'Microsoft.Network/virtualNetworks/subnets@2022-01-01' = {
    name: 'my-subnet'
    parent: resVnet
  }
```

#### Via parent resource

```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' = {
    name: 'my-vnet'

    resource resChildSubnet 'subnets' = {
      name: 'my-subnet'
    }
  }
```

### Reference to an existing resource

```bicep
  resource resKeyVaultRef 'Microsoft.KeyVault/vaults@2019-09-01' = existing {
    name: 'myExistingKeyVaultName'
  }
```

### Access a nested resource (::)

```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' existing = {
    name: 'my-vnet'
    resource resChildSubnet 'subnets' existing = {
      name: 'my-subnet'
    }
  }

  // access child resource
  output outChildSubnetId string = resVnet::resChildSubnet.id
```

### Declare a variable

```bicep
  var varEnvironment = 'dev'
```

There is no need to declare a datatype for a variable, because the type is inferred from the value.

### Declare a parameter

```bicep
  param parStorageAccountName string
  param parLocation string = resourceGroup().location
```

Available datatypes are: `string`, `bool`, `int`, `object`, `array` and `custom (user defined type)`.

### Declare a secure parameter

```bicep
  @secure()
  param parSecureParameter string
```

### Declare an output

```bicep
  resource resPublicIp 'Microsoft.Network/publicIPAddresses@2023-02-01' ={
    name: parPublicIpName
    tags: parTags
    location: parLocation
    zones: parAvailabilityZones
    sku: parPublicIpSku
    properties: parPublicIpProperties
  }

  output outPublicIpId string = resPublicIp.id
  output outMyString string = 'Hello!'
```

Available datatypes are: `string`, `bool`, `int`, `object`, `array` and `custom (user defined type)`.

### String interpolation

```bicep
var varGreeting = 'Hello'
output outResult string = '${varGreeting} World'
```

</details>

<details>
  <summary>
    <h2>Modules</h2><br>
    <i>Split your deployment into smaller, reusable components.</i>
  </summary>

### Create a module

```bicep
  module modVirtualNetwork './network.bicep' = {
    name: 'networkModule'
    params: {
      parLocation: 'westeurope'
      parVnetName: 'my-vnet-name'
    }
  }

```

### Reference to a module using a bicep registry

```bicep
  module modBicepRegistryReference 'br/<bicep registry name>:<file path>:<tag>' = {
      name: 'deployment-name'
      params: {}
  }
```

</details>

<details>
  <summary>
    <h2>Conditions</h2><br>
    <i>Resource definitions based on conditions.</i>
  </summary>

### If condition

```bicep
param parDeployResource bool

resource resDnsZone 'Microsoft.Network/dnszones@2018-05-01' = if (parDeployResource) {
  name: 'myZone'
  location: 'global'
}
```

### Ternary if/else condition

```bicep
param parEnvironment string

var varSku = parEnvironment == 'prod' ? 'premium' : 'standard'
```

</details>

<details>
  <summary>
    <h2>Loops</h2><br>
    <i>Loop constructions</i>
  </summary>

### foreach using an array

```bicep
param parStorageAccountNames array = [
  'storageaccount1'
  'storageaccount2'
  'storageaccount3'
]

resource resStorageAccounts 'Microsoft.Storage/storageAccounts@2021-04-01' = [for name in parStorageAccountNames: {
  name: name
  location: 'westeurope'
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}]
```

### foreach using an array of objects

``` bicep
param parStorageAccountNames array = [
  {
      name: 'storageaccount1'
      kind: 'StorageV2'
      sku: {
          name: 'Standard_LRS'
      }
  }
  {
      name: 'storageaccount2'
      kind: 'StorageV2'
      sku: {
          name: 'Standard_LRS'
      }
  }
]

resource resStorageAccounts 'Microsoft.Storage/storageAccounts@2021-04-01' = [for storageAccount in parStorageAccountNames: {
  name: storageAccount.name
  location: 'westeurope'
  kind: storageAccount.kind
  sku: {
      name: storageAccount.sku
  }
}]
```

</details>

<details>
  <summary>
    <h2>Data manipulation</h2><br>
    <i>Functions used to manipulate data.</i>
  </summary>

### Example data

```bicep
var varGroceryStore = [
  {
    productName: 'Icecream'
    productPrice: 2
    productCharacteristics: [
      'Vegan'
      'Seasonal'
    ]
  }
  {
    productName: 'Banana'
    productPrice: 4
    productCharacteristics: [
      'Bio'
    ]
  }
]
```

### filter() function

```bicep
  output outProducts array = filter(varGroceryStore, item => item.productPrice >= 4)
```

#### returns

```json
[
  {
    "productName": "Banana",
    "productPrice": 4,
    "productCharacteristics": [
      "Bio"
    ]
  }
]
```

### map() function

```bicep
  output outDiscount array = map(range(0, length(varGroceryStore)), item => {
    productNumber: item
    productName: varGroceryStore[item].productName
    discountedPrice: 'The item ${varGroceryStore[item].productName} is on sale. Sale price: ${(varGroceryStore[item].productPrice / 2)}'
  })
```

#### returns

```json
[
  {
    "productNumber": 0,
    "productName": "Icecream",
    "discountedPrice": "The item Icecream is on sale. Sale price: 1"
  },
  {
    "productNumber": 1,
    "productName": "Banana",
    "discountedPrice": "The item Banana is on sale. Sale price: 2"
  }
]
```

### sort() function

```bicep
  output outUsingSort array = sort(varGroceryStore, (a, b) => a.productPrice <= b.productPrice)
```

#### returns

```json
[
  {
    "productName": "Icecream",
    "productPrice": 2,
    "productCharacteristics": [
      "Vegan"
      "Seasonal"
    ]
  },
  {
    "productName": "Banana",
    "productPrice": 4,
    "productCharacteristics": [
      "Bio"
    ]
  }
]
```

</details>

<details>
  <summary>
    <h2>User Defined Types</h2><br>
    <i>TODO.</i>
  </summary>
  TODO
</details>

<details>
  <summary>
    <h2>Bicepconfig</h2><br>
    <i>Customize your Bicep development experience.</i>
  </summary>

### Azure Container Registry configuration

```json
{
  "moduleAliases": {
    "br": {
      "<bicep registry name>": {
        "registry": "<url to registry>",
        "modulePath": "<module path of the alias>"
      }
    }
  }
}
```

</details>

<details>
  <summary>
    <h2>Dependencies</h2><br>
    <i>Implicit and explicit dependencies.</i>
  </summary>

### Implicit dependency using symbolic name

```bicep
  resource resNetworkSecurityGroup 'Microsoft.Network/networkSecurityGroups@2019-11-01' = {
    name: 'my-networkSecurityGroup'
    location: resourceGroup().location
  }
  resource nsgRule 'Microsoft.Network/networkSecurityGroups/securityRules@2019-11-01' = {
    name: '${resNetworkSecurityGroup}/AllowAllRule'
    properties: {
      // resource properties here
    }
  }
```

### Explicit dependency using dependsOn

```bicep
  resource resDnsZone 'Microsoft.Network/dnsZones@2018-05-01' = {
    name: 'contoso.com'
    location: 'global'
  }
  module modVirtualNetwork './network.bicep' = {
    name: 'networkModule'
    params: {
      parLocation: 'westeurope'
      parVnetName: 'my-vnet-name'
    }
    dependsOn: [
      resDnsZone
    ]
  }
```

</details>

<details>
  <summary>
    <h2>Deployment</h2><br>
    <i>Orchestration commands to deploy Azure Bicep to your Azure Environment.</i>
  </summary>

### Azure CLI

| Scope            | Command       |
| ---------------- | ------------- |
| resourceGroup    | `az deployment group create --resource-group ResourceGroupName --template-file template.bicep --parameters parameters.bicepparam`  |
| subscription     | `az deployment sub create --location location --template-file template.bicep --parameters parameters.bicepparam`  |
| managementGroup  | `az deployment mg create --management-group-id YourManagementGroupId --template-file template.bicep --parameters parameters.bicepparam`  |
| tenant           | `az deployment tenant create --location location --template-file template.bicep --parameters parameters.bicepparam`  |

### Azure PowerShell

| Scope            | Command       |
| ---------------- | ------------- |
| resourceGroup    | `New-AzResourceGroupDeployment -ResourceGroupName "YourResourceGroupName" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam`  |
| subscription     | `New-AzDeployment -Location "Location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"`  |
| managementGroup  | `New-AzManagementGroupDeployment -ManagementGroupId "ManagementGroupId" -Location "location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"`  |
| tenant           | `New-AzTenantDeployment -Location "Location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"`  |

</details>

<details>
  <summary>
    <h2>Target Scopes</h2><br>
    <i>Deployment scopes.</i>
  </summary>

### Target scopes

The `targetScope` directive in Azure Bicep determines the level at which the Bicep template will be deployed within Azure. The default is `targetScope = 'resourceGroup'`.

Azure Bicep supports multiple levels of `targetScope`:

| Scope           | Description     |
| --------------- | --------------- |
| resourceGroup   | The Bicep file is intended to be deployed at the Resource Group level. |
| subscription    | The Bicep file targets a Subscription, allowing you to manage resources or configurations across an entire subscription. |
| managementGroup | For managing resources or configurations across multiple subscriptions under a specific Management Group. |
| tenant          | The highest scope, targeting the entire Azure tenant. This is useful for certain global resources or policies. |

</details>
