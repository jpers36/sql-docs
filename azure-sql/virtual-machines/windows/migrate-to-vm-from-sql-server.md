---
title: Migrate a SQL Server database to SQL Server on a virtual machine
description: Learn about how to migrate an on-premises user database to SQL Server on an Azure virtual machine.
author: bluefooted
ms.author: pamela
ms.reviewer: mathoma, randolphwest
ms.date: 05/24/2022
ms.service: virtual-machines-sql
ms.subservice: migration
ms.topic: how-to
tags: azure-service-management
---
# Migrate a SQL Server database to SQL Server on an Azure virtual machine

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

There are many ways to migrate an on-premises SQL Server user database to SQL Server in an Azure virtual machine (VM). This article will briefly discuss various methods and recommend the best method for various scenarios.

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

> [!NOTE]
> SQL Server 2012 is approaching its [end of support life cycle](/lifecycle/products/microsoft-sql-server-2012) for on-premises instances. To extend support, you can either migrate your SQL Server instance to an Azure VM, or buy Extended Security Updates to keep it on-premises. For more information, see [Extend support for SQL Server with Azure](sql-server-extend-end-of-support.md).

## What are the primary migration methods?

The primary migration methods are:

* Perform an on-premises backup using compression, and then manually copy the backup file into the Azure VM.
* Perform a backup to URL and then restore into the Azure VM from the URL.
* Detach the data and log files, copy them to Azure Blob storage, and then attach them to SQL Server in the Azure VM from the URL.
* Convert the on-premises physical machine to a Hyper-V VHD, upload it to Azure Blob storage, and then deploy it as new VM using uploaded VHD.
* Ship the hard drive using the Windows Import/Export Service.
* If you have an Always On Availability Group deployment on-premises, use the [Add Azure Replica Wizard](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-onprem-availability) to create a replica in Azure, failover, and point users to the Azure database instance.
* Use SQL Server [transactional replication](/sql/relational-databases/replication/transactional/transactional-replication) to configure the Azure SQL Server instance as a subscriber, disable replication, and point users to the Azure database instance.

> [!TIP]
> You can also use these same techniques to move databases between SQL Server VMs in Azure. For example, there is no supported way to upgrade a SQL Server gallery-image VM from one version/edition to another. In this case, you should create a new SQL Server VM with the new version/edition, and then use one of the migration techniques in this article to move your databases. 

## Choose a migration method

For best data transfer performance, migrate the database files into the Azure VM using a compressed backup file.

To minimize downtime during the database migration process, use either the Always On option or the transactional replication option.

If it isn't possible to use the above methods, manually migrate your database. Generally, you start with a database backup, follow it with a copy of the database backup into Azure, and then restore the database. You can also copy the database files themselves into Azure and then attach them. There are several methods by which you can accomplish this manual process of migrating a database into an Azure VM.

> [!NOTE]
> When you upgrade to SQL Server 2014 or SQL Server 2016 from older versions of SQL Server, you should consider whether changes are needed. We recommend that you address all dependencies on features not supported by the new version of SQL Server as part of your migration project. For more information on the supported editions and scenarios, see [Upgrade to SQL Server](/sql/database-engine/install-windows/upgrade-sql-server).

The following table lists each of the primary migration methods and discusses when the use of each method is most appropriate.

| Method | Source database version | Destination database version | Source database backup size constraint | Notes |
| --- | --- | --- | --- | --- |
| [Perform an on-premises backup using compression and manually copy the backup file into the Azure virtual machine](#back-up-and-restore) |SQL Server 2005 or greater |SQL Server 2005 or greater |[Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) | This technique is simple and well-tested for moving databases across machines. |
| [Perform a backup to URL and restore into the Azure virtual machine from the URL](#backup-to-url-and-restore-from-url) |SQL Server 2012 SP1 CU2 or greater | SQL Server 2012 SP1 CU2 or greater | < 12.8 TB for SQL Server 2016, otherwise < 1 TB | This method is just another way to move the backup file to the VM using Azure storage. |
| [Detach and then copy the data and log files to Azure Blob storage and then attach to SQL Server in Azure virtual machine from URL](#detach-and-attach-from-a-url) | SQL Server 2005 or greater |SQL Server 2014 or greater | [Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) | Use this method when you plan to [store these files using the Azure Blob storage service](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure) and attach them to SQL Server running in an Azure VM, particularly with very large databases |
| [Convert on-premises machine to Hyper-V VHDs, upload to Azure Blob storage, and then deploy a new virtual machine using uploaded VHD](#convert-to-a-vm-upload-to-a-url-and-deploy-as-a-new-vm) |SQL Server 2005 or greater |SQL Server 2005 or greater |[Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) |Use when [bringing your own SQL Server license](../../azure-sql-iaas-vs-paas-what-is-overview.md), when migrating a database that you'll run on an older version of SQL Server, or when migrating system and user databases together as part of the migration of database dependent on other user databases and/or system databases. |
| [Ship hard drive using Windows Import/Export Service](#ship-a-hard-drive) |SQL Server 2005 or greater |SQL Server 2005 or greater |[Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) |Use the [Windows Import/Export Service](/azure/import-export/storage-import-export-service) when manual copy method is too slow, such as with very large databases |
| [Use the Add Azure Replica Wizard](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-onprem-availability) |SQL Server 2012 or greater |SQL Server 2012 or greater |[Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) |Minimizes downtime, use when you have an Always On on-premises deployment |
| [Use SQL Server transactional replication](/sql/relational-databases/replication/transactional/transactional-replication) |SQL Server 2005 or greater |SQL Server 2005 or greater |[Azure VM storage limit](/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-limits) |Use when you need to minimize downtime and don't have an Always On on-premises deployment |

## Back up and restore

Back up your database with compression, copy the backup to the VM, and then restore the database. If your backup file is larger than 1 TB, you must create a striped set because the maximum size of a VM disk is 1 TB. Use the following general steps to migrate a user database using this manual method:

1. Perform a full database backup to an on-premises location.
2. Create or upload a virtual machine with the desired version of SQL Server.
3. Setup connectivity based on your requirements. See [Connect to a SQL Server Virtual Machine on Azure (Resource Manager)](ways-to-connect-to-sql.md).
4. Copy your backup file(s) to your VM using remote desktop, Windows Explorer, or the copy command from a command prompt.

## Backup to URL and Restore from URL

Instead of backing up to a local file, you can use [Backup to URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) and then Restore from URL to the VM. SQL Server 2016 supports striped backup sets. They're recommended for performance and required to exceed the size limits per blob. For very large databases, the use of the [Windows Import/Export Service](/azure/import-export/storage-import-export-service) is recommended.

## Detach and attach from a URL

Detach your database and log files and transfer them to [Azure Blob storage](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure). Then attach the database from the URL on your Azure VM. Use this method if you want the physical database files to reside in Blob storage, which might be useful for very large databases. Use the following general steps to migrate a user database using this manual method:

1. Detach the database files from the on-premises database instance.
2. Copy the detached database files into Azure Blob storage using the [AZCopy command-line utility](/azure/storage/common/storage-use-azcopy-v10).
3. Attach the database files from the Azure URL to the SQL Server instance in the Azure VM.

## Convert to a VM, upload to a URL, and deploy as a new VM

Use this method to migrate all system and user databases in an on-premises SQL Server instance to an Azure virtual machine. Use the following general steps to migrate an entire SQL Server instance using this manual method:

1. Convert physical or virtual machines to Hyper-V VHDs.
2. Upload VHD files to Azure Storage by using the [Add-AzureVHD cmdlet](/previous-versions/azure/dn495173(v=azure.100)).
3. Deploy a new virtual machine by using the uploaded VHD.

> [!NOTE]
> To migrate an entire application, consider using [Azure Site Recovery](/azure/site-recovery/site-recovery-overview)].

## Ship a hard drive

Use the [Windows Import/Export Service method](/azure/import-export/storage-import-export-service) to transfer large amounts of file data to Azure Blob storage in situations where uploading over the network is prohibitively expensive or not feasible. With this service, you send one or more hard drives containing that data to an Azure data center where your data will be uploaded to your storage account.

## Next steps

For more information, see [SQL Server on Azure Virtual Machines overview](sql-server-on-azure-vm-iaas-what-is-overview.md).

> [!TIP]
> If you have questions about SQL Server virtual machines, see the [Frequently Asked Questions](frequently-asked-questions-faq.yml).

For instructions on creating SQL Server on an Azure Virtual Machine from a captured image, see [Tips & Tricks on ‘cloning’ Azure SQL virtual machines from captured images](/archive/blogs/psssql/tips-tricks-on-cloning-azure-sql-virtual-machines-from-captured-images) on the CSS SQL Server Engineers blog.