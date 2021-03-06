<properties 
	title="Upgrade to the Latest Elastic Scale Client Library" 
	pageTitle="Upgrade to the Latest Elastic Scale Client Library" 
	description="Upgrade instructions using PowerShell and C#" 
	metaKeywords="sharding,elastic scale, Azure SQL DB sharding" 
	services="sql-database" 
	documentationCenter="" 
	manager="stuarto" 
	authors="stuarto"/>

<tags 
	ms.service="sql-database" 
	ms.workload="sql-database" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/05/2015" 
	ms.author="stuarto" />

# Upgrade to the latest Elastic Scale client library

New versions of the Elastic Scale Client Library are   available through [NuGet](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/) and the NuGetPackage Manager interface in Visual Studio. Upgrades contain bug fixes and support for new capabilities of Elastic Scale.

## Upgrade steps

Upgrading requires you to rebuild your application with the new library, as well as change your existing Shard Map Manager metadata stored in your Azure SQL Databases to support new features.

Follow the sequence below to upgrade your applications, the Shard Map Manager database, and the local Shard Map Manager metadata on each shard.  Performing upgrade steps in this order ensures that old versions of the client library are no longer present in your environment when metadata objects are updated, which means that old-version metadata objects won’t be created after upgrade.   

**1. Upgrade your applications.** In Visual Studio, download and reference the latest client library version into all of your Elastic Scale projects; then rebuild and deploy. 

 * In your Visual Studio solution, select **Tools** --> **NuGet Package Manager** -->  **Manage NuGet Packages for Solution**. 
 * In the left panel, select **Updates**, and then select the **Update** button on the package **Azure SQL Database Elastic Scale Client Library** that appears in the window.
	![Upgrade Nuget Pacakges][1]
 
 * Build and Deploy. 

**2. Upgrade your scripts.** If you are using **PowerShell** scripts to manage shards, [download the new library version](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/) and copy it into the directory from which you execute scripts. 

**3. Upgrade your Split-Merge Service.** If you use the Elastic Scale Split-Merge service to reorganize sharded data, [download and deploy the latest version of the Service](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge/). Detailed upgrade steps for the Service can be found [here](http://azure.microsoft.com/en-us/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/). 

**4. Upgrade your Shard Map Manager DBs**. Upgrade the metadata supporting your Shard Maps in Azure SQL Database.  There are two ways you can accomplish this, using PowerShell or C#. Both options are shown below.

***Option 1: Upgrade metadata using PowerShell***

1. Download the latest command-line utility for NuGet from [here](http://nuget.org/nuget.exe) and save to a folder. 

2. Open a Command Prompt, navigate to the same folder, and issue the command:
`nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Client`

3. Navigate to the subfolder containing the new client DLL version you have just downloaded, for example:
`cd .\Microsoft.Azure.SqlDatabase.ElasticScale.Client.0.8.0\lib\net45`

4. Download the Elastic Scale Client Upgrade scriptlet from [Script Center](http://go.microsoft.com/?linkid=9876343), and save it into the same folder containing the DLL.

5. From that folder, run “PowerShell .\upgrade.ps1” from the command prompt and follow the prompts.
 
***Option 2: Upgrade metadata using C#***

Alternatively, create a Visual Studio application that opens your ShardMapManager, iterates over all shards, and performs the metadata upgrade by calling the methods [UpgradeLocalStore](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradelocalstore.aspx) and [UpgradeGlobalStore](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradeglobalstore.aspx) as in this example: 

	ShardMapManager smm =
	   ShardMapManagerFactory.GetSqlShardMapManager
	   (connStr, ShardMapManagerLoadPolicy.Lazy); 
	smm.UpgradeGlobalStore(); 
	
	foreach (ShardLocation loc in
	 smm.GetDistinctShardLocations()) 
	{   
	   smm.UpgradeLocalStore(loc); 
	} 

These techniques for metadata upgrades can be applied multiple times without harm. For example, if an older client version inadvertently creates a shard after you have already updated, you can run upgrade again across all shards to ensure that the latest metadata version is present throughout your infrastructure. 

**Note:**  New versions of the client library published to-date continue to work with prior versions of the Shard Map Manager metadata on Azure SQL DB, and vice-versa.   However to take advantage of some of the new features in the latest client, metadata needs to be upgraded.   Note that metadata upgrades will not affect any user-data or application-specific data, only objects created and used by the Shard Map Manager.  And applications continue to operate through the upgrade sequence described above. 

## Elastic Scale client version history 

**Version 0.8 – March 2015**

* Async support added for data-dependent routing with the new ShardMap.OpenConnectionForKeyAsync methods. 
* Public KeyType property added to ShardMap 
* Added improvements supporting database restore and disaster recovery scenarios for shards 

**Version 0.7 – October 2014** 

Initial Preview version 


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]  


<!--Image references-->
[1]:./media/sql-database-elastic-scale-upgrade-client-library/nuget-upgrade.png
