---
title: Register Azure Stack | Microsoft Docs
description: Register Azure Stack with your Azure subscription.
services: azure-stack
documentationcenter: ''
author: ErikjeMS
manager: byronr
editor: ''

ms.assetid: 
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2017
ms.author: erikje

---
# Register Azure Stack with your Azure Subscription

*Applies to: Azure Stack integrated systems and Azure Stack Development Kit*

You can register [Azure Stack](azure-stack-poc.md) with Azure to download marketplace items from Azure and to set up commerce data reporting back to Microsoft. 

> [!NOTE]
>Registration is recommended because it enables you to test important Azure Stack functionality, like marketplace syndication and usage reporting. After you register Azure Stack, usage is reported to Azure commerce. You can see it under the subscription you used for registration. Azure Stack Development Kit users aren't charged for any usage they report.
>


## Get Azure subscription

Before registering Azure Stack with Azure, you must have:

- The subscription ID for an Azure subscription. To get the ID, sign in to Azure, click **More services** > **Subscriptions**, click the subscription you want to use, and under **Essentials** you can find the **Subscription ID**. China, Germany, and US government cloud subscriptions are not currently supported.
- The username and password for an account that is an owner for the subscription (MSA/2FA accounts are supported).
- The Azure Active Directory for the Azure subscription. You can find this directory in Azure by hovering over your avatar at the top right corner of the Azure portal. 
- [Registered the Azure Stack resource provider](#register-azure-stack-resource-provider-in-azure).

If you don’t have an Azure subscription that meets these requirements, you can [create a free Azure account here](https://azure.microsoft.com/en-us/free/?b=17.06). Registering Azure Stack incurs no cost on your Azure subscription.



## Register Azure Stack resource provider in Azure
> [!NOTE] 
> This step only needs to be completed once in an Azure Stack environment.
>

1. Start Powershell ISE as an administrator.
2. Log in to the Azure account that is an owner of the Azure subscription with -EnvironmentName parameter set to "AzureCloud".
3. Register the Azure resource provider "Microsoft.AzureStack".

Example: 
```Powershell
Login-AzureRmAccount -EnvironmentName "AzureCloud"
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.AzureStack
```


## Register Azure Stack with Azure

> [!NOTE]
>All these steps must be completed on the host computer.
>

1. [Install PowerShell for Azure Stack](azure-stack-powershell-install.md). 
2. Copy the [RegisterWithAzure.psm1 script](https://go.microsoft.com/fwlink/?linkid=842959) to a folder (such as C:\Temp).
3. Start PowerShell ISE as an administrator and import the RegisterWithAzure module.    
4. Ensure you are logged into the correct Azure Powershell context that you want to register your Azure Stack environment with

    ```Powershell
    Set-AzureRmContext -SubscriptionId $YourAzureSubscriptionId
    ```
    
5. From the RegisterWithAzure.psm1 module, run the Set-AzsRegistration function. Replace the following placeholders: 
    - *YourCloudAdminCredential* is a PowerShell object that contains the local domain credentials for the domain\cloudadmin (for the development kit, this is azurestack\cloudadmin). 
    - *YourPrivilegedEndpoint* is the name of the [privileged end point](azure-stack-privileged-endpoint.md).

    ```powershell
    Set-AzsRegistration -CloudAdminCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint
    ```

## Verify the registration

1. Sign in to the administrator portal (https://adminportal.local.azurestack.external).
2. Click **More Services** > **Marketplace Management** > **Add from Azure**.
3. If you see a list of items available from Azure (such as WordPress), your activation was successful.

> [!NOTE]
> After registration is complete, the active warning for not registering will no longer appear.

## Modify the registration

If you would like to change the billing model or the subscription of your registration you must first run Remove-AzsRegistration, ensure you are logged in to the correct Azure Powershell context, and then run Set-AzsRegistration with any changed parameters

```Powershell
Remove-AzsRegistration -CloudAdminCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint
Set-AzureRmContext -SubscriptionId $NewSubscriptionId
Set-AzsRegistration -CloudAdminCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel PayAsYouUse
```

## Disconnected registration

If you are registering Azure Stack in a disconnected environment you will need to get a registration token from the AzureStack environment and then use that token on a machine that can connect to Azure for registration.

AzureStack Environment
1. Get registration token
```Powershell
$FilePathForRegistrationToken = $env:SystemDrive\RegistrationToken.txt

$RegistrationToken = Get-AzsRegistrationToken -CloudAdminCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel Capacity -AgreementNumber -TokenOutputFilePath $FilePathForRegistrationToken
```
2. Save this registration token for use in the Azure connected machine

Azure Connected Machine
1. Copy the [RegisterWithAzure.psm1 script](https://go.microsoft.com/fwlink/?linkid=842959) to a folder (such as C:\Temp).
2. Start PowerShell ISE as an administrator and import the RegisterWithAzure module.    
3. Ensure you are logged into the correct Azure Powershell context that you want to register your Azure Stack environment with

    ```Powershell
    Set-AzureRmContext -SubscriptionId $YourAzureSubscriptionId
    ```
4. Use the registration token to register with Azure

```Powershell
$registrationToken = "*Your Registration Token*"
Register-AzsEnvironment -RegistrationToken $registrationToken
```

> [NOTE!]
> You must save either the registration resource name or the registration token for future reference

If you want to remove the registration resource then you must use UnRegister-AzsEnvironment and pass in either the registration resource name or the registration token used for Register-AzsEnvironment

```Powershell
UnRegister-AzsEnvironment -RegistrationName "*Name of the registration resource*"
```
or
```Powershell
$registrationToken = "*Your copied registration token*"
UnRegister-AzsEnvironment -RegistrationToken $registrationToken
```

## Next steps

[Connect to Azure Stack](azure-stack-connect-azure-stack.md)

