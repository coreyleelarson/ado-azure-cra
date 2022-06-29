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

- Under "Security + Networking" in the left navigation, select "Azure CDN".

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


### Assign Storage Account Permissions to Service Principal

- Navigate to "Service Connections" in your Azure DevOps Project Settings.

- Select your newly created service connection, and click "Manage Service Principal".

- Copy the "Display Name" to your clipboard and navigate to your previously created Storage Account.

- In the left navigation, click "Access Control (IAM)"

- Click "+ Add" and "Add role assignment"

- Select "Storage Blob Data Contributor" and click "Next"

- Ensure "User, Group, or service principal" is selected and click "+ Select Members"

- Paste your service principal display name in the input, select the service principal, and click "Select".

- Click "Review + Assign" to assign the permission.


### Setup Azure Build Pipeline

- In Azure DevOps, navigate to "Pipelines" -> "Pipelines" in the left navigation, and click "New Pipeline".

- Connect your repo from Azure Repos, Github, etc.

- Copy and paste the following sample **azure-pipelines.yml** file. This should kick off a build. If it does not, go ahead and run one.

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

- In Azure DevOps, navigate to "Pipelines" -> "Releases" in the left navigation, and click "+ New" -> "New Release Pipeline"

- Select "Empty Job"

- In the overlay, enter a name for your stage (i.e. "DEV" / "UAT" / "PROD") and close.

- Under "Artifacts", click "+ Add an Artifact", select "Build", and select your build pipeline from the source dropdown, and click "Add".

- Click the lightning bolt icon on your newly created artifact and enable the Continuous Deployment Trigger.

- Click "Tasks", select your stage, and click the "+" button next to "Agent Job".

**Azure Replace Tokens Task**

- In the right pane, add the "Replace tokens" task.

- Enter the following for your target files: `**/*`

- Select "custom" for token pattern.

- Enter the following for your token prefix: `__CONFIG_`

- Enter the following for your token suffex: `__`

**Azure File Copy Task**

- In the right pane, add the "Azure file copy" task.

- Enter the following for your source:

```
$(System.DefaultWorkingDirectory)/$(Release.PrimaryArtifactSourceAlias)/drop/*
```

- Select your service connection for the azure subscription.

- Select "Azure Blob" for the destination type.

- Select your storage account for the RM Storage Account.

- Enter "$web" for your container name.


**Azure CLI Task**

- In the right pane, add the "Azure CLI" task.

- Select your service connection.

- Select "Powershell" for the script type.

- Select "Inline Script" for the script location.

- Copy and paste the following into the inline script input:

```
az cdn endpoint purge -g $(ResourceGroupName) -n $(EndpointName) --profile-name $(ProfileName) --content-paths "/*" --no-wait
```

**Variables**

Click the "Variables" tab and add the following name/value pairs, selecting your stage for the "Scope":

|Name|Value|
|---|---|
|EndpointName|Your Azure CDN Endpoint Name|
|ProfileName|Your Azure CDN Profile Name|
|ResourceGroupName|Your Azure Resource Group Name|
|TestName|Dev Success!|
