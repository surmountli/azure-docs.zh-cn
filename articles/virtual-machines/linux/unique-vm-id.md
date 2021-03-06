---
title: "获取 Azure Linux VM ID | Microsoft Docs"
description: "介绍如何获取和使用 Azure Linux VM 唯一 ID。"
services: virtual-machines-linux
documentationcenter: virtual-machines
author: kmouss
manager: timlt
editor: 
ms.assetid: 136c5d28-ff6b-4466-b27f-7a29785b5d27
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 01/23/2017
ms.author: kmouss
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 258ce425d5692730011cf2f4468dc0ba77f4cb79
ms.lasthandoff: 04/03/2017


---
# <a name="accessing-and-using-azure-vm-unique-id"></a>访问和使用 Azure VM 唯一 ID
Azure VM 的唯一 ID 是在所有 Azure IaaS VM 的 SMBIOS 中编码并存储的一个 128 位标识符，目前可以使用平台的 BIOS 命令读取。

Azure VM 唯一 ID 是只读属性。 在重新启动关机（计划内或计划外）、开始/停止取消分配、服务修复或就地还原之后，Azure 唯一 VM ID 不会更改。 但是，如果 VM 是快照，并且是为了创建新的实例而复制的，将配置新的 Azure VM ID。

> [!NOTE]
> 如果你有旧的 VM 并且自此新功能推出（2014 年 9 月 18 日）之后运行了这些 VM，请重新启动 VM 以自动获取 Azure 唯一 ID。
> 
> 

若要从 VM 内部访问 Azure 唯一 VM ID，请执行以下操作：

## <a name="create-a-vm"></a>创建 VM
有关详细信息，请参阅 [Create a Virtual Machine](../windows/creation-choices.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)（创建虚拟机）

## <a name="connect-to-the-vm"></a>连接到 VM
有关详细信息，请参阅 [SSH from Linux](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)（从 Linux 执行 SSH）

## <a name="query-vm-unique-id"></a>查询 VM 唯一 ID
命令（示例使用 **Ubuntu**）：

```bash
sudo dmidecode | grep UUID
```

示例的预期结果：

```bash
UUID: 090556DA-D4FA-764F-A9F1-63614EDA019A
```

由于 Big Endian 位顺序的原因，此案例中实际的唯一 VM ID 将是：

```bash
DA 56 05 09 – FA D4 – 4f 76 - A9F1-63614EDA019A
```

无论 VM 是在 Azure 上还是本地运行，都可以在不同的方案中使用 Azure VM 唯一 ID，并且可以帮助在 Azure IaaS 部署中可能有的授权、报告或一般跟踪要求。 构建应用程序并在 Azure 上进行认证的许多独立软件供应商可能需要在整个生命周期找出 Azure VM，并辨别 VM 是在 Azure、本地，还是其他云提供程序上运行。 例如，此平台标识符可帮助检测软件是否获得正确授权，或帮助将任何 VM 数据及其源相关联，以帮助为适当的平台设置适当的指标，并在其他用途之间跟踪和关联这些指标。


