---
title: "Azure CLI 脚本示例 - 装载操作系统磁盘 | Microsoft 文档"
description: "Azure CLI 脚本示例 - 装载操作系统磁盘"
services: virtual-machines-linux
documentationcenter: virtual-machines
author: neilpeterson
manager: timlt
editor: tysonn
tags: azure-service-management
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/27/2017
ms.author: nepeters
ms.translationtype: Human Translation
ms.sourcegitcommit: 6ea03adaabc1cd9e62aa91d4237481d8330704a1
ms.openlocfilehash: 940171e5289ce735ad1cdf0cd67759e9069ea135
ms.contentlocale: zh-cn
ms.lasthandoff: 04/06/2017

---

# <a name="troubleshoot-a-vms-operating-system-disk"></a>对 VM 操作系统磁盘进行故障排除

此脚本将失败或有问题的虚拟机的操作系统磁盘作为数据磁盘装载到第二个虚拟机。 排查磁盘问题或恢复数据时，此脚本会很有用。 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

[!code-azurecli[主要](../../../cli_scripts/virtual-machine/mount-os-disk/mount-os-disk.sh "快速创建 VM")]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az vm show](https://docs.microsoft.com/cli/azure/vm#show) | 返回虚拟机列表。 在此示例中，查询选项用于返回虚拟机操作系统磁盘。 然后，将此值添加到名为“uri”的变量。 |
| [az vm delete](https://docs.microsoft.com/cli/azure/vm#delete) | 删除虚拟机。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#create) | 创建虚拟机。  |
| [az vm disk attach](https://docs.microsoft.com/cli/azure/vm/disk#attach) | 将磁盘附加到虚拟机。 |
| [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm#list-ip-addresses) | 返回虚拟机的 IP 地址。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure/overview)。

可以在 [Azure Linux VM 文档](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他虚拟机 CLI 脚本示例。

