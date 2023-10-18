# Azure Bicep Cheat sheet

This Azure Bicep Cheatsheet repository contain a set of quick code references for Azure Bicep!

## What is a cheat sheet?

A cheat sheet is a concise set of notes or a reference guide used for quick retrieval of essential information. It's often a single page that contains summaries, commands, formulas, or procedures that someone might need to reference frequently, especially when learning a new topic or skill.

## What is Azure Bicep?

Azure Bicep is a domain-specific language (also known as DSL) designed by Microsoft for defining and deploying Azure resources in a declarative manner. It's the next generation of Azure Resource Manager (ARM) templates, offering a cleaner syntax, improved type safety, and better support for modularization. While ARM templates use JSON syntax, Bicep uses a more concise syntax that aims to make it easier for developers to author and maintain Azure deployments

### **Installation**

- [Agnostic] Via Azure CLI: `az bicep install`
- [Windows] Via Chocolatey: `choco install bicep`
- [MacOS] Via Homebrew: `brew install bicep`

<details>
  <summary><h2>Basics</h2></summary>

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
  
  **Via name**
  
  ```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' = {
    name: 'my-vnet'
  }
  
  resource resChildSubnet 'Microsoft.Network/virtualNetworks/subnets@2022-01-01' = {
    name: '${resVnet}/my-subnet'
  }
  ```
  
  **Via parent property**
  
  ```bicep
  resource resVnet 'Microsoft.Network/virtualNetworks@2022-01-01' = {
    name: 'my-vnet'
  }
  
  resource resChildSubnet 'Microsoft.Network/virtualNetworks/subnets@2022-01-01' = {
    name: 'my-subnet'
    parent: resVnet
  }
  ```
  
  **Via parent resource**
  
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
  
  // query child resource
  output outChildSubnetId string = resVnet::resChildSubnet.id
  ```
</details>
