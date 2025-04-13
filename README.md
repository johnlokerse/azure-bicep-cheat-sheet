# Azure Bicep Cheat Sheet

Azure Bicep has become my preferred tool for writing infrastructure-as-code for Azure, thanks to its modularity and effortless adaptability. This repository serves as a handy cheat sheet, offering code snippets and references for Azure Bicep.

> [!NOTE]
> Community contributions are welcome :-)! If you have content that you want to add, please add it to the cheat sheet using a pull request.

<p align="center">
  <img src="images/AzureBicepCheatSheetHeader.png">
</p>

## What is a cheat sheet?

A cheat sheet is a concise set of notes or a reference guide used for quick retrieval of essential information. It's often a single page that contains summaries, commands, formulas, or procedures that someone might need to reference frequently, especially when learning a new topic or skill.

## What is Azure Bicep?

Azure Bicep is a domain-specific language (also known as DSL) designed by Microsoft for defining and deploying Azure resources in a declarative manner. It's the next generation of Azure Resource Manager (ARM) templates, offering a cleaner syntax, improved type safety, and better support for modularization. While ARM templates use JSON syntax, Bicep uses a more concise syntax that aims to make it easier for developers to author and maintain Azure deployments.

## Topics

> [!NOTE]
> Click the arrow next to a topic to expand its content.

<details>
  <summary>
    <span><b>Basics</b></span>
    <p><i>Declarations of new and existing resources, variables, parameters and outputs, etcetera.</i></p>
  </summary>

### Create a resource

[GitHub Copilot Prompt - Learn more on resource creation](https://github.com/copilot?prompt=how%20to%20define%20a%20resource%20in%20azure%20bicep)

how to define a resource in azure bicep

```bicep
resource resourceName 'ResourceType@version' = {
  name: 'exampleResourceName'
  properties: {
    // resource properties here
  }
}
```

### Create a child resource

[GitHub Copilot Prompt - Learn more about creating child resources](https://github.com/copilot?prompt=Can%20you%20explain%20the%20differences%20between%20creating%20child%20resources%20in%20Azure%20Bicep%20using%20a%20name%20property,%20using%20a%20parent%20property,%20and%20nesting%20them%20within%20a%20parent%20resource%20block?%20Please%20include%20examples%20and%20clarify%20when%20to%20use%20each%20approach.)

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

### Multi-line strings

```bicep
var varMultiLineString = '''
  This is a
  Muli-line string
  variable.
'''
```

</details>

<details>
  <summary>
    <span><b>Modules</b></span>
    <p><i>Split your deployment into smaller, reusable components.</i></p>
  </summary>

### Create a module

[GitHub Copilot Prompt](https://github.com/copilot?prompt=Can%20you%20explain%20what%20Azure%20Bicep%20modules%20are%20and%20how%20they%20work?%20Also,%20how%20can%20you%20reference%20a%20module%20stored%20in%20an%20external%20Azure%20Container%20Registry%20(ACR)?)

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
    <span><b>Conditions</b></span>
    <p><i>Resource definitions based on conditions.</i></p>
  </summary>

[GitHub Copilot - Learn more about conditions](https://github.com/copilot?prompt=How%20do%20conditions%20work%20in%20Azure%20Bicep?%20Can%20you%20explain%20it%20in%20detail%20and%20provide%20examples%20of%20how%20to%20use%20conditions%20to%20control%20resource%20deployment%20or%20property%20values?)

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
    <span><b>Loops</b></span>
    <p><i>Loop constructions.</i></p>
  </summary>

[GitHub Copilot - Learn more about loops](https://github.com/copilot?prompt=How%20do%20you%20create%20loops%20in%20Azure%20Bicep?%20Can%20you%20show%20examples%20using%20the%20for%20keyword%20to%20deploy%20multiple%20resources%20or%20set%20values%20in%20an%20array?)

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
    <span><b>Data manipulation</b></span>
    <p><i>Functions used to manipulate data.</i></p>
  </summary>

[GitHub Copilot - Learn more about lambda functions](https://github.com/copilot?prompt=In%20Azure%20Bicep,%20how%20do%20the%20filter(),%20map(),%20and%20sort()%20functions%20work?%20Can%20you%20explain%20each%20function%20in%20detail%20and%20show%20how%20they%E2%80%99re%20used%20in%20practice?%20For%20example:%20%E2%80%A2output%20outProducts%20array%20=%20filter(varGroceryStore,%20item%20=%3E%20item.productPrice%20%3E=%204)%20%E2%80%A2output%20outDiscount%20array%20=%20map(range(0,%20length(varGroceryStore)),%20item%20=%3E%20{%20productNumber:%20item,%20productName:%20varGroceryStore[item].productName,%20discountedPrice:%20%27The%20item%20${varGroceryStore[item].productName}%20is%20on%20sale.%20Sale%20price:%20${(varGroceryStore[item].productPrice%20/%202)}%27%20})%20%E2%80%A2output%20outUsingSort%20array%20=%20sort(varGroceryStore,%20(a,%20b)%20=%3E%20a.productPrice%20%3C=%20b.productPrice)%20Please%20walk%20through%20what%20each%20of%20these%20functions%20does%20and%20when%20to%20use%20them)

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
    <span><b>User Defined Types</b></span>
    <p><i>Define custom complex data structures.</i></p>
  </summary>

[GitHub Copilot - Learn more on User Defined Types](https://github.com/copilot?prompt=Can%20you%20explain%20in%20detail%20how%20User%20Defined%20Types%20work%20in%20Azure%20Bicep?%20Please%20include%20examples%20of%20defining%20custom%20object%20types,%20including%20how%20to%20structure%20them%20and%20how%20to%20define%20optional%20properties.%20Optional%20properties%20should%20be%20defined%20using%20a%20?%20at%20the%20type%20level%20(e.g.,%20type%20Bla%20=%20{%20name:%20string?%20}).%20Also%20include%20examples%20of%20arrays%20of%20objects%20with%20optional%20properties,%20such%20as:type%20arrayWithObjectsType%20=%20{%20name:%20string%20age:%20int%20hasChildren:%20bool?%20hasPets:%20bool?%20}[])

### Primitive types

```bicep
// a string type with two allowed strings ('Standard_LRS' or 'Standard_GRS')
type skuType = 'Standard_LRS' | 'Standard_GRS'

// an integer type with one allowed value (1337)
type integerType = 1337

// an boolean type with one allowed value (true)
type booleanType = true

// Reference the type
param parMyStringType skuType
param parMyIntType integerType
param parMyBoolType booleanType
```

### A custom type that enforced an array with a specific object structure

```bicep
type arrayWithObjectsType = {
  name: string
  age: int
}[]

param parCustomArray arrayWithObjectsType = [
  {
    name: 'John'
    age: 30
  }
]
```

### Optional properties in objects (using ?)

```bicep
type arrayWithObjectsType = {
  name: string
  age: int
  hasChildren: bool?
  hasPets: bool?
}[]

param parCustomArray arrayWithObjectsType = [
  {
    name: 'John'
    age: 30
  }
  {
    name: 'Jane'
    age: 31
    hasPets: true
  }
  {
    name: 'Jack'
    age: 45
    hasChildren: true
    hasPets: true
  }
]
```

</details>

<details>
  <summary>
    <span><b>User Defined Functions</b></span>
    <p><i>Define custom complex expressions.</i></p>
  </summary>

[GitHub Copilot - Learn more about User Defined Functions](https://github.com/copilot?prompt=Can%20you%20explain%20in%20detail%20how%20User%20Defined%20Functions%20work%20in%20Azure%20Bicep?%20Please%20include%20examples%20that%20show%20how%20to%20define%20and%20use%20custom%20functions,%20including%20parameter%20definitions,%20return%20types,%20and%20function%20logic.%20Also%20include%20a%20simple%20example%20such%20as:func%20funcSayHelloTo()%20string%20=%3E%20%27Hello%20and%20welcome,%20John%20Doe%27%20output%20outName%20string%20=%20funcSayHelloTo()%20/*%20Outputs:%20%27Hello%20and%20welcome,%20John%20Doe%27%20*/)

### User-defined function syntax

```func <function-name> (<parameter-name> <data-type>) <return-type> => <expression>```

### Basic user-defined function

```bicep
func funcSayHelloTo() string => 'Hello and welcome, John Doe'
```

### User-defined function with parameters

```bicep
func funcSayHelloTo(name string) string => 'Hello and welcome, ${name}'
```

With multiple parameters:

```bicep
func funcPersonNameAndAge(name string, age int) string => 'My name is ${name} and my age is ${age}'
```

### User-defined function return types

```bicep
func funcReturnTypeArray() array => [1, 2, 3, 4, 5]
func funcReturnTypeObject() object => {name: 'John Doe', age: 31}
func funcReturnTypeInt() int => 1337
func funcReturnTypeBool(key string) bool => contains({}, key)
func funcReturnTypeUserDefinedType() customTypeUsedAsReturnType => {
  hello: 'world'
}

type customTypeUsedAsReturnType = {
  hello: string
}
```

</details>

<details>
  <summary>
    <span><b>Compile-time imports</b></span>
    <p><i>Import and export() enable reuse of user-defined types variables, functions.<br>Supported in Bicep and Bicepparam files.</i></p>
  </summary>

[GitHub Copilot - learn more about compile-time imports](https://github.com/copilot?prompt=Show%20me%20how%20to%20use%20export%20and%20import%20in%20Azure%20Bicep%20to%20reuse%20a%20storage%20account%20resource%20across%20multiple%20Bicep%20files.)

### export() decorator (shared.bicep)

```bicep
@export()
var region = 'we'

@export()
type tagsType = {
  Environment: 'Prod' | 'Dev' | 'QA' | 'Stage' | 'Test'
  CostCenter: string
  Owner: string
  BusinessUnit: string
  *: string
}
```

### import statement

```bicep
import { region, tagsType } from 'shared.bicep'

output outRegion string = region
output outTags tagsType = {
  Environment: 'Dev'
  CostCenter: '12345'
  BusinessUnit: 'IT'
  Owner: 'John Lokerse'
}
```

### import statement with alias

```bicep
using 'keyVault.bicep'
import { region as importRegion } from 'shared.bicep'

param parKeyVaultName = 'kv-${importRegion}-${uniqueString(importRegion)}'
```

### import statement using a wildcard

```bicep
import * as shared from 'shared.bicep'

output outRegion string = shared.region
output outTags shared.tagsType = {
  Environment: 'Dev'
  CostCenter: '12345'
  BusinessUnit: 'IT'
  Owner: 'John Lokerse'
}
```

</details>

<details>
  <summary>
    <span><b>Networking</b></span>
    <p><i>CIDR functions to make subnetting easier.</i></p>
  </summary>

[GitHub Copilot - learn more about the networking functions](https://github.com/copilot?prompt=Can%20you%20explain%20how%20to%20use%20CIDR%20functions%20in%20Azure%20Bicep%20to%20make%20subnetting%20easier?%20Show%20how%20functions%20like%20cidrSubnet(),%20cidrHost()%20and%20others%20help%20to%20calculate%20subnet%20ranges,%20addresses,%20or%20specific%20IPs%20based%20on%20a%20base%20CIDR%20block.%20Include%20practical%20examples%20such%20as%20splitting%20a%20/16%20address%20space%20into%20multiple%20/24%20subnets.)

### parseCidr() function

```bicep
output outParseCidrInformation object = parseCidr('192.168.1.0/24')
```

#### returns

```json
"outParseCidrInformation": {
  "type": "Object",
  "value": {
    "broadcast": "192.168.1.255",
    "cidr": 24,
    "firstUsable": "192.168.1.1",
    "lastUsable": "192.168.1.254",
    "netmask": "255.255.255.0",
    "network": "192.168.1.0"
  }
}
```

### cidrSubnet() function

```bicep
output outCidrSubnet string = cidrSubnet('192.168.1.0/24', 25, 0)
```

#### returns

```json
"outCidrSubnet": {
  "type": "String",
  "value": "192.168.1.0/25"
}
```

### cidrHost() function

```bicep
output outCidrHost array = [for i in range(0, 10): cidrHost('192.168.1.0/24', i)]
```

#### returns

```json
"outCidrHost": {
  "type": "Array",
  "value": [
    "192.168.1.1",
    "192.168.1.2",
    "192.168.1.3",
    "192.168.1.4",
    "192.168.1.5",
    "192.168.1.6",
    "192.168.1.7",
    "192.168.1.8",
    "192.168.1.9",
    "192.168.1.10"
  ]
}
```
</details>

<details>
  <summary>
    <span><b>Bicepconfig</b></span>
    <p><i>Customize your Bicep development experience.</i></p>
  </summary>

[GitHub Copilot - Learn more about bicepconfig.json](https://github.com/copilot?prompt=Tell%20me%20more%20about%20the%20bicepconfig.json%20in%20Azure%20Bicep.%20What%20does%20it%20do%20and%20why%20do%20I%20need%20it?)

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
    <span><b>Dependencies</b></span>
    <p><i>Implicit and explicit dependencies.</i></p>
  </summary>

[GitHub Copilot - learn more about dependencies](https://github.com/copilot?prompt=How%20do%20dependencies%20work%20in%20Azure%20Bicep?%20Explain%20it%20to%20me%20in%20detail%20and%20add%20examples.)

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
    <span><b>Deployment</b></span>
    <p><i>Orchestration commands to deploy Azure Bicep to your Azure Environment.</i></p>
  </summary>

### Azure CLI

| Scope           | Command                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| resourceGroup   | `az deployment group create --resource-group ResourceGroupName --template-file template.bicep --parameters parameters.bicepparam`   |
| subscription    | `az deployment sub create --location location --template-file template.bicep --parameters parameters.bicepparam`                    |
| managementGroup | `az deployment mg create --management-group-id ManagementGroupId --template-file template.bicep --parameters parameters.bicepparam` |
| tenant          | `az deployment tenant create --location location --template-file template.bicep --parameters parameters.bicepparam`                 |

### Azure PowerShell

| Scope           | Command                                                                                                                                                                     |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| resourceGroup   | `New-AzResourceGroupDeployment -ResourceGroupName "ResourceGroupName" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam`                         |
| subscription    | `New-AzDeployment -Location "Location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"`                                                       |
| managementGroup | `New-AzManagementGroupDeployment -ManagementGroupId "ManagementGroupId" -Location "location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"` |
| tenant          | `New-AzTenantDeployment -Location "Location" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.bicepparam"`                                                 |

</details>

<details>
  <summary>
    <span><b>Target Scopes</b></span>
    <p><i>Deployment scope definitions.</i></p>
  </summary>

[GitHub Copilot - Learn more about target scopes](https://github.com/copilot?prompt=What%20are%20target%20scopes%20in%20Azure%20Bicep%20when%20deploying%20to%20Azure?)

### Target scopes

The `targetScope` directive in Azure Bicep determines the level at which the Bicep template will be deployed within Azure. The default is `targetScope = 'resourceGroup'`.

Azure Bicep supports multiple levels of `targetScope`:

| Scope           | Description                                                                                                              |
| --------------- | ------------------------------------------------------------------------------------------------------------------------ |
| resourceGroup   | The Bicep file is intended to be deployed at the Resource Group level.                                                   |
| subscription    | The Bicep file targets a Subscription, allowing you to manage resources or configurations across an entire subscription. |
| managementGroup | For managing resources or configurations across multiple subscriptions under a specific Management Group.                |
| tenant          | The highest scope, targeting the entire Azure tenant. This is useful for certain global resources or policies.           |

```bicep
targetScope = 'resourceGroup'

resource resKeyVault 'Microsoft.KeyVault/vaults@2019-09-01' = {
  // key vault properties here
}
```

Use the scope property on modules to deploy on a different scope than the target scope:

```bicep
// Uses the targetScope
module modStorageModule1 'storage.bicep' = {
  name: 'storageModule1'
}

// Uses the scope of the module
module modStorageModule2 'storage.bicep' = {
  name: 'storageModule2'
  scope: resourceGroup('other-subscription-id', 'other-resource-group-name')
  // module properties here
}
```

</details>

<details>
  <summary>
    <span><b>Azure Verified Modules</b></span>
    <p><i>Microsoft building blocks for Azure Bicep right at your fingertips.</i></p>
  </summary>

[GitHub Copilot - Learn more about Azure Verified Modules](https://github.com/copilot?prompt=What%20are%20Azure%20Verified%20Modules%20(https://aka.ms/avm),%20and%20how%20can%20I%20use%20them%20in%20my%20Azure%20Bicep%20files?%20Please%20explain%20their%20purpose,%20benefits,%20and%20show%20an%20example%20of%20how%20to%20reference%20and%20deploy%20a%20verified%20module%20in%20a%20Bicep%20template.)

### Azure Verified Modules reference

When you're writing Bicep, you can reference Azure Verified Modules (AVM) directly in your Bicep files. To get access to the IntelliSense prompt, you need the Azure Bicep VSCode extension installed. Additionally, to restore the Bicep modules successfully, make sure you have access to the Microsoft Container Registry at mcr.microsoft.com.

As an example, here is how to reference to an Azure Key Vault from the Microsoft Container Registry:
![AVM](images/AVM-KV.gif)

More information on Azure Verified Modules can be found [here](https://aka.ms/avm).

</details>