---
title: "使用 Azure PowerShell 管理 Azure 磁盘 | Microsoft Docs"
description: "教程 - 使用 Azure PowerShell 管理 Azure 磁盘"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: neilpeterson
manager: timlt
editor: tysonn
tags: azure-service-management
ms.assetid: 
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/21/2017
ms.author: nepeters
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: d68fc54f3fd08b3d7d66a9cd1ddb91c340191468
ms.contentlocale: zh-cn
ms.lasthandoff: 05/03/2017

---

# <a name="manage-azure-disks-with-powershell"></a>使用 PowerShell 管理 Azure 磁盘

本教程介绍了不同类型的 VM 磁盘、如何选择磁盘配置，以及如何创建磁盘并将其附加到 Azure VM。 本教程还讲解了如何拍摄磁盘快照。  

可使用最新版 [Azure PowerShell](/powershell/azure/overview) 模块完成本教程中的步骤。

## <a name="default-azure-disks"></a>默认 Azure 磁盘

创建 Azure 虚拟机后，将自动向此虚拟机附加两个磁盘。 

操作系统磁盘 - 操作系统磁盘大小可达 1 TB，并可托管 VM 操作系统。  OS 磁盘默认分配有一个 c: 驱动器号。 已针对 OS 性能优化了 OS 磁盘的磁盘缓存配置。 OS 磁盘不得承载应用程序或数据。 对于应用程序和数据，请使用数据磁盘，详情请参见本文稍后部分。

临时磁盘- 临时磁盘使用 VM 所在的 Azure 主机上的固态驱动器。 临时磁盘具有高性能，可用于临时数据处理等操作。 但是，如果将 VM 移动到新的主机，临时磁盘上存储的数据都将被删除。 临时磁盘的大小由 VM 大小决定。 临时磁盘默认分配有一个 d: 驱动器号。

### <a name="temporary-disk-sizes"></a>临时磁盘大小

| 类型 | VM 大小 | 临时磁盘大小上限 |
|----|----|----|
| [常规用途](sizes-general.md) | A 和 D 系列 | 800 |
| [计算优化](sizes-compute.md) | F 系列 | 800 |
| [内存优化](../virtual-machines-windows-sizes-memory.md) | D 和 G 系列 | 6144 |
| [存储优化](../virtual-machines-windows-sizes-storage.md) | L 系列 | 5630 |
| [GPU](sizes-gpu.md) | N 系列 | 1440 |
| [高性能](sizes-hpc.md) | A 和 H 系列 | 2000 |

## <a name="azure-data-disks"></a>Azure 数据磁盘

可添加额外的数据磁盘，用于安装应用程序和存储数据。 在任何需要持久和灵敏数据存储的情况下，都应使用数据磁盘。 每个数据磁盘的最大容量为 1 TB。 虚拟机的大小决定可附加到 VM 的数据磁盘数。 每个 VM 核心可附加两个数据磁盘。 

### <a name="max-data-disks-per-vm"></a>每个 VM 的最大数据磁盘数

| 类型 | VM 大小 | 每个 VM 的最大数据磁盘数 |
|----|----|----|
| [常规用途](sizes-general.md) | A 和 D 系列 | 32 |
| [计算优化](sizes-compute.md) | F 系列 | 32 |
| [内存优化](../virtual-machines-windows-sizes-memory.md) | D 和 G 系列 | 64 |
| [存储优化](../virtual-machines-windows-sizes-storage.md) | L 系列 | 64 |
| [GPU](sizes-gpu.md) | N 系列 | 48 |
| [高性能](sizes-hpc.md) | A 和 H 系列 | 32 |

## <a name="vm-disk-types"></a>VM 磁盘类型

Azure 提供两种类型的磁盘。

### <a name="standard-disk"></a>标准磁盘

标准存储受 HDD 支持，可以在确保性能的同时提供经济高效的存储。 标准磁盘适用于经济高效的开发和测试工作负荷。

### <a name="premium-disk"></a>高级磁盘

高级磁盘由基于 SSD 的高性能、低延迟磁盘提供支持。 完美适用于运行生产工作负荷的 VM。 高级存储支持 DS 系列、DSv2 系列、GS 系列和 FS 系列 VM。 高级磁盘分为 3 种类型（P10、P20 和 P30），磁盘大小决定磁盘类型。 选择时，磁盘大小值舍入为下一类型。 例如，大小在 128 GB 以下的磁盘类型为 P10，大小在 129 和 512 之间的磁盘类型为 P20，大小在 512 以上的磁盘类型为 P30。 

### <a name="premium-disk-performance"></a>高级磁盘性能

|高级存储磁盘类型 | P10 | P20 | P30 |
| --- | --- | --- | --- |
| 磁盘大小（向上舍入） | 128 GB | 512 GB | 1,024 GB (1 TB) |
| 每个磁盘的 IOPS | 500 | 2,300 | 5,000 |
每个磁盘的吞吐量 | 100 MB/秒 | 150 MB/秒 | 200 MB/秒 |

尽管上表确定了每个磁盘的最大 IOPS，但还可通过条带化多个数据磁盘实现更高级别的性能。 例如，可向 Standard_GS5 VM 附加 64 个数据磁盘。 如果这些磁盘的大小都为 P30，则最大可实现 80,000 IOPS。 若要详细了解每个 VM 的最大 IOPS，请参阅 [Linux VM 大小](./sizes.md)。

## <a name="create-and-attach-disks"></a>创建并附加磁盘

若要完成本教程中的示例，必须具备现有虚拟机。 如果需要，此[脚本示例](../scripts/virtual-machines-windows-powershell-sample-create-vm.md)可替你创建一个虚拟机。 通过教程操作时，根据需要替换资源组和 VM 名称。

使用 [New-AzureRmDiskConfig](/powershell/module/azurerm.compute/new-azurermdiskconfig) 创建初始配置。 以下示例配置大小为 128 GB 的磁盘。

```powershell
$diskConfig = New-AzureRmDiskConfig -Location EastUS -CreateOption Empty -DiskSizeGB 128
```

使用 [New-AzureRmDisk](/powershell/module/azurerm.compute/new-azurermdisk) 命令创建数据磁盘。

```powershell
$dataDisk = New-AzureRmDisk -ResourceGroupName myResourceGroup -DiskName myDataDisk -Disk $diskConfig
```

使用 [Get-AzureRmVM](/powershell/module/azurerm.compute/get-azurermvm) 命令获取要向其添加数据磁盘的虚拟机。

```powershell
$vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM
```

使用 [Add-AzureRmVMDataDisk](/powershell/module/azurerm.compute/add-azurermvmdatadisk) 命令向虚拟机配置添加数据磁盘。

```powershell
$vm = Add-AzureRmVMDataDisk -VM $vm -Name myDataDisk -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 1
```

使用 [Update-AzureRmVM](/powershell/module/azurerm.compute/add-azurermvmdatadisk) 命令更新虚拟机。

```powershell
Update-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm
```

## <a name="prepare-data-disks"></a>准备数据磁盘

将磁盘附加到虚拟机后，需要将操作系统配置为使用该磁盘。 以下示例演示如何手动配置添加到 VM 的第一个磁盘。 还可使用[自定义脚本扩展](./tutorial-automate-vm-deployment.md)自动执行此过程。

### <a name="manual-configuration"></a>手动配置

创建与虚拟机的 RDP 连接。 打开 PowerShell 并运行此脚本。

```powershell
Get-Disk | Where partitionstyle -eq 'raw' | `
Initialize-Disk -PartitionStyle MBR -PassThru | `
New-Partition -AssignDriveLetter -UseMaximumSize | `
Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false
```

## <a name="next-steps"></a>后续步骤

在本教程中，你已了解 VM 磁盘。 转到下一教程，了解如何自动配置 VM。

[自动配置 VM](./tutorial-automate-vm-deployment.md)

