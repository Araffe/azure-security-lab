# Securing Azure Infrastructure - Hands on Lab Guide

**Authors:**

Adam Raffe - Microsoft

Tom Wilde - Microsoft

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

![Main Lab Image](https://github.com/Araffe/azure-security-lab/blob/master/Images/Overview.jpg "Security Lab Environment")

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

![Office 365 Signup](https://github.com/araffe/azure-security-lab/blob/master/Images/O365-Signup.jpg "Office 365 Signup")

**Figure 2:** Office 365 Signup

**3)** Once the sign-up process is complete, open to https://www.microsoftazurepass.com/ and claim the promo code to your new tenant (making sure you’re logged in as the admin user you just created)

**4)** Open http://portal.azure.com and click Azure Active Directory > Licenses > All Products > Try/buy > Free Trial under ENTERPRISE MOBILITY + SECURITY E5 > Activate

**5)** Open a Cloud Shell window using the “>_”  on the top right hand side of the screen.

**6)** Make sure the Cloud Shell window is set to “Powershell” (not “Bash”) as shown in Figure 3.

![Cloud Shell](https://github.com/araffe/azure-security-lab/blob/master/Images/cloudshell.jpg "Cloud Shell")

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

![Assign Role](https://github.com/araffe/azure-security-lab/blob/master/Images/assignroles.jpg "Assign Role")

**Figure 4:** Assign Global Admin Role

**2)** Repeat this process for the user Isaiah Langer.

**3)** Navigate to Azure Active Directory > Licenses > All Products > Enterprise Mobility + Security E5 > select all of the users > Assign

**4)** Repeat the above process to assign Office 365 to the users Alex and Isaiah, your admin user should already be licensed as part of the trial sign up process.

# Lab 1: Azure Security Center <a name="asc"></a>

In this lab, we’ll use Azure Security Center (ASC) to view recommendations and implement security policy in the Contoso environment. ASC provides centralized security policy management, actionable recommendations, alerting and incident reporting for both Azure and on-premises environments (as well other cloud platforms).

## 1.1: Enable Azure Security Center <a name="enableasc"></a>

**1)** In the Azure portal, click on Security Center on the left hand menu.

**2)** It may take a few minutes before Security Center is ready – resources will show as “refreshing” during this time. (Figure 5).

![ASC Initial Screen](https://github.com/araffe/azure-security-lab/blob/master/Images/ascoverview.jpg "ASC Initial Screen")

**Figure 5:** Initial Security Center Screen

**3)** At the top of the Security Center main pane, you will see a message stating that “Your security experience may be limited”. Click this message and you will be taken to a new pane entitled “Enable advanced security for subscriptions”. Click on your subscription and you will be taken to a new screen, as shown in Figure 6.

![ASC Pricing](https://github.com/araffe/azure-security-lab/blob/master/Images/ascoverview.jpg "ASC Pricing")

**Figure 6:** Azure Security Center Pricing

**4)** Select the “Standard” tier and then click “save”.

You have just upgraded to the “Standard” tier of Azure Security Center. This tier provides additional functionality over the free tier, including advanced threat detection and adaptive security controls. More details on the Standard tier are available from https://docs.microsoft.com/en-us/azure/security-center/security-center-pricing.

## 1.2: Explore Azure Security Center <a name="exploreasc"></a>

In this section of the lab, we’ll take a look around Azure Security Center and explore what it has to offer.

**1)** In the Azure portal, click on Security Center on the left hand menu.

**2)** The overview section of the Security Center shows an 'at-a-glance' view of any security recommendations, alerts and prevention items relating to compute, storage, networking and applications, as shown in Figure 7.

![ASC Main](https://github.com/araffe/azure-security-lab/blob/master/Images/ascmain.jpg "ASC Main")

**Figure 7:** Azure Security Center Main Screen

**3)** Click on 'Recommendations' in the Security Center menu. You will see a list of recommendations relating to various areas of the environment - for example, the need to add Network Security Groups on subnets and VMs, or the recommendation to apply disk encryption to VMs.

![ASC Recommendations](https://github.com/araffe/azure-security-lab/blob/master/Images/ascrecommendations.jpg "ASC Recommendations")

**Figure 8:** Azure Security Center Recommendations

**4)** Return to the main Security Center page and then click on Compute. This will take you to a compute specific recommendations page where we can begin to apply recommendations.

**5)** Click on the ‘VMs and Computers’ tab where you will see a list of all VMs in your subscription and the issues that ASC has found.

**6)** One of the common warnings is related to endpoint protection on virtual machines. Return to the ‘Overview’ tab and then click on the warning for ‘Endpoint Protection Issues’. This will take you to a screen showing how many VMs are not protected.

![ASC Endpoint Protection](https://github.com/araffe/azure-security-lab/blob/master/Images/ascendpoint.jpg "ASC Endpoint Protection")

**Figure 9:** Azure Security Center Endpoint Protection

**7)** Click on the ‘Endpoint Protection Not Installed’ item and then select the eligible VMs (VM1 & VM2 in your case). Click the button ‘Install on 2 VMs’.

**8)** Select ‘Microsoft Anti-Malware’ and then select all defaults before clicking ‘OK’ and letting the anti-malware software install on your VMs.

**9)** Return to the ‘Overview’ page within the Compute section and click on ‘Add a vulnerability assessment solution’. Select all four virtual machines and then click ‘Install’. From here, you can install a 3rd party vulnerability assessment tool (Qualys) on your VMs. Do not proceed with the installation, but instead proceed to the next step.

**10)** Return to the main ASC screen and then click on Networking. From here, you’ll be able to see that your VMs (VM1 – 4) are listed as ‘Internet Facing Endpoints’ but have no protection from either Network Security Groups or Next Generation Firewalls (Figure 10). You’ll add Network Security Groups to the environment later.

![ASC Networking](https://github.com/araffe/azure-security-lab/blob/master/Images/ascnetworking.jpg "ASC Networking")

**Figure 10:** Azure Security Center Networking Recommendations

**11)** From the main ASC page, click on Security Policy on the left hand menu. Click on your subscription.

**12)** From here, you can control the security policy recommendations (in the security policy section), set up email addresses for automated alerting and configure the pricing tier.

**13)** From the ‘Data Collection’ page, turn on the automatic provisioning of the monitoring agent and click save. This will allow Azure Security Center to automatically install the monitoring agent on the VMs in your subscription.


