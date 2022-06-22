# ADO Azure CRA

## Prerequisites
- Azure Portal Account
- Azure DevOps Organization

## Azure Portal Setup

- Azure Subscription
- Azure Resource Group
- Azure Storage Account
- Azure CDN Profile & Endpoint


### Setup Azure Subscription

- From the Azure Portal Home, click "More Services", and click "Subscriptions" under the "General" heading.

- Click the "+ Add" button.

- Enter a unique name for your subscription.

- Review all other fields (for my testing purposes, each only had one option, but for client projects, this may have multiple).

- Click the "Create" button. It may take some time for the subscription to be available.


### Setup Azure Resource Group

- From the Azure Portal Home, click "More Services", and click "Resource Groups" under the "General" heading.

- Click the "+ Create" button.

- Select your previously created Subscription and enter a unique name for your Resource Group.

- Review all other fields (for my testing purposes, I kept the default Region of "(US) East US" but for client projects they may have a preference).

- Click the "Create" button. It may take some time for the resource group to be available.


### Setup Azure Storage Account

- From the Azure Portal Home, click "More Services", and click "Storage Accounts" under the "Storage" heading.

- Click the "+ Create" button.

- Select your previously created Subscription and Resource Group.

- Enter a unique name for your Storage Account.

- Review all other fields (for my testing purposes, I kept the defaults, but for client projects they may have a preference).

- Click the "Create" button. It may take some time for the storage account to be available.

### Configure Azure Storage Account Static Website

- Navigate to the newly created storage account resource.

- Under "Data Management" in the left navigation, select "Static Website".

- Click the "Enable" option, enter "index.html" for both document fields, and click "Save".

### Configure Azure CDN Profile and Endpoint

- Navigate to the newly created storage account resource.

- Under "Security + Networkgin" in the left navigation, select "Azure CDN".

- Under "New endpoint", select "Create New", enter a unique name for your CDN Profile, select a pricing tier (for my testing purposes, I selected the classic one), enter a unique name for your endpoint, and select the origin hostname that ends with "(Static Website)".

- Click the "Create" button. It may take some time for the CDN profile and endpoint to be available.


---

## Azure DevOps Setup

- Azure DevOps Service Connection
- Azure DevOps Build Pipeline
- Azure DevOps Release Pipeline


### Setup Azure Service Connection

- In your Azure DevOps project settings, select "Service Connections" under the "Pipelines" heading in the left navigation.

- Click "New Service Connection", select "Azure Resource Manager", and click "Next" at the bottom.

- Ensure "Service Principal (automatic)" is selected and click "Next".

- Select your newly created subscription and resource group from the dropdowns, enter a unique name for your service connection, select the checkbox to "Grant access permissions to all pipelines", and click "Save".


### Setup Azure Build Pipeline

- In Azure DevOps, navigate to "Pipelines" and click "New Pipeline".

- Connect your repo from Azure Repos, Github, etc.

- Copy and paste this sample **azure-pipelines.yml** file:

```
# Node.js with React

# Build a Node.js project that uses React.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/javascript

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '16.x'
  displayName: 'Install Node.js'

- script: |
    npm install
    npm run build
  displayName: 'Install Dependencies'

- script: |
    npm run build
  displayName: 'Create Production Build'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'build'
    ArtifactName: 'drop'
    publishLocation: 'Container'
  displayName: 'Publish Build Artifacts'
```


### Setup Azure Release Pipeline

This is it.