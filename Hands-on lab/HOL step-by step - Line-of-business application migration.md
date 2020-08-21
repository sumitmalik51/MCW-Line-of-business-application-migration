![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Line-of-business application migration
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
June 2020
</div>


Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2020 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents** 

<!-- TOC -->

- [Line-of-business application migration hands-on lab step-by-step](#line-of-business-application-migration-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
  - [Exercise 1: Discover and assess the on-premises environment](#exercise-1-discover-and-assess-the-on-premises-environment)
    - [Task 1: Create the Azure Migrate project and add assessment and migration tools](#task-1-create-the-azure-migrate-project-and-add-assessment-and-migration-tools)
    - [Task 2: Deploy the Azure Migrate appliance](#task-2-deploy-the-azure-migrate-appliance)
    - [Task 3: Configure the Azure Migrate appliance](#task-3-configure-the-azure-migrate-appliance)
    - [Task 4: Create a migration assessment](#task-4-create-a-migration-assessment)
    - [Task 5: Configure dependency visualization](#task-5-configure-dependency-visualization)
    - [Task 6: Explore dependency visualization](#task-6-explore-dependency-visualization)
  - [Exercise 2: Migrate the Application Database](#exercise-2-migrate-the-application-database)
    - [Task 1: Register the Microsoft.DataMigration resource provider](#task-1-register-the-microsoftdatamigration-resource-provider)
    - [Task 2: Create an Azure SQL Database](#task-2-create-an-azure-sql-database)
    - [Task 3: Create the Database Migration Service](#task-3-create-the-database-migration-service)
    - [Task 4: Assess the on-premises database using Data Migration Assistant](#task-4-assess-the-on-premises-database-using-data-migration-assistant)
    - [Task 5: Create a DMS migration project](#task-5-create-a-dms-migration-project)
    - [Task 6: Migrate the database schema](#task-6-migrate-the-database-schema)
    - [Task 7: Migrate the on-premises data](#task-7-migrate-the-on-premises-data)
  - [Exercise 3: Migrate the application and web tiers using Azure Migrate: Server Migration](#exercise-3-migrate-the-application-and-web-tiers-using-azure-migrate-server-migration)
    - [Task 1: Create a Storage Account](#task-1-create-a-storage-account)
    - [Task 2: Create a Virtual Network](#task-2-create-a-virtual-network)
    - [Task 3: Register the Hyper-V Host with Azure Migrate Server Migration](#task-3-register-the-hyper-v-host-with-azure-migrate-server-migration)
    - [Task 4: Enable Replication from Hyper-V to Azure Migrate](#task-4-enable-replication-from-hyper-v-to-azure-migrate)
    - [Task 5: Configure static internal IP addresses for each VM](#task-5-configure-static-internal-ip-addresses-for-each-vm)
    - [Task 6: Server migration](#task-6-server-migration)
    - [Task 7: Enable Azure Bastion](#task-7-enable-azure-bastion)
    - [Task 8: Configure the database connection](#task-8-configure-the-database-connection)
    - [Task 9: Configure the public IP address and test the SmartHotel application](#task-9-configure-the-public-ip-address-and-test-the-smarthotel-application)
    - [Task 10: Post-migration steps](#task-10-post-migration-steps)
  - [After the hands-on lab](#after-the-hands-on-lab)
    - [Task 1: Clean up resources](#task-1-clean-up-resources)

<!-- /TOC -->

# Line-of-business application migration hands-on lab step-by-step 

## Abstract and learning objectives 

In this hands-on lab, you will learn how to assess and migrate a multi-tier application from Hyper-V to Azure. You will learn how to use Azure Migrate as the hub for executing a migration, together with accompanying tools.

After this hands-on lab, you will know the role of Azure Migrate and related migration tools and how to use them to successfully migrate an on-premises multi-tier application to Azure.

## Overview

Before the lab, you will have pre-deployed an on-premises infrastructure hosted in Hyper-V.  This infrastructure is hosting a multi-tier application called 'SmartHotel', using Hyper-V VMs for each of the application tiers.

During the lab, you will migrate this entire application stack to Azure. This will include assessing the on-premises application using Azure Migrate; assessing the database migration using SQL Server Data Migration Assistant (DMA); migrating the database using the Azure Database Migration Service (DMS); and migrating the web and application tiers using Azure Migrate: Server Migration. This last step includes migration of both Windows and Linux VMs.

## Solution architecture

The SmartHotel application comprises 4 VMs hosted in Hyper-V:

- **Database tier** Hosted on the smarthotelSQL1 VM, which is running Windows Server 2016 and SQL Server 2017

- **Application tier** Hosted on the smarthotelweb2 VM, which is running Windows Server 2012R2.

- **Web tier** Hosted on the smarthotelweb1 VM, which is running Windows Server 2012R2.

- **Web proxy** Hosted on the  UbuntuWAF VM, which is running Nginx on Ubuntu 18.04 LTS.

For simplicity, there is no redundancy in any of the tiers.

>**Note:** For convenience, the Hyper-V host itself is deployed as an Azure VM. For the purposes of the lab, you should think of it as an on-premises machine.

![A slide shows the on-premises SmartHotel application architecture. This comprises a SmartHotelHost server running Microsoft Hyper-V. This server hosts 4 VMs: UbuntuWAF, SmartHotelWeb1, SmartHotelWeb2, and SmartHotelSQL1. A series of arrows show how these VMs will be migrated to Azure. The first 3 VMs have an arrow labeled 'Azure Migrate: Server Migration' pointing to 3 similarly-labeled VMs in Azure. The last VM, SmartHotelSQL1, has an arrow labeled 'Azure Database Migration Service' pointing to an Azure SQL Database. A third arrow labeled 'Azure Migrate: Server Assessment' and 'Data Migration Assistant (DMA)' points from all 4 on-premises VMs to an Azure Migrate dashboard showing migration readiness.](images/overview.png)

Throughout this lab, you will use Azure Migrate as your primary tool for assessment and migration. In conjunction with Azure Migrate, you will also use a range of other tools, as detailed below.

To assess the Hyper-V environment, you will use Azure Migrate: Server Assessment. This includes deploying the Azure Migrate appliance on the Hyper-V host to gather information about the environment. For deeper analysis, the Microsoft Monitoring Agent and Dependency Agent will be installed on the VMs, enabling the Azure Migrate dependency visualization.

The SQL Server database will be assessed by installing the Microsoft SQL Server Data Migration Assistant (DMA) on the Hyper-V host, and using it to gather information about the database. Schema migration and data migration will then be completed using the Azure Database Migration Service (DMS).

The application, web, and web proxy tiers will be migrated to Azure VMs using Azure Migrate: Server Migration. You will walk through the steps of building the Azure environment, replicating data to Azure, customizing VM settings, and performing a failover to migrate the application to Azure.

> **Note**: After migration, the application could be modernized to use Azure Application Gateway instead of the Ubuntu Nginx VM, and to use Azure App Service to host both the web tier and application tier. These optimizations are out of scope of this lab, which is focused only on a 'lift and shift' migration to Azure VMs.

## Requirements

1. You will need Owner or Contributor permissions for an Azure subscription to use in the lab.

2. Your subscription must have sufficient unused quota to deploy the VMs used in this lab.

## Before the hands-on lab

Refer to the [Before the HOL - Line-of-business application migration](./Before%20the%20HOL%20-%20Line-of-business%20application%20migration.md) setup guide manual before continuing to the lab exercises.

**IMPORTANT: Allow at least 60 minutes to deploy the on-premises environment before you start the lab.**

## Exercise 1: Discover and assess the on-premises environment

Duration: 60 minutes

In this exercise, you will use Azure Migrate: Server Assessment to assess the on-premises environment. This will include selecting Azure Migrate tools, deploying the Azure Migrate appliance into the on-premises environment, creating a migration assessment, and using the Azure Migrate dependency visualization.

### Task 1: Create the Azure Migrate project and add assessment and migration tools

In this task, you will create the Azure Migrate project and select the assessment and migration tools.

> **Note**: In this lab, you will use the Microsoft-provided assessment and migration tools within Azure Migrate. A number of third-party tools are also integrated with Azure Migrate for both assessment and migration. You may wish to spend some time exploring these third-party options outside of this lab.

1. Open your browser, navigate to **https://portal.azure.com**, and log in with your Azure subscription credentials.

2. Select **All services** in the portal's left navigation, then search for and select **Azure Migrate** to open the Azure Migrate Overview blade, shown below.

    ![Screenshot of the Azure Migrate overview blade.](images/Exercise1/azure-migrate-overview.png)

3. Select **Assess and migrate servers**, then **Add tool(s)**, to open the **Add a tool** wizard at the **Migrate project** section.  Select your subscription and create a new resource group named **AzureMigrateRG**. Enter **SmartHotelMigration** as the Migrate project name, and choose a geography close to you to store the migration assessment data. Select **Next**.

    >**Note**: If you are running this lab in a shared subscription you will need to use a migrate project name that is unique in the subscription. Append characters to the end of migrate project name to make your project name unique. For example: **SmartHotelMigration1234**.

    ![Screenshot of the Azure Migrate 'Add a tool' wizard, at the 'Migrate project' step.](images/Exercise1/add-tool-1.png)

4. At the **Select assessment tool** section, select **Azure Migrate: Server Assessment**, then select **Next**.

    ![Screenshot of the Azure Migrate 'Add a tool' wizard, at the 'Select assessment tool' step.](images/Exercise1/add-tool-2.png)

5. At the **Select migration tool** step, select **Azure Migrate: Server Migration**, then select **Next**.

    ![Screenshot of the Azure Migrate 'Add a tool' wizard, at the 'Select migration tool' step.](images/Exercise1/add-tool-3.png)

6. At the **Review + add tool(s)** section, review the settings and select **Add tool(s)**.

    ![Screenshot of the Azure Migrate 'Add a tool' wizard, at the 'Review + Add tool(s)' step.](images/Exercise1/add-tool-4.png)

7. The Azure Migrate deployment will start. Once it has completed, select the **Servers** panel of the Azure Migrate blade (it may open automatically). You should now see the **Azure Migrate: Server Assessment** and **Azure Migrate: Server Migration** panels for the current migration project, as shown below.

    ![Screenshot of the Azure Migrate 'Servers' blade, showing the Azure Migrate: Server Assessment tool for the selected subscription and resource group.](images/Exercise1/servers.png)

#### Task summary 

In this task you created an Azure Migrate project, selecting the built-in tools for assessment and migration.

### Task 2: Deploy the Azure Migrate appliance

In this task, you will deploy and configure the Azure Migrate appliance in the on-premises Hyper-V environment. This appliance communicates with the Hyper-V server to gather configuration and performance data about your on-premises VMs.

1. Select **Discover** under **Azure Migrate: Server Assessment** to open the **Discover machines** blade on the right. Under **Are your machines virtualized?**, select **Yes, with Hyper-V**.

    ![Screenshot of the Azure Migrate 'Discover machines' blade, with Hyper-V selected.](images/Exercise1/discover-machines.png)

    Read through the instructions on how to download, deploy and configure the Azure Migrate appliance. Close the 'Discover machines' blade (do **not** download the .VHD file, it has already been downloaded for you).

2. In a separate browser tab, navigate to the Azure portal. In the global search box, enter **SmartHotelHost**, then select the **SmartHotelHost** virtual machine.

    ![Screenshot of the Azure portal search box, searching for the SmartHotelHost virtual machine.](images/Exercise1/find-smarthotelhost.png)

3. Select **Connect**, select **RDP**, then download the RDP file and connect to the virtual machine using username **demouser** and password **demo!pass123**.

4. In Server Manager, select **Tools**, then **Hyper-V Manager** (if Server Manager does not open automatically, open it by selecting **Start**, then **Server Manager**). In Hyper-V manager, select **SMARTHOTELHOST**. You should now see a list of the four VMs that comprise the on-premises SmartHotel application.

    ![Screenshot of Hyper-V Manager on the SmartHotelHost, showing 4 VMs: smarthotelSQL1, smarthotelweb1, smarthotelweb2 and UbuntuWAF.](images/Exercise1/hyperv-vm-list.png)

    You will now deploy the Azure Migrate appliance virtual machine.  Normally, you would first need to download the .ZIP file containing the appliance to your Hyper-V host, and unzip it. To save time, these steps have been completed for you.

5. In Hyper-V Manager, under **Actions**, select **Import Virtual Machine...** to open the **Import Virtual Machine** wizard.

    ![Screenshot of Hyper-V Manager, with the 'Import Virtual Machine' action highlighted.](images/Exercise1/import-vm-1.png)

6. At the first step, **Before You Begin**, select **Next**.

7. At the **Locate Folder** step, select **Browse** and navigate to **F:\\VirtualMachines\\AzureMigrateAppliance** (the folder name may also include a version number), then choose **Select Folder**, then select **Next**.

    ![Screenshot of the Hyper-V 'Import Virtual Machine' wizard with the F:\VirtualMachines\AzureMigrateAppliance folder selected.](images/Exercise1/import-vm-2.png)

8. At the **Select Virtual Machine** step, the **AzureMigrateAppliance** VM should already be selected. Select **Next**.

9. At the **Choose Import Type** step, keep the default setting **Register the virtual machine in-place**. Select **Next**.

10. At the **Connect Network** step, you will see an error that the virtual switch previously used by the Azure Migrate appliance could not be found. From the **Connection** drop down, select the **Azure Migrate Switch**, then select **Next**.

    ![Screenshot of the Hyper-V 'Import Virtual Machine' wizard at the 'Connect Network' step. The 'Azure Migrate Switch' has been selected.](images/Exercise1/import-vm-4.png)

    > **Note**:  The Azure Migrate appliance needs access to the Internet to upload data to Azure. It also needs access to the Hyper-V host. However, it does not need direct access to the application VMs running on the Hyper-V host. To protect the application environment, the Azure Migrate Appliance should be deployed to a separate subnet within Hyper-V, rather than in the same subnet as your application. 
    >
    > The Hyper-V environment has a NAT network using the IP address space 192.168.0.0/16. The internal NAT switch used by the SmartHotel application uses the subnet 192.168.0.0/24, and each VM in the application has been assigned a static IP address from this subnet.
    >
    > The Azure Migrate Appliance will be connected to a separate subnet 192.168.1.0/24, which has been set up for you. Using the 'Azure Migrate Switch' connects the appliance to this subnet. The appliance is assigned an IP address from this subnet using a DHCP service running on the SmartHotelHost.

11. Review the summary page, then select **Finish** to create the Azure Migrate appliance VM.

12. In Hyper-V Manager, select the **AzureMigrateAppliance** VM, then select **Start** on the left.

   ![Screenshot of Hyper-V Manager showing the start button for the Azure Migrate appliance.](images/Exercise1/start-migrate-appliance.png)

#### Task summary 

In this task you deployed the Azure Migrate appliance in the on-premises Hyper-V environment.

### Task 3: Configure the Azure Migrate appliance

In this task, you will configure the Azure Migrate appliance and use it to complete the discovery phase of the migration assessment.

1. In Hyper-V Manager, select the **AzureMigrateAppliance** VM, then select **Connect** on the left.

    ![Screenshot of Hyper-V Manager showing the connect button for the Azure Migrate appliance.](images/Exercise1/connect-appliance.png)

2. A new window will open showing the Azure Migrate appliance. Wait for the License terms screen to show, then select **Accept**.

    ![Screenshot of the Azure Migrate appliance showing the license terms.](images/Exercise1/license-terms.png)

3. On the **Customize settings** screen, set the Administrator password to **demo!pass123** (twice). Then select **Finish**.

    > **Note**: When entering the password, the VM uses a US keyboard mapping. If you are using a non-US keyboard, some characters may be entered incorrectly. Select the 'eyeball' icon in the second password entry box to check the password has been entered correctly.

    ![Screenshot of the Azure Migrate appliance showing the set Administrator password prompt.](images/Exercise1/customize-settings.png)

4. At the **Connect to AzureMigrateAppliance** prompt, set the appliance screen size using the slider, then select **Connect**.

5. Log in with the Administrator password **demo!pass123** (the login screen may pick up your local keyboard mapping, use the 'eyeball' icon to check).

6. **Wait.** After a minute or two, an Internet Explorer windows will open showing the Azure Migrate appliance configuration wizard. If the **Set up Internet Explorer 11** prompt is shown, select **OK** to accept the recommended settings. If the Internet Explorer **Content from the website listed below is being blocked...** prompt is shown, select **Close** and return to the Azure Migrate Appliance browser tab.

    ![Screenshot of the opening step of the Azure Migrate appliance configuration wizard.](images/Exercise1/appliance-config-1.png)

7. Under **Set up prerequisites**, accept the license terms. The following two steps to verify Internet connectivity and time synchronization should pass automatically.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the first step 'Set up prerequisites' in progress. The license terms, internet connectivity, and time sync steps have been completed, and the Azure Migrate updates are in progress.](images/Exercise1/appliance-config-2.png)

8. **Wait** while the wizard installs the latest Azure Migrate updates. If prompted for credentials, enter user name **Administrator** and password **demo!pass123**. Once the Azure Migrate updates are completed, check at the top of the browser window to see if a management app restart is required, and if so, select the link to restart the app. 

   ![Screenshot of the Azure Migrate appliance configuration wizard, showing the prompt to restart the management app after installing updates.](images/Exercise1/appliance-config-3a.png)

   Once restarted, repeat the steps above to complete the 'Set up prerequisites' phase of the Azure Migrate wizard. Select **Continue** to proceed.

   ![Screenshot of the Azure Migrate appliance configuration wizard, with the 'Set up prerequisites' phase completed and the 'Continue' button highlighted.](images/Exercise1/appliance-config-3b.png)

9. At the next phase of the wizard, **Register with Azure Migrate**, select **Login**. This opens a separate browser tab where you enter your Azure subscription credentials.  

10. Once you have logged in, return to the Azure Migrate Appliance tab and select your subscription and the **SmartHotelRegistration(AzureMigrateRG)** Migrate project using the drop-downs. Enter **SmartHotelAppl** as the appliance name, then select **Register**. After a short pause, the registration should be successful. Select **Continue**.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the subscription registration step.](images/Exercise1/appliance-config-4.png)

11. In the next step, **Provide Hyper-V hosts details**, enter the user name **demouser** and password **demo!pass123**. These are the credentials for the Hyper-V host. Enter **Host login** as the friendly name, then select **Save details**.

    > **Note**: The Azure Migrate appliance may not have picked up your local keyboard mapping. Select the 'eyeball' in the password box to check the password was entered correctly.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the Hyper-V host credentials.](images/Exercise1/appliance-config-5.png)

    The next step is to register the Hyper-V host with the Azure Migrate appliance.

12. Under **Specify the list of Hyper-V hosts and clusters to discover**, select **Add**.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the button to add Hyper-V hosts.](images/Exercise1/appliance-config-6.png)

13. A window will appear prompting for a list of Hyper-V hosts. Enter the Hyper-V hostname, **SmartHotelHost**. Then select **Validate**.

    > **Note**: The Hyper-V host must be specified using a hostname, and that hostname must resolve to the IP address of the host. The Hyper-V host cannot be specified using an IP address directly.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the button to add Hyper-V hosts.](images/Exercise1/appliance-config-7.png)

14. Below, a table shows the SmartHotelHost, with status 'green'. Select **Validate** again, and check that the status stays green. Then select **Save and start discovery**.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing the Hyper-V host has been added, with the 'Save and start discovery' button enabled.](images/Exercise1/appliance-config-8.png)

15. A message **Initiating discovery and configuring appliance** will appear.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing a progress ticker labelled 'Creating site ant initiating discovery'.](images/Exercise1/appliance-config-9a.png)

16. Wait for the Azure Migrate status to show **Initiated discovery and completed appliance configuration**. This will take several minutes.

    ![Screenshot of the Azure Migrate appliance configuration wizard, showing a green check mark labelled 'Created Site and initiating discovery'.](images/Exercise1/appliance-config-9b.png)

17. Return to the **Azure Migrate** blade in the Azure portal.  Select **Servers**, then select **Refresh**.  Under **Azure Migrate: Server Assessment** you should see a count of the number of servers discovered so far. If discovery is still in progress, select **Refresh** periodically until 5 discovered servers are shown. This may take several minutes.

    ![Screenshot of the Azure Migrate portal blade. Under 'Azure Migrate: Server Assessment' the value for 'discovered servers' is '5'.](images/Exercise1/discovered-servers.png)

>**Wait for the discovery process to complete before proceeding to the next Task**.

#### Task summary 

In this task you configured the Azure Migrate appliance in the on-premises Hyper-V environment and started the migration assessment discovery process.

### Task 4: Create a migration assessment

In this task, you will use Azure Migrate to create a migration assessment for the SmartHotel application, using the data gathered during the discovery phase.

1. Continuing from Task 3, select **Assess** under **Azure Migrate: Server Assessment** to start a new migration assessment.

    ![Screenshot of the Azure Migrate portal blade, with the '+Assess' button highlighted.](images/Exercise1/start-assess.png)

2. On the Assess servers blade, enter **SmartHotelAssessment** as the assessment name.

    ![Screenshot of the Azure Migrate 'Assess servers' blade, showing the assessment name.](images/Exercise1/assess-servers-1.png)

3. Under **Assessment properties**, select **View all**.

    ![Screenshot of the Azure Migrate 'Assess servers' blade, with the 'view all' assessment properties link highlighted.](images/Exercise1/assess-servers-2.png)

4. The **Assessment properties** blade allows you to tailor many of the settings used when making a migration assessment report. Take a few moments to explore the wide range of assessment properties. Hover over the information icons to see more details on each setting. Choose any settings you like, then select **Save**. (You have to make a change for the Save button to be enabled; if you don't want to make any changes, just close the blade.)

    ![Screenshot of the Azure Migrate 'Assessment properties' blade, showing a wide range of migration assessment settings.](images/Exercise1/assessment-properties.png)

5. On the Assess servers blade, under **Select or create a group**, choose **Create New** and enter the group name **SmartHotel VMs**. Select the **smarthotelweb1**, **smarthotelweb2** and **UbuntuWAF** VMs. Select **Create assessment**.

    ![Screenshot of the Azure Migrate 'Assess servers' page. A new server group containing servers smarthotelweb1, smarthotelweb2, and UbuntuWAF.](images/Exercise1/assessment-vms.png)

6. On the **Azure Migrate - Servers** blade, select **Refresh** periodically until the number of assessments shown is **1**. This may take several minutes.

    ![Screenshot from Azure Migrate showing the number of assessments as '1'.](images/Exercise1/assessments-refresh.png)

7. Select **Assessments** under **Azure Migrate: Server Assessment** to see a list of assessments. Then select the actual assessment.

    ![Screenshot showing a list of Azure Migrate assessments. There is only one assessment in the list. It has been highlighted.](images/Exercise1/assessment-list.png)

8. Take a moment to study the assessment overview.

    ![Screenshot showing an Azure Migrate assessment overview for the SmartHotel application.](images/Exercise1/assessment-overview.png)

9. Select **Edit properties**. Note how you can now modify the assessment properties you chose earlier. Change a selection of settings, and **Save** your changes. After a few moments, the assessment report will update to reflect your changes.

10. Select **Azure readiness** (either the chart or on the left navigation). Note that for each VM, a specific concern is listed regarding the readiness of the VM for migration.

    ![Screenshot showing the Azure Migrate assessment report on the VM readiness page, with the VM readiness for each VM highlighted.](images/Exercise1/readiness.png)

11. Select **Unsupported boot type** for **smarthotelweb1**. A new browser tab opens showing Azure Migrate documentation. Note on the page that the issue relates to EFI boot not being supported, and the recommendation to migrate using Azure Migrate Server Migration, which will convert the boot type to BIOS.

    ![Screenshot of Azure documentation showing troubleshooting advice for the 'Unsupported boot type' issue. It states that EFI boot is not supported in Azure, and suggests using Azure Migrate Server Migration for Azure migration since this will convert the boot type to BIOS.](images/Exercise1/unsupported-boot-type-doc.png)

12. Return to the portal browser tab to see details of the issue, and once again the suggestion to migrate using Azure Migrate Server Migration. (The 'Suggestion' text referring to Azure Site Recovery reflects that Azure Migrate: Server Migration uses Azure Site Recovery under the hood as the migration engine.)

    ![Screenshot of Azure documentation showing troubleshooting advice for the 'Unsupported boot type' issue. It suggests using Azure Migrate: Server Migration for Azure migration since this will convert the boot type to BIOS.](images/Exercise1/unsupported-boot-type-portal.png)

13. Take a few minutes to explore other aspects of the migration assessment. Check why the UbuntuWAF is marked as 'conditionally ready for Azure', and explore the costs associated with the migration.

#### Task summary 
 
In this task you created and configured an Azure Migrate migration assessment.

### Task 5: Configure dependency visualization

When migrating a workload to Azure, it is important to understand all workload dependencies. A broken dependency could mean that the application doesn't run properly in Azure, perhaps in hard-to-detect ways. Some dependencies, such as those between application tiers, are obvious. Other dependencies, such as DNS lookups, Kerberos ticket validation or certificate revocation checks, are not.

In this task, you will configure the Azure Migrate dependency visualization feature. This requires you to first create a Log Analytics workspace, and then to deploy agents on the to-be-migrated VMs.

1. Return to the **Azure Migrate** blade in the Azure Portal, and select **Servers**. Under **Azure Migrate: Server Assessment** select **Groups**, then select the **SmartHotel VMs** group to see the group details. Note that each VM has their **Dependencies** status as **Requires agent installation**. Select **Requires agent installation** for the **smarthotelweb1** VM.

    ![Screenshot showing the SmartHotel VMs group. Each VM has dependency status 'Requires agent installation'.](images/Exercise1/requires-agent-installation.png)

2. On the **Dependencies** blade, select **Configure OMS workspace**.

    ![Screenshot of the Azure Migrate 'Dependencies' blade, with the 'Configure OMS Workspace' button highlighted.](images/Exercise1/configure-oms-link.png)

3. Create a new OMS workspace. Use **AzureMigrateWS\<unique number\>** as the workspace name, where \<unique number\> is a random number. Choose a workspace location close to your lab deployment, then select **Configure**.

    ![Screenshot of the Azure Migrate 'Configure OMS workspace' blade.](images/Exercise1/configure-oms.png)

4. Wait for the workspace to be deployed. Once it is deployed, navigate to it and select **Advanced settings** under **Settings** on the left. Make a note of the **Workspace ID** and **Primary Key** (for example by using Notepad).

    ![Screenshot of part of the Azure Migrate 'Dependencies' blade, showing the OMS workspace ID and key.](images/Exercise1/workspace-id-key.png)

5. Right-click and copy the link in the address bar and make a note of it in Notepad. 
   
    ![Screenshot of part of the Azure Migrate 'Advanced Settings' blade, showing the agent download links.](images/Exercise1/agent-links.png)

6. Return to the RDP session with the **SmartHotelHost**. In **Hyper-V Manager**, select **smarthotelweb1** and select **Connect**.

    ![Screenshot from Hyper-V manager highlighting the 'Connect' button for the smarthotelweb1 VM.](images/Exercise1/connect-web1.png)

7. Select **Connect** again when prompted and log in to the **Administrator** account using the password **demo!pass123**.

8. Open **Internet Explorer**, and paste the link to the portal into the address bar. You may be prompted to sign in again. Choose the 64 bit download link and run the installer. 

    > **Note**: If Internet Explorer does not load the site properly you may need to turn of the **Internet Explorer Enhanced Security Configuration** on **Server Manager** under **Local Server** and install Chrome on the VM to navigate to the link. 

    ![Screenshot showing the Internet Explorer prompt to run the installer for the Microsoft Monitoring Agent.](images/Exercise1/mma-win-run.png)

9. Select through the installation wizard until you get to the **Agent Setup Options** page. From there, select **Connect the agent to Azure Log Analytics (OMS)** and select **Next**. Enter your Workspace ID and Workspace Key that were copied earlier on the next page, and select **Azure Commercial** from the Azure Cloud drop-down. Select through the remaining pages and install the agent.

    ![Screenshot of the Microsoft Monitoring Agent install wizard, showing the Log Analytics (OMS) workspace ID and key.](images/Exercise1/mma-wizard.png)

10. Paste the following link to the Dependency Agent Windows installer into the browser address bar. **Run** the installer and select through the install wizard to complete the installation.

    ```
    https://aka.ms/dependencyagentwindows
    ```

    ![Screenshot showing the Internet Explorer prompt to run the installer for the Dependency Agent.](images/Exercise1/da-win-run.png)

11. Close the virtual machine connection window for the **smarthotelweb1** VM.  Connect to the **smarthotelweb2** VM and repeat the installation process (steps 8-10) for both agents (the administrator password is the same as for smarthotelweb1).

    You will now deploy the Linux versions of the Microsoft Monitoring Agent and Dependency Agent on the UbuntuWAF VM. To do so, you will first connect to the UbuntuWAF remotely using an SSH session to the IP address of the SmartHotelHost. The SmartHotelHost has been pre-configured with a NAT rule which forwards SSH connections to the UbuntuWAF VM.

12. Return to the RDP session with the **SmartHotelHost** and open a command prompt using the desktop shortcut.  

    > **Note**: The SmartHotelHost runs Windows Server 2019 with the Windows Subsystem for Linux enabled. This allows the command prompt to be used as an SSH client.

13. Enter the following command to connect to the **UbuntuWAF** VM running in Hyper-V on the SmartHotelHost:

    ```bash
    ssh demouser@192.168.0.8
    ```

14. Enter 'yes' when prompted whether to connect. Use the password **demo!pass123**.

    ![Screenshot showing the command prompt with an SSH session to UbuntuWAF.](images/Exercise1/ssh.png)

15. Enter the following command, followed by the password **demo!pass123** when prompted:

    ```s
    sudo -s
    ```

    This gives the terminal session elevated privileges.

16. Enter the following command, substituting \<Workspace ID\> and \<Workspace Key\> with the values copied previously. Answer **<Yes>** when prompted to restart services during package upgrades without asking. 

    ```s
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <Workspace ID> -s <Workspace Key>
    ```

17. Enter the following command, substituting \<Workspace ID\> with the value copied earlier:

    ```s
    /opt/microsoft/omsagent/bin/service_control restart <Workspace ID>
    ```

18. Enter the following command. This downloads a script that will install the Dependency Agent.

    ```s
    wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
    ```

19. Install the dependency agent by running the script download in the previous step.

    ```s
    sh InstallDependencyAgent-Linux64.bin -s
    ```

    ![Screenshot showing that the Dependency Agent install on Linux was successful.](images/Exercise1/da-linux-done.png)

20. The agent installation is now complete. Next, you need to generate some traffic on the SmartHotel application so the dependency visualization has some data to work with. Browse to the public IP address of the SmartHotelHost, and spend a few minutes refreshing the page and checking guests in and out.

#### Task summary 

In this task you configured the Azure Migrate dependency visualization feature, by creating a Log Analytics workspace and deploying the Azure Monitoring Agent and Dependency Agent on both Windows and Linux on-premises machines.

### Task 6: Explore dependency visualization

In this task, you will explore the dependency visualization feature of Azure Migrate. This feature uses data gathered by the dependency agent you installed in Task 5.

1. Return to the Azure Portal and refresh the Azure Migrate **SmartHotel VMs** VM group blade. The 3 VMs on which the dependency agent was installed should now show their status as 'Installed'. (If not, refresh the page **using the browser refresh button**, not the refresh button in the blade.)

    ![Screenshot showing the dependency agent installed on each VM in the Azure Migrate VM group.](images/Exercise1/dependency-viz-installed.png)

2. Select **View dependencies**.

    ![Screenshot showing the view dependencies button in the Azure Migrate VM group blade.](images/Exercise1/view-dependencies.png)

3. Take a few minutes to explore the dependencies view. Expand each server to show the processes running on that server. Select a process to see process information. See which connections each server makes.

    ![Screenshot showing the dependencies view in Azure Migrate.](images/Exercise1/dependencies.png)


#### Task summary 

In this task you explored the Azure Migrate dependency visualization feature.

### Exercise summary 

In this exercise, you used Azure Migrate to assess the on-premises environment. This included selecting Azure Migrate tools, deploying the Azure Migrate appliance into the on-premises environment, creating a migration assessment, and using the Azure Migrate dependency visualization.

## Exercise 2: Migrate the Application Database

Duration: 60 minutes

In this exercise you will migrate the application database from the on-premises Hyper-V virtual machine to a new database hosted in the Azure SQL Database service. You will use the Azure Database Migration Service to complete the migration, which uses the SQL Server Data Migration Assistant for the database assessment and schema migration phases.

### Task 1: Register the Microsoft.DataMigration resource provider

Prior to using the Azure Database Migration Service, the resource provider **Microsoft.DataMigration** must be registered in the target subscription.

1. Open the Azure Cloud Shell by navigating to **https://shell.azure.com**. Log in using your Azure subscription credentials if prompted to do so, select a **PowerShell** session, and accept any prompts.

2. Run the following command to register the **Microsoft.DataMigration** resource provider:
   
    ```PowerShell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataMigration
    ```
    > **Note**: It may take several minutes for the resource provider to register. You can proceed to the next task without waiting for the registration to complete. You will not use the resource provider until task 3.
    >
    > You can check the status by running:

    > ```PowerShell
    > Get-AzResourceProvider -ProviderNamespace Microsoft.DataMigration | Select-Object ProviderNamespace, RegistrationState, ResourceTypes
    > ```

#### Task summary 

In this task you registered the **Microsoft.DataMigration** resource provider with your subscription. This enables this subscription to use the Azure Database Migration Service.

### Task 2: Create an Azure SQL Database

In this task you will create a new Azure SQL database to migrate the on-premises database to.

1. Open the Azure portal at https://portal.azure.com and log in using your subscription credentials if it's not still up.

2. Expand the portal's left navigation by selecting **Show portal menu** in the top left then select **+ Create a resource**, then select **Databases**, then select **SQL Database**.

    ![Azure portal screenshot showing the select path to create a SQL Database.](images/Exercise2/new-sql-db.png)

3. The **Create SQL Database** blade opens, showing the **Basics** tab. Complete the form as follows:

    - Subscription: **Select your subscription**.
  
    - Resource group: (create new) **SmartHotelDBRG**
  
    - Database name: **smarthoteldb**
  
    - Server: Select **Create new** and fill in the New server blade as follows then select **OK**:
  
        - Server name: **smarthoteldb\[unique number\]**
  
        - Server admin login: **demouser**
  
        - Password: **demo!pass123**
  
        - Location: **IMPORTANT: For most users, select the same region you used when you started your lab - this makes migration faster. If you are using an Azure Pass subscription, choose a different region to stay within the Total Regional vCPU limit.**

    > **Note**: You can verify the location by opening another browser tab, navigating to https://portal.azure.com and selecting Virtual Machines on the left navigation. Use the same region as the **SmartHotelHost** virtual machine.

    - Use SQL elastic pool: **No**
  
    - Compute + storage: **Standard S0**

    > **Note**: To select the **Standard S0** database tier, select **Configure database**, then **Looking for basic, standard, premium?**, select **Standard** and select **Apply**.

    ![Screenshot from the Azure portal showing the Create SQL Database blade.](images/Exercise2/new-db.png)

    ![Screenshot from the Azure portal showing the New server blade (when creating a SQL database).](images/Exercise2/new-db-server.png)

4. Select **Next: Networking >** to move to the **Networking** tab. Confirm that **No access** is selected.

    > **Note**: We will configure private endpoints to access our database later in the lab.

5. Select **Review + Create**, then select **Create** to create the database. Wait for the deployment to complete.

#### Task summary 

In this task you created an Azure SQL Database running on an Azure SQL Database Server.

### Task 3: Create the Database Migration Service

In this task you will create an Azure Database Migration Service resource. This resource is managed by the Microsoft.DataMigration resource provider which you registered in task 1.

> **Note**: The Azure Database Migrate Service (DMS) requires network access to your on-premises database to retrieve the data to transfer. To achieve this access, the DMS is deployed into an Azure VNet. You are then responsible for connecting that VNet securely to your database, for example by using a Site-to-Site VPN or ExpressRoute connection.
>
> In this lab, the 'on-premises' environment is simulated by a Hyper-V host running in an Azure VM. This VM is deployed to the 'smarthotelvnet' VNet. The DMS will be deployed to a separate VNet called 'DMSVnet'. To simulate the on-premises connection, these two VNet have been peered.

1. Return to the cloud shell browser tab you used in task 1 to register the Microsoft.DataMigration resource provider. Check that the registration has been completed by running the following command before proceeding further.

    ```
    Get-AzResourceProvider -ProviderNamespace Microsoft.DataMigration | Select-Object ProviderNamespace, RegistrationState, ResourceTypes
    ```

      ![Screenshot showing the resource provider 'registered' status.](images/Exercise2/registered-rp.png)

2. In the Azure portal, expand the portal's left navigation and select **+ Create a resource**, search for **migration**, and then select **Azure Database Migration Service** from the drop-down list.

3. On the **Azure Database Migration Service** blade, select **Create**.

    ![Screenshot showing the DMS 'create' button.](images/Exercise2/dms-create-1.png)

   > **Tip**: If the migration service blade will not load, refresh the portal blade in your browser.

4. In the **Create Migration Service** blade, on the **Basics** tab, enter the following values:
   
    - Subscription: **Select your Azure subscription**.
  
    - Resource group: **AzureMigrateRG**
  
    - Service Name: **SmartHotelDBMigration**
  
    - Location: **Choose the same region as the SmartHotel host**.
  
    - Pricing tier: **Standard: 1 vCore**

    ![Screenshot showing the Create DMS 'Basics' tab.](images/Exercise2/create-dms.png)

5. Select **Next: Networking** to move to the **Networking** tab, and select the **DMSvnet/DMS** virtual network and subnet in the **SmartHotelHostRG** resource group.
   
    ![Screenshot showing the Create DMS 'Networking' tab.](images/Exercise2/create-dms-network.png)

6. Select **Review + create**, followed by **Create**.

> **Note**: Creating a new migration service can take around 20 minutes. You can continue to the next task without waiting for the operation to complete. You will not use the Database Migration Service until task 5.

#### Task summary 

In this task you created a new Azure Database Migration Service resource.

### Task 4: Assess the on-premises database using Data Migration Assistant

In this task you will install and use Microsoft SQL Server Data Migration Assistant (DMA) to assess the on-premises database. The DMA is integrated with Azure Migrate providing a single hub for assessment and migration tools.

1. Return to the **Azure Migrate** blade in the Azure portal. Select the **Overview** panel, then select **Assess and migrate databases**.

    ![Screenshot showing the Azure Migrate Overview blade in the Azure portal, with the 'Assess and migrate databases' button highlighted.](images/Exercise2/assess-migrate-db.png)  

2. Select **Add tool(s)** to open the **Add a tool** wizard.

3. Under **Select assessment tool**, select **Azure Migrate: Database Assessment**, then select **Next**.

    ![Screenshot showing the 'Select assessment tool' step of the 'Add a tool' wizard in Azure Migrate, with the 'Azure Migrate: Database Assessment' tool selected.](images/Exercise2/add-db-assessment-tool.png)

4. Under **Select migration tool**, select **Azure Migrate: Database Migration**, then select **Next**.

    ![Screenshot showing the 'Select assessment tool' step of the 'Add a tool' wizard in Azure Migrate, with the 'Azure Migrate: Database Migration' tool selected.](images/Exercise2/add-db-migration-tool.png)

5. Under **Review and add tools**, select **Add tool(s)**.

6. Once the tools are installed in Azure Migrate, the portal should show the **Azure Migrate - Databases** blade. Under **Azure Migrate: Database Assessment** select **+ Assess**.

    ![Screenshot highlighting the '+Assess' link on the 'Azure Migrate - Databases' blade in the Azure portal.](images/Exercise2/db-assess.png)

7. Select **Download** to open the Data Migration Assistant download page. Copy the page URL to the clipboard.

8. Return to your remote desktop session with the **SmartHotelHost** VM. Open **Chrome** from the desktop and paste the Data Migration Assistant download URL into the address bar. Download and install the Data Migration Assistant.

9.  From within **SmartHotelHost**, open **Windows Explorer** and navigate to the **C:\\Program Files\\Microsoft Data Migration Assistant** folder. Open the **Dma.exe.config** file using Notepad. Search for **AzureMigrate** and remove the **\<\!--** and **--\>** around the line setting the **EnableAssessmentUploadToAzureMigrate** key. **Save** the file and close Notepad when done.

  ![Screenshot showing the Dma.exe.config setting enabling upload to Azure Migrate.](images/Exercise2/dma-enable-upload.png)

10. From within **SmartHotelHost** launch **Microsoft Data Migration Assistant** using the desktop icon. 

11. In the Data Migration Assistant, select the **+ New** icon.  Fill in the project details as follows:

    - Project type: **Assessment**
  
    - Project name: **SmartHotelAssessment**
  
    - Assessment type: **Database Engine**
  
    - Source server type: **SQL Server**
  
    - Target server type: **Azure SQL Database**
     
12. Select **Create** to create the project.

    ![Screenshot showing the new DMA project creation dialog.](images/Exercise2/new-dma-assessment.png)

13. On the **Options** tab select **Next**.

14. On the **Select sources** page, in the **Connect to a server** dialog box, provide the connection details to the SQL Server, and then select **Connect**.

    - Server name: **192.168.0.6**
  
    - Authentication type: **SQL Server Authentication**
  
    - Username: **sa**
  
    - Password: **demo!pass123**
  
    - Encrypt connection: **Checked**
  
    - Trust server certificate: **Checked**

    ![Screenshot showing the DMA connect to a server dialog.](images/Exercise2/connect-to-a-server.png)

15. In the **Add sources** dialog box, select **SmartHotel.Registration**, then select **Add**.

    ![Screenshot of the DMA showing the 'Add sources' dialog.](images/Exercise2/add-sources.png)

16. Select **Start Assessment** to start the assessment. 

    ![Screenshot of the DMA showing assessment in progress.](images/Exercise2/assessment-in-progress.png)

17. **Wait** for the assessment to complete, and review the results. The results should show one unsupported feature, **Service Broker feature is not supported in Azure SQL Database**. For this migration, you can ignore this issue.

    > **Note**: For Azure SQL Database, the assessments identify feature parity issues and migration blocking issues.

    >- The SQL Server feature parity category provides a comprehensive set of recommendations, alternative approaches available in Azure, and mitigating steps to help you plan the effort into your migration projects.

    >- The Compatibility issues category identifies partially supported or unsupported features that reflect compatibility issues that might block migrating on-premises SQL Server database(s) to Azure SQL Database. Recommendations are also provided to help you address those issues.

18. Select **Upload to Azure Migrate**  to upload the database assessment to your Azure Migrate project (this button may take a few seconds to become enabled).

    ![Screenshot of the DMA showing the assessment results and the 'Update to Azure Migrate' button.](images/Exercise2/db-upload-btn.png)

19. Select **Azure** from the dropdown on the right then select **Connect**. Enter your subscription credentials when prompted. Select your **Subscription** and **Azure Migrate Project** using the dropdowns, then select **Upload**. Once the upload is complete, select **OK** to dismiss the notification.

    ![Screenshot of the DMA showing the assessment results upload panel.](images/Exercise2/db-upload.png)

20. Minimize the remote desktop window and return to the **Azure Migrate - Databases** blade in the Azure portal. Refreshing the page should now show the assessed database.

    ![Screenshot of the 'Azure Migrate - Databases' blade in the Azure portal, showing 1 assessed database.](images/Exercise2/db-assessed.png)

#### Task summary 

In this task you used Data Migration Assistant to assess an on-premises database for readiness to migrate to Azure SQL, and uploaded the assessment results to your Azure Migrate project. The DMA is integrated with Azure Migrate providing a single hub for assessment and migration tools.

### Task 5: Create a DMS migration project

In this task you will create a Migration Project within the Azure Database Migration Service (DMS). This project contains the connection details for both the source and target databases. In order to connect to the target database, you will also create a private endpoint allowing connectivity from the subnet used by the DMS.

In subsequent tasks, you will use this project to migrate both the database schema and the data itself from the on-premises SQL Server database to the Azure SQL Database.

We'll start by creating the private endpoint that allows the DMS to access the database server.

1. In the Azure portal, navigate to the **SmartHotelHostDBRG** resource group, and then to the database server.

2. Under **Security**, select **Firewalls and virtual networks**. Note that there is no access granted to the database public IP address.

    ![Screenshot showing the link to add an existing virtual network to the SQL database network security settings.](images/Exercise2/db-network.png)

3. Select **Private endpoint connections** under **Security**, then **+ Private endpoint**.

4. On the **Basics** tab that appears, enter the following configuration then select **Next: Resource**. 

    - Resource group: **SmartHotelDBRG**
  
    - Name: **SmartHotel-DB-for-DMS**
  
    - Region: **Select the same location as the DMSvnet (Should be the region closest to you)**.
  
    ![Screenshot showing the 'Create a private endpoint' blade, 'Basics' tab.](images/Exercise2/private-endpoint-1.png)

5. On the **Resource** tab, entering the following configuration then select **Next: Configuration**. 

    - Connection method: **Connect to an Azure resource in my directory**.
  
    - Subscription: **Select your subscription**.
  
    - Resource type: **Microsoft.Sql/servers**
  
    - Resource: **Your SQL database server name**.
  
    - Target sub-resource: **sqlServer**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Resource' tab.](images/Exercise2/private-endpoint-2.png)
   
6. On the **Configuration** tab enter the following configuration then select **Review + create**, then **Create**.

    - Virtual network: **DMSvnet**
  
    - Subnet: **DMS (10.1.0.0/24)**
  
    - Integrate with private DNS zone: **Yes**
  
    - Private DNS zones: (default) **privatelink.database.windows.net**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Configuration' tab.](images/Exercise2/private-endpoint-3.png)

7. **Wait** for the deployment to complete. Open the Private Endpoint blade, and note that the FQDN for the endpoint is listed as **\<your database\>.database.windows.net**, with an internal IP address **10.1.0.5**.

    ![Screenshot showing the DNS entry for the SQL database server private endpoint](images/Exercise2/private-endpoint-dns.png)

    >**Note**: Private DNS is used so that the database domain name, **\<your server\>.database.windows.net** resolves to the internal private endpoint IP address **10.1.0.5** when resolved from the DMSvnet, but resolves to the Internet-facing IP address of the database server when resolved from outside the DMSvnet. This means the same connection string (which contains the domain name) can be used in both cases.

8. Check that the Database Migration Service resource you created in task 3 has completed provisioning. You can check the deployment status from the **Deployments** pane in the **AzureMigrateRG** resource group blade.

    ![Screenshot showing the AzureMigrateRG - Deployments blade in the Azure portal. The Microsoft.AzureDMS deployment shows status 'Successful'.](images/Exercise2/dms-deploy.png)

9. Navigate to the Database Migration Service resource blade in the **AzureMigrateRG** resource group and select **+ New Migration Project**.

    ![Screenshot showing the Database Migration Service blade in the Azure portal, with the 'New Migration Project' button highlighted.](images/Exercise2/new-dms-project.png)

10. In the **New migration project** blade, enter **DBMigrate** as the project name. Leave the source server type as **SQL Server** and target server type as **Azure SQL Database**. Select **Choose type of activity** and select **Create project only**. Select **Save** then select **Create**.

    ![Screenshot showing the Database Migration Service blade in the Azure portal, with the 'New Migration Project' button highlighted.](images/Exercise2/new-migrate-project.png)

11. The Migration Wizard opens, showing the **Select source** step. Complete the settings as follows, and select **Save**.

    - Source SQL Server instance name: **10.0.0.4**
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **sa**
  
    - Password: **demo!pass123**

    - Encryption connection: **Checked**
  
    - Trust server certificate: **Checked**

    ![Screenshot showing the 'Select source' step of the DMS Migration Wizard.](images/Exercise2/select-source.png)

    > **Note**: The DMS service connects to the Hyper-V host, which has been pre-configured with a NAT rule to forward incoming SQL requests (TCP port 1433) to the SQL Server VM. In a real-world migration, the SQL Server VM would most likely have its own IP address on the internal network, via an external Hyper-V switch.
    >
    > The Hyper-V host is accessed via its private IP address (10.0.0.4). The DMS service accesses this IP address over the peering connection between the DMS VNet and the SmartHotelHost VNet. This simulates a VPN or ExpressRoute connection between a DMS VNet and an on-premises network.

12. In the **Select source databases** step, the **Smarthotel.Registration** database should already be selected. Select **Save**.

    ![Screenshot showing the 'Select databases' step of the DMS Migration Wizard.](images/Exercise2/select-databases.png)

13. Complete the **Select target** step as follows, then select **Save**:

    - Target server name: **Value from your database, {something}.database.windows.net**.
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **demouser**
  
    - Password: **demo!pass123**
  
    - Encrypt connection: **Checked**

    ![Screenshot showing the DMS migration target settings.](images/Exercise2/select-target.png)

    > **Note**: You can find the target server name in the Azure portal by browsing to your database.

    ![Screenshot showing the Azure SQL Database server name.](images/Exercise2/sql-db-name.png)

14. At the **Project summary** step, review the settings and select **Save** to create the migration project.

    ![Screenshot showing the DMS project summary.](images/Exercise2/project-summary.png)

#### Task summary 

In this task you created a Migration Project within the Azure Database Migration Service. This project contains the connection details for both the source and target databases. A private endpoint was used to avoid exposing the database on a public IP address.

### Task 6: Migrate the database schema

In this task you will use the Azure Database Migration Service to migrate the database schema to Azure SQL Database. This step is a prerequisite to migrating the data itself.

The schema migration will be carried out using a schema migration activity within the migration project created in task 5.

1. Following task 5, the Azure portal should show a blade for the DBMigrate DMS project. Select **+ New Activity** and select **Schema only migration** from the drop-down.

    ![Screenshot showing the 'New Activity' button within an Azure Database Migration Service project, with 'Schema only migration' selected from the drop-down.](images/Exercise2/new-activity-schema.png)

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **demo!pass123**, then select **Save**.

    ![Screenshot showing the 'Select source' step of the DMS Migration Wizard. The source database password is highlighted.](images/Exercise2/select-source-pwd-only.png)

3. At the **Select target** step, enter the password **demo!pass123** and select **Save**.

    ![Screenshot showing the 'Select target' step of the DMS Migration Wizard. The target database password is highlighted.](images/Exercise2/select-target-pwd-only.png)

4. At the **Select database and schema** step, check that the **SmartHotel.Registration** database is selected. Under **Target Database** select **smarthoteldb** and under **Schema Source** select **Generate from source**. Select **Save**.

    ![Screenshot showing the 'Select database and schema' step of the DMS Migration Wizard.](images/Exercise2/select-database-and-schema.png)

5. At the **Summary** step, enter **SchemaMigration** as the **Activity name**. Select **Run migration** to start the schema migration process.

    ![Screenshot showing the 'Summary' step of the DMS Migration Wizard. The activity name, validation option, and 'Run migration' button are highlighted](images/Exercise2/run-schema-migration.png)

6. The schema migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

    ![Screenshot showing the SchemaMigration progress blade. The status is 'Completed'.](images/Exercise2/schema-completed.png)

#### Task summary

In this task you used a schema migration activity in the Azure Database Migration Service to migrate the database schema from the on-premises SQL Server database to the Azure SQL database.

### Task 7: Migrate the on-premises data

In this task you will use the Azure Database Migration Service to migrate the database data to Azure SQL Database.

The schema migration will be carried out using an offline data migration activity within the migration project created in task 5.

1. Return to the Azure portal blade for your **DBMigrate** migration project in DMS. Select **+ New Activity** and select **Offline data migration** from the drop-down.

    ![Screenshot showing the 'New Activity' button within an Azure Database Migration Service project, with 'Offline data migration' selected from the drop-down.](images/Exercise2/new-activity-data.png)

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **demo!pass123**, then select **Save**.

    ![Screenshot showing the 'Select source' step of the DMS Migration Wizard. The source database password is highlighted.](images/Exercise2/select-source-pwd-only-data.png)

3. At the **Select target** step, enter the password **demo!pass123** and select **Save**.

    ![Screenshot showing the 'Select target' step of the DMS Migration Wizard. The target database password is highlighted.](images/Exercise2/select-target-pwd-only-data.png)

4. At the **Map to target databases** step, check the **SmartHotel.Registration** database. Under **Target Database** select **smarthoteldb**. Select **Save**.

    ![Screenshot showing the 'Select database and schema' step of the DMS Migration Wizard.](images/Exercise2/map-target-db.png)

5. The **Configure migration settings** step allows you to specify which tables should have their data migrated. Select the **Bookings** table (Make sure the **MigrationHistory** table is not checked) and select **Save**.

    ![Screenshot from DMS showing tables being selected for replication.](images/Exercise2/select-tables.png)

6. At the **Migration summary** step, enter **DataMigration** as the **Activity name**. Select **Run migration**.

    ![Screenshot from DMS showing a summary of the migration settings.](images/Exercise2/run-data-migration.png)

7. The data migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

    > **Note:** You may receive a warning message about validation issues but the migration will still show as '1 of 1 table(s) completed'.

    ![Screenshot from DMS showing the data migration in completed.](images/Exercise2/data-migration-completed.png)

As a final step, we will remove the private endpoint that allows the DMS service access to the database, since this access is no longer required.

8.  In the Azure portal, navigate to the **SmartHotelHostDBRG** resource group, and then to the database server. Under **Security**, select **Private endpoint connections**.

9.  Select the **SmartHotel-DB-for-DMS** endpoint added earlier, and select **Remove**.

    ![Screenshot from the SQL server showing the SmartHotel-DB-for-DMS private endpoint being removed.](images/Exercise2/private-endpoint-remove.png)

#### Task summary 

In this task you used an off-line data migration activity in the Azure Database Migration Service to migrate the database data from the on-premises SQL Server database to the Azure SQL database.

### Exercise summary 

In this exercise you migrated the application database from on-premises to Azure SQL Database. The SQL Server Data Migration Assistant was used for migration assessment, and the Azure Database Migration Service was used for schema migration and data migration.

## Exercise 3: Migrate the application and web tiers using Azure Migrate: Server Migration

Duration: 90 minutes

In this exercise you will migrate the web tier and application tiers of the application from on-premises to Azure using Azure Migrate: Server Migration.

Having migrated the virtual machines, you will reconfigure the application tier to use the application database hosted in Azure SQL. This will enable you to verify that the migration application is working end-to-end.

### Task 1: Create a Storage Account

In this task you will create a new Azure Storage Account that will be used by Azure Migrate: Server Migration for storage of your virtual machine data during migration.

1. In the Azure portal's left navigation, select **+ Create a resource**, then select **Storage**, followed by **Storage account**.

    ![Screenshot of the Azure portal showing the create storage account navigation.](images/Exercise3/create-storage-1.png)

2. In the **Create storage account** blade on the **Basics** tab, use the following values:

    - Subscription: **Select your Azure subscription**.
  
    - Resource group: **AzureMigrateRG**
  
    - Storage account name: **migrationstorage\[unique number\]**
  
    - Location: **IMPORTANT: Select the same location as your Azure SQL Database** (can be found in the Azure portal).
  
    - Account kind: **Storage (general purpose v1)** (do not use a v2 account).
  
    - Replication: **Locally-redundant storage (LRS)**

    ![Screenshot of the Azure portal showing the create storage account blade.](images/Exercise3/create-storage-2.png)

3. Select **Review + create**, then select **Create**.

#### Task summary 

In this task you created a new Azure Storage Account that will be used by Azure Migrate.

### Task 2: Create a Virtual Network

In this task you will create a new virtual network that will be used by your migrated virtual machines when they are migrated to Azure. (Azure Migrate will only create the VMs, their network interfaces, and their disks; all other resources must be staged in advance.)

You will also configure a private endpoint in this network for the SQL Database.

1. In the Azure portal's left navigation, select **+ Create a resource**, then select **Networking**, followed by **Virtual network**.

    ![Screenshot of the Azure portal showing the create virtual network navigation.](images/Exercise3/create-vnet-1.png)

2. In the **Create virtual network** blade, enter the following values:

    - Subscription: **Select your Azure subscription**.
  
    - Resource group: (create new) **SmartHotelRG**
  
    - Name: **SmartHotelVNet**
  
    - Region: **IMPORTANT: Select the same location as your Azure SQL Database**.

    ![Screenshot of the Azure portal showing the create virtual network blade 'Basics' tab.](images/Exercise3/create-vnet-2.png)

3. Select **Next: IP Addresses >**, and enter the following configuration. Then select **Review + create**, then **Create**.

    - IPv4 address space: **192.168.0.0/24** 
  
    - First subnet: Select **Add subnet** and enter the following then select **Add**

        - Subnet name: **SmartHotel**
   
        - Address range: **192.168.0.0/25**
  
    - Second subnet: Select **Add subnet** and enter the following then select **Add**. 

        - Subnet name: **SmartHotelDB**
   
        - Address range: **192.168.0.128/25**

    ![Screenshot of the Azure portal showing the create virtual network blade 'IP Addresses' tab.](images/Exercise3/create-vnet-3.png)

4. Navigate to the **SmartHotelHostDBRG** resource group, and then to the database server. Under **Security**, select **Private endpoint connections**, then select **+ Private endpoint**.

5. On the **Basics** tab, enter the following configuration then select **Next: Resource**:

    - Resource group: **SmartHotelDBRG**
  
    - Name: **SmartHotel-DB-Endpoint**
  
    - Region: **Select the same location as the SmartHotelVNet**.
  
    ![Screenshot showing the 'Create a private endpoint' blade, 'Basics' tab.](images/Exercise3/private-endpoint-1.png)

6.  On the **Resource** tab, enter the following configuration then select **Next: Configuration**:

    - Connection method: **Connect to an Azure resource in my directory**.
  
    - Subscription: **Select your subscription**.
  
    - Resource type: **Microsoft.Sql/servers**
  
    - Resource: **Your SQL database server**.
  
    - Target sub-resource: **sqlServer**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Resource' tab.](images/Exercise3/private-endpoint-2.png)
   
7.  On the **Configuration** tab, enter the following configuration then select **Review + Create** then **Create**:

    - Virtual network: **SmartHotelVNet**
  
    - Subnet: **SmartHotelDB (192.168.0.128/25)**
  
    - Integrate with private DNS zone: **Yes**
  
    - Private DNS zone: (default) **privatelink.database.windows.net**

    ![Screenshot showing the 'Create a private endpoint' blade, 'Configuration' tab.](images/Exercise3/private-endpoint-3.png)

8. **Wait** for the deployment to complete. Open the Private Endpoint blade, and note that the FQDN for the endpoint is listed as **\<your database\>.database.windows.net**, with an internal IP address **192.168.0.132**.

    >**Note**: Private DNS is used so that the database domain name, **\<your server\>.database.windows.net** resolves to the internal private endpoint IP address **192.168.0.132** when resolved from the SmartHotelVNet, but resolves to the Internet-facing IP address of the database server when resolved from outside the VNet. This means the same connection string (which contains the domain name) can be used in both cases.

#### Task summary 

In this task you created a new virtual network that will be used by your virtual machines when they are migrated to Azure. You also created a private endpoint in this network, which will be used to access the SQL database.

### Task 3: Register the Hyper-V Host with Azure Migrate Server Migration

In this task, you will register your Hyper-V host with the Azure Migrate: Server Migration service. This service uses Azure Site Recovery as the underlying migration engine. As part of the registration process, you will deploy the Azure Site Recovery Provider on your Hyper-V host.

1. Return to the **Azure Migrate** blade in the Azure Portal, and select **Servers** under **Migration goals** on the left. Under **Migration Tools**, select **Discover**.

    **Note:** You may need to add the migration tool yourself by following the link below the **Migration Tools** section, selecting **Azure Migrate: Server Migration**, then selecting **Add tool(s)**. 

    ![Screenshot of the Azure portal showing the 'Discover' button on the Azure Migrate Server Migration panel.](images/Exercise3/discover-1.png)

2. In the **Discover machines** panel, under **Are your machines virtualized**, select **Yes, with Hyper-V**. Under **Target region** enter **the same region as used for your Azure SQL Database** which can be found in the Azure portal and check the confirmation checkbox. Select **Create resources** to begin the deployment of the Azure Site Recovery resource used by Azure Migrate: Server Migration for Hyper-V migrations.

    ![Screenshot of the Azure portal showing the 'Discover machines' panel from Azure Migrate.](images/Exercise3/discover-2.png)

    Once deployment is complete, the 'Discover machines' panel should be updated with additional instructions.
  
3. Copy the **Download** link for the Hyper-V replication provider software installer to your clipboard.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link for the Hyper-V replication provider software installer.](images/Exercise3/discover-3.png)

4. Open the **SmartHotelHost** remote desktop window, launch **Chrome** from the desktop shortcut, and paste the link into a new browser tab to download the Azure Site Recovery provider installer.

5. Return to the **Discover machines** page in your browser (outside the SmartHotelHost remote desktop session). Select the blue **Download** button and download the registration key file.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link Hyper-V registration key file.](images/Exercise3/discover-4.png)

6. Open the file location in Windows Explorer, and copy the file to your clipboard. Return to the **SmartHotelHost** remote desktop session and paste the file to the desktop.

7. Still within the **SmartHotelHost** remote desktop session, open the **AzureSiteRecoveryProvider.exe** installer you downloaded a moment ago. On the **Microsoft Update** tab, select **Off** and select **Next**. Accept the default installation location and select **Install**.

    ![Screenshot of the ASR provider installer.](images/Exercise3/asr-provider-install.png)

8. When the installation has completed select **Register**. Browse to the location of the key file you downloaded. When the key is loaded select **Next**.

    ![Screenshot of the ASR provider registration settings.](images/Exercise3/asr-registration.png)

9.  Select **Connect directly to Azure Site Recovery without a proxy server** and select **Next**. The registration of the Hyper-V host with Azure Site Recovery will begin.

10. Wait for registration to complete (this may take several minutes). Then select **Finish**.

    ![Screenshot of the ASR provider showing successful registration.](images/Exercise3/asr-registered.png)

11. Minimize the SmartHotelHost remote desktop session and return to the Azure Migrate browser window. **Refresh** your browser, then re-open the **Discover machines** panel by selecting **Discover** under **Azure Migrate: Server Migration** and selecting **Yes, with Hyper-V** for **Are your machines virtualized?**.

12. Select **Finalize registration**, which should now be enabled.

    ![Screenshot of the Discover machines' panel from Azure Migrate, highlighting the download link Hyper-V registration key file.](images/Exercise3/discover-5.png)

13. Azure Migrate will now complete the registration with the Hyper-V host. **Wait** for the registration to complete. This may take several minutes.

    ![Screenshot of the 'Discover machines' panel from Azure Migrate, showing the 'Finalizing registration...' message.](images/Exercise3/discover-6.png)

14. Once the registration is complete, close the **Discover machines** panel.

    ![Screenshot of the 'Discover machines' panel from Azure Migrate, showing the 'Registration finalized' message.](images/Exercise3/discover-7.png)

16. The **Azure Migrate: Server Migration** panel should now show 5 discovered servers.

    ![Screenshot of the 'Azure Migrate - Servers' blade showing 6 discovered servers under 'Azure Migrate: Server Migration'.](images/Exercise3/discover-8.png)

#### Task summary 

In this task you registered your Hyper-V host with the Azure Migrate Server Migration service.

### Task 4: Enable Replication from Hyper-V to Azure Migrate

In this task, you will configure and enable the replication of your on-premises virtual machines from Hyper-V to the Azure Migrate Server Migration service.

1. Under **Azure Migrate: Server Migration**, select **Replicate**. This opens the **Replicate** wizard.

    ![Screenshot highlighting the 'Replicate' button in the 'Azure Migrate: Server Migration' panel of the Azure Migrate - Servers blade.](images/Exercise3/replicate-1.png)

2. In the **Source settings** tab, under **Are your machines virtualized?**, select **Yes, with Hyper-V** from the drop-down. Then select **Next: Virtual machines**.

    ![Screenshot of the 'Source settings' tab of the 'Replicate' wizard in Azure Migrate Server Migration. Hyper-V replication is selected.](images/Exercise3/replicate-2.png)

3. In the **Virtual machines** tab, under **Import migration settings from an assessment**, select **Yes, apply migration settings from an Azure Migrate assessment**. Select the **SmartHotel VMs** VM group and the **SmartHotelAssessment** migration assessment.

    ![Screenshot of the 'Virtual machines' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The Azure Migrate assessment created earlier is selected.](images/Exercise3/replicate-3.png)

4. The **Virtual machines** tab should now show the virtual machines included in the assessment. Select the **UbuntuWAF**, **smarthotelweb1**, and **smarthotelweb2** virtual machines, then select **Next: Target settings**.

    ![Screenshot of the 'Virtual machines' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The UbuntuWAF, smarthotelweb1, and smarthotelweb2 machines are selected.](images/Exercise3/replicate-4.png)

5. In the **Target settings** tab, select your subscription and the existing **SmartHotelRG** resource group. Under **Replication storage account** select the **migrationstorage** storage account and under **Virtual Network** select **SmartHotelVNet**. Under **Subnet** select **SmartHotel**. Select **Next: Compute**.

    ![Screenshot of the 'Target settings' tab of the 'Replicate' wizard in Azure Migrate Server Migration. The resource group, storage account and virtual network created earlier in this exercise are selected.](images/Exercise3/replicate-5.png)

6. In the **Compute** tab, select the **Standard_F2s_v2** VM size for each virtual machine. Select the **Windows** operating system for the **smarthotelweb** virtual machines and the **Linux** operating system for the **UbuntuWAF** virtual machine. Select **Next: Disks**. 

    > **Note**: If you are using an Azure Pass subscription, your subscription may not have a quota allocated for FSv2 virtual machines. In this case, use **DS2_v2 or D2s_v3** virtual machines instead.

    ![Screenshot of the 'Compute' tab of the 'Replicate' wizard in Azure Migrate Server Migration. Each VM is configured to use a Standard_F2s_v2 SKU, and has the OS Type specified.](images/Exercise3/replicate-6.png)

7. In the **Disks** tab, review the settings but do not make any changes. Select **Next: Review + Start replication**, then select **Replicate** to start the server replication.

8. In the **Azure Migrate - Servers** blade, under **Azure Migrate: Server Migration**, select the **Overview** button.

    ![Screenshot of the 'Azure Migrate - Servers' blade with the 'Overview' button in the 'Azure Migrate: Server Migration' panel highlighted.](images/Exercise3/replicate-7.png)

9. Confirm that the 3 machines are replicating.

    ![Screenshot of the 'Azure Migrate: Server Migration' overview blade showing the replication state as 'Healthy' for 3 servers.](images/Exercise3/replicate-8.png)

10. Select **Replicating Machines** under **Manage** on the left.  Select **Refresh** occasionally and wait until all three machines have a **Protected** status, which shows the initial replication is complete. This may take several minutes.

    ![Screenshot of the 'Azure Migrate: Server Migration - Replicating machines' blade showing the replication status as 'Protected' for all 3 servers.](images/Exercise3/replicate-9.png)

#### Task summary 

In this task you enabled replication from the Hyper-V host to Azure Migrate, and configured the replicated VM size in Azure.

### Task 5: Configure static internal IP addresses for each VM

In this task you will modify the settings for each replicated VM to use a static private IP address that matches the on-premises IP addresses for that machine.

1. Still using the **Azure Migrate: Server Migration - Replicating machines** blade, select the **smarthotelweb1** virtual machine. This opens a detailed migration and replication blade for this machine. Take a moment to study this information.

    ![Screenshot from the 'Azure Migrate: Server Migration - Replicating machines' blade with the smarthotelweb1 machine highlighted.](images/Exercise3/config-0.png)

2. Select **Compute and Network** under **General** on the left, then select **Edit**.

   ![Screenshot of the smarthotelweb1 blade with the 'Compute and Network' and 'Edit' links highlighted.](images/Exercise3/config-1.png)

3. Confirm that the VM is configured to use the **F2s_v2** VM size (or **DS2_v2 or D2s_v3** if using an Azure Pass subscription) and that **Use managed disks** is set to **Yes**.

4. Under **Network Interfaces**, select **InternalNATSwitch** to open the network interface settings.

    ![Screenshot showing the link to edit the network interface settings for a replicated VM.](images/Exercise3/nic.png)

5. Change the **Private IP address** to **192.168.0.4**.

    ![Screenshot showing a private IP address being configured for a replicated VM in ASR.](images/Exercise3/private-ip.png)

6. Select **OK** to close the network interface settings blade, then **Save** the **smarthotelweb1** settings.

7. Repeat these steps to configure the private IP address for the other VMs.
 
    - For **smarthotelweb2** use private IP address **192.168.0.5**
  
    - For **UbuntuWAF** use private IP address **192.168.0.8**

#### Task summary

In this task you modified the settings for each replicated VM to use a static private IP address that matches the on-premises IP addresses for that machine

> **Note**: Azure Migrate makes a "best guess" at the VM settings, but you have full control over the settings of migrated items. In this case, setting a static private IP address ensures the virtual machines in Azure retain the same IPs they had on-premises, which avoids having to reconfigure the VMs during migration (for example, by editing web.config files).

### Task 6: Server migration

In this task you will perform a migration of the UbuntuWAF, smarthotelweb1, and smarthotelweb2 machines to Azure.

> **Note**: In a real-world scenario, you would perform a test migration before the final migration. To save time, you will skip the test migration in this lab. The test migration process is very similar to the final migration.

1. Return to the **Azure Migrate: Server Migration** overview blade. Under **Step 3: Migrate**, select **Migrate**.

    ![Screenshot of the 'Azure Migrate: Server Migration' overview blade, with the 'Migrate' button highlighted.](images/Exercise3/migrate-1.png)

2. On the **Migrate** blade, select the 3 virtual machines then select **Migrate** to start the migration process.

    ![Screenshot of the 'Migrate' blade, with 3 machines selected and the 'Migrate' button highlighted.](images/Exercise3/migrate-2.png)

    > **Note**: You can optionally choose whether the on-premises virtual machines should be automatically shut down before migration to minimize data loss. Either setting will work for this lab.

3. The migration process will start.

    ![Screenshot showing 3 VM migration notifications.](images/Exercise3/migrate-3.png)

4. To monitor progress, select **Jobs** under **Manage** on the left and review the status of the three **Planned failover** jobs.

    ![Screenshot showing the **Jobs* link and a jobs list with 3 in-progress 'Planned failover' jobs.](images/Exercise3/migrate-4.png)

5. **Wait** until all three **Planned failover** jobs show a **Status** of **Successful**. You should not need to refresh your browser. This could take up to 15 minutes.

    ![Screenshot showing the **Jobs* link and a jobs list with all 'Planned failover' jobs successful.](images/Exercise3/migrate-5.png)

6. Navigate to the **SmartHotelRG** resource group and check that the VM, network interface, and disk resources have been created for each of the virtual machines being migrated.

   ![Screenshot showing resources created by the test failover (VMs, disks, and network interfaces).](images/Exercise3/migrate-6.png)

#### Task summary

In this task you used Azure Migrate to create Azure VMs using the settings you have configured, and the data replicated from the Hyper-V machines. This migrated your on-premises VMs to Azure.

### Task 7: Enable Azure Bastion

We will need to access our newly-migrated virtual machines to make some configuration changes. However, the machines do not currently have public IP addresses. Rather than add public IP addresses, we will access them using Azure Bastion.

Azure Bastion requires a dedicated subnet within the same virtual network as the virtual machines. Unfortunately, our SmartHotelVNet does not have any free network space available. To address this, we will first extend the network space.

1. Navigate to the **SmartHotelVNet** virtual network, then select **Address space** under **Settings** on the left.  Add the address space **10.10.0.0/24**, and **Save**.

2. Select **Subnets** under **Settings** on the left, and add a new subnet named **AzureBastionSubnet**, with address space **10.10.0.0/27**.

3. Select **+ Create a resource** in the portal's left navigation, then search for and select **Bastion**, then select **Create**.

4. Fill in the **Create a Bastion** blade as follows:

    - Subscription: **Your subscription**
  
    - Resource group: (Create new) **BastionRG**
  
    - Name: **SmartHotelBastion**
  
    - Region: **Same as SmartHotelVNet**
  
    - Virtual Network: **SmartHotelVNet**
  
    - Subnet: **AzureBastionSubnet**
  
    - Public IP address: (Create new) **Bastion-IP**

    ![Screenshot showing the 'Create a Bastion' blade.](images/Exercise3/bastion-create.png)

5. Select **Review + create**, then **Create**.

6. **Wait** for the Bastion to be deployed. This will take several minutes.

### Task 8: Configure the database connection

The application tier machine **smarthotelweb2** is configured to connect to the application database running on the **smarthotelsql** machine.

On the migrated VM **smarthotelweb2**, this configuration needs to be updated to use the Azure SQL Database instead.

> **Note**: You do not need to update any configuration files on **smarthotelweb1** or the **UbuntuWAF** VMs, since the migration has preserved the private IP addresses of all virtual machines they connect with.

1. Navigate to the **smarthotelweb2** VM overview blade, and select **Connect**. Select **Bastion** and connect to the machine with the username **Administrator** and the password **demo!pass123**.

    **Note:** You may have to wait a few minutes and refresh to have the option to enter the credentials. 

    ![Screenshot showing the Azure Bastion connection blade.](images/Exercise3/web2-connect.png)

2. In the **smarthotelweb2** remote desktop session, open Windows Explorer and navigate to the **C:\\inetpub\\SmartHotel.Registration.Wcf** folder. Double-select the **Web.config** file and open with Notepad.

3. Update the **DefaultConnection** setting to connect to your Azure SQL Database.

    You can find the connection string for the Azure SQL Database in the Azure portal by browsing to the database, and selecting **Show database connection strings**.

     ![Screenshot showing the 'Show database connection strings' link for an Azure SQL Database.](images/Exercise3/show-connection-strings.png)

    Copy the **ADO.NET** connection string, and paste into the web.config file on **smarthotelweb2**, replacing the existing connection string.  **Be careful not to overwrite the 'providerName' parameter which is specified after the connection string.**

    > **Note:** You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the connection string there, and then paste into the VM.

    Set the password in the connection string to **demo!pass123**.

    ![Screenshot showing the user ID and Password in the web.config database connection string.](images/Exercise3/web2-connection-string.png)

4. **Save** the `web.config` file and exit your Bastion remote desktop session.

#### Task summary 

In this task, you updated the **smarthotelweb2** configuration to connect to the Azure SQL Database.

### Task 9: Configure the public IP address and test the SmartHotel application

In this task, you will associate a public IP address with the UbuntuWAF VM. This will allow you to verify that the SmartHotel application is running successfully in Azure.

1. Navigate to the **UbuntuWAF** VM blade, select **Networking** under **Settings** on the left, then select the network interface (in bold text). 

    ![Screenshot showing the path to the NIC of the UbuntuWAF VM.](images/Exercise3/waf-nic.png)

2. Select **IP configuration** under **Settings** on the left, then select the IP configuration listed.

    ![Screenshot showing the path to the ipConfig of the UbuntuWAF VM's NIC.](images/Exercise3/waf-ipconfig.png)

3. Set the **Public IP address** to **Associate**, and create a new public IP address named **UbuntuWAF-IP**. Choose a **Basic** tier IP address with **Dynamic** assignment. **Save** your changes.

    ![Screenshot showing the public IP configured on the UbuntuWAF VM.](images/Exercise3/waf-ip.png)

4. Return to the **UbuntuWAF** VM overview blade and copy the **Public IP address** value.

    ![Screenshot showing the IP address for the UbuntuWAF VM.](images/Exercise3/ubuntu-public-ip.png)

5. Open a new browser tab and paste the IP address into the address bar. Verify that the SmartHotel360 application is now available in Azure.

    ![Screenshot showing the SmartHotel application.](images/Exercise3/smarthotel.png)

#### Task summary 

In this task, you assigned a public IP address to the UbuntuWAF VM and verified that the SmartHotel application is now working in Azure.

### Task 10: Post-migration steps

There are a number of post-migration steps that should be completed before the migrated services is ready for production use. These include:

- Installing the Azure VM Agent

- Cleaning up migration resources

- Enabling backup and disaster recovery

- Encrypting VM disks

- Ensuring the network is properly secured

- Ensuring proper subscription governance is in place, such as role-based access control and Azure Policy

- Reviewing recommendations from Azure Advisor and Security Center

In this task you will install the Azure Virtual Machine Agent (VM Agent) on your migrated Azure VMs and clean up any migration resources. The remaining steps are common for any Azure application, not just migrations, and are therefore out of scope for this hands-on lab.

> **Note**: The Microsoft Azure Virtual Machine Agent (VM Agent) is a secure, lightweight process that manages virtual machine (VM) interaction with the Azure Fabric Controller. The VM Agent has a primary role in enabling and executing Azure virtual machine extensions. VM Extensions enable post-deployment configuration of VM, such as installing and configuring software. VM extensions also enable recovery features such as resetting the administrative password of a VM. Without the Azure VM Agent, VM extensions cannot be used.
>
> In this lab, you will install the VM agent on the Azure VMs after migration. Alternatively, you could instead install the agent on the VMs in Hyper-V before migration.

1. In the Azure portal, locate the **smarthotelweb1** VM and open a remote desktop session using Azure Bastion. Log in to the **Administrator** account using password **demo!pass123** (use the 'eyeball' to check the password was entered correctly with your local keyboard mapping).

2. Open a web browser and download the VM Agent from:

    ```s
    https://go.microsoft.com/fwlink/?LinkID=394789
    ```

    **Note**: You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the URL, and then paste into the VM.

3. After the installer has downloaded, run it. Select **Next**, Select **I accept the terms in the License Agreement**, and then **Next** again. Select **Finish**.

    ![Screenshot showing the Windows installer for the Azure VM Agent.](images/Exercise3/vm-agent-win.png)

4. Close the smarthotelweb1 window. Repeat the Azure VM agent installation process on **smarthotelweb2**.

You will now install the Linux version of the Azure VM Agent on the Ubuntu VM. All Linux distributions supports by Azure have integrated the Azure VM Agent into their software repositories, making installation easy in most cases.

5. In the Azure portal, locate the **UbuntuWAF** VM and **Connect** to the VM using Azure Bastion, with the user name **demouser** and password **demo!pass123**. Since this is a Linux VM, Bastion will create an SSH session. You may need to enter the credentials again. 
 
6. In the SSH session, enter the following command:

    ```s
    sudo apt-get install walinuxagent
    ```

    When prompted, enter the password **demo!pass123**. At the *Do you want to continue?* prompt, type **Y** and press **Enter**.

    **Note**: You may need to open the clipboard panel on the left-hand edge of the Bastion window, paste the command, and then paste into the VM.

    ![Screenshot showing the Azure VM Agent install experience on Ubuntu.](images/Exercise3/ubuntu-agent-1.png)

7. Wait for the installer to finish, then close the terminal window and the Ubuntu VM window.

To demonstrate that the VM Agent is installed, we will now execute the 'Run command' feature from the Azure portal. For more information on the VM Agent, see [Windows VM Agent](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows) and [Linux VM Agent](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux).

8. Navigate to the **smarthotelweb1** blade. Under **Operations**, select **Run command**, followed by **IPConfig**, followed by **Run**. After a few seconds, you should see the output of the IPConfig command.

    ![Screenshot showing the Run command feature.](images/Exercise3/run-command.png)

9. As a final step, you will now clean up the resources that were created to support the migration and are no longer needed. These include the Azure Migrate project, the Recovery Service Vault (Azure Site Recovery resource) used by  Azure Migrate: Server Migration, and the Database Migration Service instance. Also included are various secondary resources such as the Log Analytics workspace used by the Dependency Visualization, the storage account used by Azure Migrate: Server Migration, and a Key Vault instance.

    Because all of these temporary resources have been deployed to a separate **AzureMigrateRG** resource group, deleting them is as simple as deleting the resource group. Simply navigate to the resource group blade in the Azure portal, select **Delete resource group** and complete the confirmation prompts.

#### Task summary <!-- omit in toc -->

In this task you installed the Azure Virtual Machine Agent (VM Agent) on your migrated VMs. You also cleaned up the temporary resources created during the migration process.

### Exercise summary <!-- omit in toc -->

In this exercise you migrated the web tier and application tiers of the application from on-premises to Azure using Azure Migrate: Server Migration. Having migrated the virtual machines, you reconfigured the application tier to use the migrated application database hosted in Azure SQL Database, and verified that the migrated application is working end-to-end. You also installed the VM Agent on the migrated virtual machines, and cleaned up migration resources.

## After the hands-on lab

Duration: 10 minutes

### Task 1: Clean up resources

You should complete all of these steps *after* attending the Hands-on lab. Failure to delete the resources created during the lab will result in continued billing.

1. Delete the **SmartHotelHostRG** resource group containing the SmartHotelHost.

2. Delete the **SmartHotelDBRG** resource group containing the Azure SQL Database.

3. Delete the **BastionRG** resource group by first deleting the Bastion resource then deleting the resource group itself. 

4. Delete the **SmartHotelRG** resource group containing the migrated VMs and related infrastructure resources.

5. Delete the **AzureMigrateRG** resource group containing the Azure Migrate resources (if not done already at the end of Exercise 3).

You should follow all steps provided *after* attending the Hands-on-lab.
