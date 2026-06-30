# Connecting Windows VMs to Microsoft Sentinel using distinct Data Connectors
                                 

## Introduction

In my most previous lab project, I connected Microsoft Defender for Cloud and Azure Activity data using data connectors. This lab builds on top of the data collection by connecting **Windows devices** to Microsoft Sentinel using data connectors. Specifically, this lab focuses on **Windows virtual machines inside and outside of Azure, like on-premises environments or other Public Clouds** as data sources. It goes without saying that Security analysts and Engineers working in a Microsoft stack environment in a Security Operations Center must understand different requirements and the specific considerations for connecting log data from the many data sources in an organization.

## Connect Data to Microsoft Sentinel using Data Connectors

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%201.png)

Figure 1: Architectural design

The Architectural diagram depicts the design of three tasks that my lab wants to operationalize. Based on the diagram, Task 1 involves creating a Windows VM (AZWIN01) in a resource group (rg-AZWIN01). Task 2 connects the Azure Windows VM to Microsoft Sentinel via Windows Security Events using AMA (AZWINDCR) and a Log Analytics Workspace. Task 3 connects a non-Azure Windows Machine (WINServer) using Windows AMA/Azure Arc to the same Sentinel instance. 
 

## Objective 1

## Create a Windows Virtual Machine in Azure**

## Step 1

*   The first step is to head to Microsoft Azure portal and create a virtual machine in the portal https://portal.azure.com after successful sign-in into the portal.

➢ I selected **+ Create a Resource**


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%202.png)

Figuure 2: Screenshot of Microsoft Azure Home portal showing Azure services, recent resources, navigation links, and tools. A red box highlights the "Create a resource" button.

➢ In the **Search services and marketplace** box, entered **Windows 11** and selected **Windows 11** from the drop-down list.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%203.png)

Figure 3: Screenshot of Microsoft Azure "Create a resource" page showing a search for "Windows 11" in the marketplace.

Ø In the output page, I selected Windows 11

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%204.png)

Figure 4: Screenshot of Microsoft Azure Marketplace search results for "windows 11" with various virtual machine options displayed in a tile view.

➢ I opened the Subscription drop-down list and selected **MOC Subscription-lod52789002**


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%205.png)

Figure 5: Screenshot of Microsoft Azure portal showing Windows 11 virtual machine creation page with Subscription and Plan drop-down menus.

➢ I opened the *Plan* drop-down list and selected **Windows 11 Enterprise, version 24H2**

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%206.png)

Figure 6: Screenshot of Microsoft Azure portal showing Windows 11 virtual machine creation options with a dropdown menu for plans.

Ø I selected **Start with a pre-set configuration** to continue


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%207.png)

Figure 7: Screenshot of Microsoft Azure portal showing the "Start with a pre-set configuration" button being selected for a Windows 11 virtual machine.

➢ I selected **Dev/Test** and then select Continue to create a VM


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%208.png)

Figure 8: Screenshot of Microsoft Azure portal showing "Choose recommended defaults that match your workload" page for Windows 11 VM creation.

➢ The following step required me to create Azure resources, which I will create with unique names.

* I selected **MOC Subscription-lod52732281** from the Subscription drop-down.

* I selected **ResourceGroup1** for **Resource group** and selected **OK**.

* Under the **Instance details** section, I entered AZWIN-62636865 as the **Virtual Machine name**

* I left the **(US) East US** unchanged as the default value for Region.

* I cross-checked the image of the virtual machine to ensure that **Windows 11 Enterprise, version 24H2** appears on the image field.
  

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%209.png)

Figure 9: Screenshot of Microsoft Azure portal showing the "Create a virtual machine" page with various configuration options like Subscription, Resource group, Virtual machine name, Region, and Image selection. Red arrows point to specific fields.

* I scrolled down and reviewed the Size for the virtual machine and selected **Standard_D2s_v3**. I needed to need to click "See all sizes" to choose Standard_D2s_v3.

* I entered a *Username*, and a password of my choosing and confirmed the password

* I scrolled down to the bottom of the page and selected the checkbox below Licensing to confirm I have the eligible license.

* I selected **Review + create** and waited until the validation is passed.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2010.png)

Figure 10: Microsoft Azure portal screenshot showing virtual machine creation basics


* I reviewed the confirmation of the validation being passed, the pricing for the VM creation, and I selected **Create**
  

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2011.png)

Figure 11: Microsoft Azure portal screenshot showing virtual machine review and create screen


* I got a confirmation that “Deployment is in progress” and “Your Deployment is complete”
* The deployment details list several resources including a virtual machine, network interface, virtual network, network security group, and public IP address, all with a status of "Created" or "OK".

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2012.png)

Figure 12: Screenshot of Microsoft Azure portal showing a successful deployment of a Windows 11 virtual machine. 

* It only took a few minutes to be notified that “Your Deployment is complete” and Azure VM resource was successfully created.
  

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2013.png)

Figure 13: Screenshot of Microsoft Azure portal showing a successful VM deployment.

## Objective 2

## Connecting an On-Premises Server to Azure

The next objective is to connect an on-premises server to your Azure subscription. Azure Arc has been pre-installed on this server, therefore, I was no longer required to re-install that before trigger the connection between my on-premises server and the Azure subscription. For purpose of clarity, it is important to denote that Azure Arc uses the <mark>Azure Connected Machine agent (azcmagent)</mark> installed on the device or VM that allows the device to be managed the same as an Azure VM. Azure Arc provides other functionality including running Azure based services in a hybrid environment.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2014.png)

Figure 14: Screenshot of Windows Server Manager Dashboard on a VM named WINServer

Step 1

* I opened a different VM machine named WINServer as an Administrator

* On the WINServer machine, I selected the search icon and typed <mark>cmd</mark>

* In search results right I clicked <mark>Command Prompt</mark> and selected <mark>Run as administrator</mark>.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2015.png)

Figure 15: Screenshot of Windows Server Manager with the Start menu open searching for "cmd" and highlighting "Command Prompt"


* In the **Command Prompt** window, I typed the following command and pressed "Enter" to run the command

```bash
azcmagent connect -g <mark>"ResourceGroup1"</mark> -l <mark>"CentralUS"</mark> -s <mark>"31acf289-9ebf-4db5-85bd-97b5e387e2d8"</mark>
```

* A browser selection window popped up "How do you want to open this?"

* I selected Microsoft Edge and "OK"

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2016.png)

Figure 16: Screenshot of Windows Server Manager with a Command Prompt window showing the azcmagent connect command and a "How do you want to open this?" browser selection dialog.

* I was provided with a browser.

* In the Sign in dialog box, I entered my <mark>Tenant Email</mark> and <mark>Tenant Password</mark> and selected Sign in.

* I waited for the <mark>Authentication complete</mark> message, closed the browser tab and returned to the Command Prompt window.

* The output showed that my on-premises server has been connected to Azure

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2017.png)

Figure 17: Screenshot of a Windows Command Prompt showing the successful execution of the azcmagent connect command, with a red arrow pointing to the "Connected machine to Azure" info message.

* I typed in the following command to verify that Agent status is Connected:

`azcmagent show`

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2018.png)

Figure 18: Screenshot of a Windows Command Prompt showing the output of the azcmagent connect command, indicating a successful connection of a machine named WINServer to Azure Arc.


## Objective 3

## Connect an Azure Windows virtual machine

In Objective 3, I decided to connect an Azure Windows virtual machine to Microsoft Sentinel.

It must be noted that Microsoft Sentinel has been pre-deployed in my Azure subscription with the name sentinelworkspace-01, and the required Content Hub solutions have been installed

### Step 1

* I opened a Microsoft Edge browser, navigated to Defender XDR at https://security.microsoft.com, and signed in with my credentials.
  

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2019.png)

Figure 19: Screenshot of Microsoft Defender portal showing the Home dashboard with Microsoft Sentinel sections expanded in the navigation menu.

* In the Microsoft Defender navigation menu, I scrolled down and expanded the **Microsoft Sentinel** section

* I expanded the Content management section and select Content Hub.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2020.png)

Figure 20: Screenshot of Microsoft Defender Content hub showing various installed solutions like Amazon Web Services, Azure Activity, and Cisco Cloud Security.

* In the Content hub, I searched for the <mark>Windows Security Events</mark> solution and selected it from the list. Although I had installed it beforehand before my lab demonstration, but I had to demonstrate my steps because of its significance.

* On the <mark>Windows Security Events</mark> solution page I selected **Manage**.

* The Windows Security Events solution installs both the Windows Security Events via AMA and the Security Events via Legacy Agent Data connectors. Plus 2 Workbooks, 20 Analytic Rules, and 43 Hunting Queries

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2021.png)

Figure 21: Screenshot of Microsoft Sentinel Content hub showing Windows Security Events solution selected.

> I selected the **Windows Security Events via AMA** Data connector, and selected **Open connector page** on the connector information blade.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2022.png)

Figure 22: Screenshot of Microsoft Sentinel Content hub showing the Windows Security Events solution details and the "Open connector page" button highlighted.

* On the Setup tab, scrolled down to the Configuration section, and selected the **+Create data collection rule** button.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2023.png)

Figure 23: Screenshot of Microsoft Defender portal showing the Windows Security Events via AMA connector details page with a list of data collection rules.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2024.png)

Figure 24: Screenshot of Microsoft Defender portal showing the bottom of the Windows Security Events via AMA setup page with the "+Create data collection rule" button highlighted.

* For the Rule name enter: **AZWIN62636865DCR**, then select Next: Resources.

* I Expand your **MOC Subscription** under Scope on the Resources tab.

* I expanded ResourceGroup1, then select **AZWIN-62636865**.

* I selected **Next: Collect**.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2025.png)

Figure 25: Screenshot of Microsoft Defender portal showing the Data Collection Rule (DCR) Wizard on the Resources tab. The user has selected a virtual machine named AZWIN-62636865 within ResourceGroup1 under the MOC Subscription.


* I reviewed the different Security Event collection option. Keep All Security Events and then select <mark>Next: Review + create</mark>.

* Confirmation of “Validation passed” popped up on the top corner of the information blade at the right side

* Selected **Create** to save the Data Collection Rule.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2026.png)

Figure 26: Screenshot of Microsoft Defender portal showing the DCR Wizard at the "Review + create" step with validation passed.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2027.png)

Figure 27: Screenshot of Microsoft Defender portal showing a notification "Create Data Collection Rule Succeeded" after clicking create in the DCR Wizard.

➢ "Create Data Collection Rule succeeded" appeared on the top corner


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2028.png)

Figure 28: Screenshot of Microsoft Defender for Cloud portal showing a list of Data Collection Rules (DCRs) in Microsoft Sentinel. The rule AZWIN62636865DCR is highlighted with a red box.

* I waited a minute and then selected **Refresh** to see the new data collection rule listed.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2029.png)

Figure 29: Screenshot of Microsoft Defender portal showing the DCR Wizard with ResourceGroup1 expanded and WINServer selected.

## Objective 4

# Connect a non-Azure Windows Machine

In objective 4, I added an Azure Arc connected, non-Azure Windows virtual machine to Microsoft Sentinel. As a quick reminder, the <mark>Windows Security Events via AMA</mark> data connector requires <mark>Azure Arc</mark> for non-Azure devices.

## Step 1

* Just right in the Windows Security Events via AMA data connector configuration in your Microsoft Sentinel workspace, where I initially connected an Azure Windows virtual machine, I edited the DCR wizard by navigating to the Configuration section and editing the <mark>AZWIN62636865DCR</mark> data collection rule by selecting the pencil icon.

* I selected Next: **Resources**, and expanded my MOC Subscription under **Scope** on the **Resources** tab.

* I expanded <mark>ResourceGroup1</mark>, then I selected <mark>WINServer</mark>.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2030.png)

Figure 30: Screenshot of Microsoft Defender portal showing the DCR Wizard at the Collect step with "All Security Events" selected and the "Next: Review + create" button highlighted.

* I selected <mark>**Next: Collect**</mark>, then <mark>Next: Review + create</mark>.


![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2031.png)

Figure 31: Screenshot of Microsoft Defender Sentinel Data Collection Rule (DCR) Wizard showing "Validation passed" and "Create Data Collection Rule Succeeded" notification.

➢ I selected **Create** after <mark>Validation passed</mark> is displayed.

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2032.png)

Figure 32: Screenshot of scuccessful "Validation passed"

![image alt](https://github.com/Michaelsalaja/SecOps-Engineering-Lab-2/blob/bfdc69eb3937349cc6d156b8f20caf230db525f9/Figure%2033.png)

Figure 33: Successful creation of Data Collection Rule was displayed.


## Summary

In this lab, I executed the following task for the log ingestion as a Security Engineer / Analyst working for a company that implemented Microsoft Sentinel:

* I created a Windows Virtual Machine in Azure named as AZWIN-62636865.

* I connected an On-premises Server to my Azure subscription by navigating to another virtual machine and running various commands on a Command Prompt as an administrator and verified through the agent status that the machine is integrated to my Azure subscription. Azure Arc had been preinstalled

* I connected an Azure Windows Virtual Machines to Microsoft Sentinel workspace on Microsoft Defender XDR.

* I configured Windows hosts to send event data to Microsoft Sentinel and ensured that Sysmon events are available to be used in detection rules

* Out of the two Windows security events Content Hub solution options to stream events from Windows devices to Microsoft Sentinel, I chose Windows Security Events via AMA Connector over Security Events via Legacy Agent Connector because the Legacy connector uses the Log Analytics agent, which was deprecated on Aug 31, 2024, and thus should only be installed where AMA isn't supported, as recommended by Microsoft.

* I added an Azure Arc-enabled non-Azure Windows virtual machine to Microsoft Sentinel to start collecting security-related events. Azure Arc was added because Azure Arc-enabled servers lets you manage Windows and Linux physical servers and virtual machines hosted outside of Azure, on your corporate

## Follow-up

My next lab will concentrate on connecting Linux hosts to Microsoft Sentinel using the Common Event Format (CEF) via Azure Monito Agent (AMA) and Syslog via Syslog via Legacy Agent data connectors.
