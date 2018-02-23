#Securing Azure Infrastructure - Hands on Lab Guide

**Authors:**
Adam Raffe - Microsoft
Tom Wilde - Microsoft

February 2018

# Contents

**[Prerequisites](#prereqs)**

**[VDC Lab Introduction](#intro)**

**[Initial Lab Setup](#setup)**

**[Lab 1: Azure Security Center](#asc)**

- [1.1: Enable Azure Security Center](#enableasc)

- [1.2: Explore Azure Security Center](#exploreasc)

**[Lab 2: Securing Azure Storage](#storage)**

- [2.1: Enable Logging for the Storage Account](#storagelogging)

- [2.2: Remove Public Access to Blob Storage](#removeblobaccess)

- [2.3: Implement Shared Access Signatures](#sas)

- [2.4: Inspect Logs](#storagelogs)

**[Lab 3: Securing Azure Networking](#azurenetworking)**

**[Lab 4: Just in Time VM Access](#jit)**

- [4.1: Apply Just in Time Access](#applyjit)

- [4.2: Request Access to VMs](#requestjit)

**[Lab 5: Encrypting Virtual Machines](#ade)**

**[Lab 6: Securing Azure SQL](#sql)**

- [6.1: Enable SQL Database Firewall](#sqlfirewall)

- [6.2: Enable SQL Database Audting and Threat Detection](#sqlaudit)

**[Lab 7: Privileged Identity Management](#pim)**

- [7.1: Enable and Configure PIM](#enablepim)

- [7.2: Test PIM Access](#testpim)

- [7.3: Assign Users and Roles to Resources](#assignroles)

- [7.4: Managing Azure Resources](#managingresources)

**[Decommission the lab](#decommission)**

**[Conclusion](#conclusion)**

**[Useful References](#ref)**

# Prerequisites <a name="prereqs"></a>

To complete this workshop, the following will be required:

- A valid subscription to Azure. If you don't currently have a subscription, consider setting up a free trial. If this workshop is being run by a Microsoft employee, Azure passes should be provided.

- Multiple browser windows will be required to log in as different users simultaneously.

- A mobile phone, used to respond to multi-factor authentication challenges.

# Lab Introduction <a name="intro"></a>

Contoso have recently migrated several of their on-premises resources to Microsoft Azure. These resources include virtual machines (both Windows 2016 and Ubuntu Linux), virtual networks and storage accounts. Unfortunately, as this is the first migration carried out, Contoso are somewhat unfamiliar with Azure (and public cloud platforms in general) – as a result, they have failed to consider the security implications of the infrastructure.

The Contoso security team have requested your help to secure the infrastructure resources that they have migrated to Azure.

The environment deployed by Contoso is shown in figure 1.

![Main Lab Image](https://github.com/araffe/vdc-security-lab/blob/master/images/Overview.jpg "Security Lab Environment")

**Figure 1:** Contoso Environment

The migrated Contoso environment has the following issues:

•	The storage account / container used has open, public access.
•	There is no access control in place for the virtual network / subnet.
•	Virtual Machines are not encrypted.
•	No Role Based Access Control (RBAC) is in place to determine which users have access to which resources. Contoso would like only the minimum amount of access to be given to users, including time limited access.
•	The Azure SQL Database has no firewall rules configured.


# Initial Lab Setup <a name="setup"></a>

**Important: The initial lab setup using ARM templates takes around 45 minutes - please initiate this process as soon as possible to avoid a delay in starting the lab.**

*All usernames and passwords for virtual machines are set to labuser / M1crosoft123*

Perform the following steps to initialise the lab environment:

**1)** As we need an Azure subscription licensed for Office 365 and EM+S, the best process for this is to first create an Office 365 trial account by navigating here: http://go.microsoft.com/fwlink/p/?LinkID=698279&culture=en-GB&country=GB

**2)** Fill in the details, complete the sign-up process and create an “admin” user, as shown in figure 2.

![Office 365 Signup](https://github.com/araffe/vdc-security-lab/blob/master/images/O365-Signup.jpg "Office 365 Signup")

**Figure 2:** Office 365 Signup

**3)** Once the sign-up process is complete, open to https://www.microsoftazurepass.com/ and claim the promo code to your new tenant (making sure you’re logged in as the admin user you just created)

**4)** Open http://portal.azure.com and click Azure Active Directory > Licenses > All Products > Try/buy > Free Trial under ENTERPRISE MOBILITY + SECURITY E5 > Activate

**5)** Open a Cloud Shell window using the “>_”  on the top right hand side of the screen.

**6)** Make sure the Cloud Shell window is set to “Powershell” (not “Bash”) as shown in Figure 3.

![Cloud Shell](https://github.com/araffe/vdc-security-lab/blob/master/images/cloudshell.jpg "Cloud Shell")

**Figure 3:** Azure Cloud Shell - Powershell

**7)** To create the users, copy the code below and paste into cloud shell:

<pre lang="...">
$script = Invoke-WebRequest https://raw.githubusercontent.com/Araffe/ARM-Templates/master/infra-security-lab/CreateUsers.ps1 -UseBasicParsing

Invoke-Expression $($script.Content)
 </pre>

**8)** To deploy the lab infrastructure, enter the following commands into the Powershell Cloud Shell window:

<pre lang="...">
$script = Invoke-WebRequest https://raw.githubusercontent.com/Araffe/ARM-Templates/master/infra-security-lab/CreateLab.ps1 -UseBasicParsing

Invoke-Expression $($script.Content)
 </pre>

The lab environment will deploy using an Azure ARM template – this will take approximately 10 – 15 mins.

Finally, assign directory roles and licenses to the users that have been created.

**1)** In the Azure portal, navigate to Azure Active Directory > Users > All Users > Isaiah Langer > Directory Role > Global Administrator > Save.

![Assign Role](https://github.com/araffe/vdc-security-lab/blob/master/images/assignrole.jpg "Assigin Role")

**Figure 4:** Assign Global Admin Role

**2)** Repeat this process for the user Isaiah Langer.

**3)** Navigate to Azure Active Directory > Licenses > All Products > Enterprise Mobility + Security E5 > select all of the users > Assign

**4)** Repeat the above process to assign Office 365 to the users Alex and Isaiah, your admin user should already be licensed as part of the trial sign up process.
