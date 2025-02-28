---
title: Initiate a storage account failover
titleSuffix: Azure Storage
description: Learn how to initiate an account failover in the event that the primary endpoint for your storage account becomes unavailable. The failover updates the secondary region to become the primary region for your storage account.
services: storage
author: jimmart-dev

ms.service: storage
ms.topic: how-to
ms.date: 07/22/2022
ms.author: jammart
ms.subservice: common 
ms.custom: devx-track-azurepowershell
---

# Initiate a storage account failover

If the primary endpoint for your geo-redundant storage account becomes unavailable for any reason, you can initiate an account failover. An account failover updates the secondary endpoint to become the primary endpoint for your storage account. Once the failover is complete, clients can begin writing to the new primary region. Forced failover enables you to maintain high availability for your applications.

This article shows how to initiate an account failover for your storage account using the Azure portal, PowerShell, or Azure CLI. To learn more about account failover, see [Disaster recovery and storage account failover](storage-disaster-recovery-guidance.md).

> [!WARNING]
> An account failover typically results in some data loss. To understand the implications of an account failover and to prepare for data loss, review [Understand the account failover process](storage-disaster-recovery-guidance.md#understand-the-account-failover-process).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## Prerequisites

Before you can perform an account failover on your storage account, make sure that your storage account is configured for geo-replication. Your storage account can use any of the following redundancy options:

- Geo-redundant storage (GRS) or read-access geo-redundant storage (RA-GRS)
- Geo-zone-redundant storage (GZRS) or read-access geo-zone-redundant storage (RA-GZRS)

For more information about Azure Storage redundancy, see [Azure Storage redundancy](storage-redundancy.md).

Keep in mind that the following features and services are not supported for account failover:

- Azure File Sync does not support storage account failover. Storage accounts containing Azure file shares being used as cloud endpoints in Azure File Sync should not be failed over. Doing so will cause sync to stop working and may also cause unexpected data loss in the case of newly tiered files.
- Storage accounts that have hierarchical namespace enabled (such as for Data Lake Storage Gen2) are not supported at this time.
- A storage account containing premium block blobs cannot be failed over. Storage accounts that support premium block blobs do not currently support geo-redundancy.
- A storage account containing any [WORM immutability policy](../blobs/immutable-storage-overview.md) enabled containers cannot be failed over. Unlocked/locked time-based retention or legal hold policies prevent failover in order to maintain compliance.

## Initiate the failover

## [Portal](#tab/azure-portal)

To initiate an account failover from the Azure portal, follow these steps:

1. Navigate to your storage account.
1. Under **Settings**, select **Geo-replication**. The following image shows the geo-replication and failover status of a storage account.

    :::image type="content" source="media/storage-initiate-account-failover/portal-failover-prepare.png" alt-text="Screenshot showing geo-replication and failover status":::

1. Verify that your storage account is configured for geo-redundant storage (GRS) or read-access geo-redundant storage (RA-GRS). If it's not, then select **Configuration** under **Settings** to update your account to be geo-redundant.
1. The **Last Sync Time** property indicates how far the secondary is behind from the primary. **Last Sync Time** provides an estimate of the extent of data loss that you will experience after the failover is completed. For more information about checking the **Last Sync Time** property, see [Check the Last Sync Time property for a storage account](last-sync-time-get.md).
1. Select **Prepare for failover**.
1. Review the confirmation dialog. When you are ready, enter **Yes** to confirm and initiate the failover.

    :::image type="content" source="media/storage-initiate-account-failover/portal-failover-confirm.png" alt-text="Screenshot showing confirmation dialog for an account failover":::

## [PowerShell](#tab/azure-powershell)

To use PowerShell to initiate an account failover, install the [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage) module, version 2.0.0 or later. For more information about installing Azure PowerShell, see [Install the Azure Az PowerShell module](/powershell/azure/install-az-ps).

To initiate an account failover from PowerShell, call the following command:

```powershell
Invoke-AzStorageAccountFailover -ResourceGroupName <resource-group-name> -Name <account-name>
```

## [Azure CLI](#tab/azure-cli)

To use Azure CLI to initiate an account failover, call the following commands:

```azurecli-interactive
az storage account show \ --name accountName \ --expand geoReplicationStats
az storage account failover \ --name accountName
```

---

## Important implications of account failover

When you initiate an account failover for your storage account, the DNS records for the secondary endpoint are updated so that the secondary endpoint becomes the primary endpoint. Make sure that you understand the potential impact to your storage account before you initiate a failover.

To estimate the extent of likely data loss before you initiate a failover, check the **Last Sync Time** property. For more information about checking the **Last Sync Time** property, see [Check the Last Sync Time property for a storage account](last-sync-time-get.md).

The time it takes to failover after initiation can vary though typically less than one hour.

After the failover, your storage account type is automatically converted to locally redundant storage (LRS) in the new primary region. You can re-enable geo-redundant storage (GRS) or read-access geo-redundant storage (RA-GRS) for the account. Note that converting from LRS to GRS or RA-GRS incurs an additional cost. The cost is due to the network egress charges to re-replicate the data to the new secondary region. For additional information, see [Bandwidth Pricing Details](https://azure.microsoft.com/pricing/details/bandwidth/).

After you re-enable GRS for your storage account, Microsoft begins replicating the data in your account to the new secondary region. Replication time depends on many factors, which include:

- The number and size of the objects in the storage account. Many small objects can take longer than fewer and larger objects.
- The available resources for background replication, such as CPU, memory, disk, and WAN capacity. Live traffic takes priority over geo replication.
- If using Blob storage, the number of snapshots per blob.
- If using Table storage, the [data partitioning strategy](/rest/api/storageservices/designing-a-scalable-partitioning-strategy-for-azure-table-storage). The replication process can't scale beyond the number of partition keys that you use.

## Next steps

- [Disaster recovery and storage account failover](storage-disaster-recovery-guidance.md)
- [Check the Last Sync Time property for a storage account](last-sync-time-get.md)
- [Use geo-redundancy to design highly available applications](geo-redundant-design.md)
- [Tutorial: Build a highly available application with Blob storage](../blobs/storage-create-geo-redundant-storage.md)
