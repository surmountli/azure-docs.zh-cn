---
title: "Azure Cloud Shell（预览版）快速入门 | Microsoft Docs"
description: "Azure Cloud Shell 快速入门"
services: 
documentationcenter: 
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: 
ms.service: 
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/10/2017
ms.author: juluk
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: b087ce2b7ee71b867e2123a70a2bfbf0fbea1e3b
ms.contentlocale: zh-cn
ms.lasthandoff: 05/10/2017

---

# <a name="quickstart"></a>快速入门
本文档详细介绍如何在 [Azure 门户](https://ms.portal.azure.com/)中使用 Azure Cloud Shell。

## <a name="start-cloud-shell"></a>启动 Cloud Shell
1. 从 Azure 门户的顶部导航栏启动 **Cloud Shell** <br>
![](media/shell-icon.png)
2. 选择用于创建存储帐户和 Azure 文件共享的订阅 <br>
![](media/storage-prompt.png)
3. 选择“创建存储”

> [!TIP]
> 在每个会话中将自动针对 Azure CLI 2.0 对你进行身份验证。

### <a name="set-your-subscription"></a>设置订阅
1. 列出你可以访问的订阅： <br>
`az account list`
2. 设置首选订阅： <br>
`az account set --subscription my-subscription-name`

> [!TIP]
> 将使用 $Home 目录中保存的 `azureProfile.json` 为将来的会话记住订阅。

### <a name="create-a-resource-group"></a>创建资源组
在 WestUS 中创建名为“MyRG”的新资源组： <br>
`az group create -l westus -n MyRG` <br>

### <a name="create-a-linux-vm"></a>创建 Linux VM
在新资源组中创建 Ubuntu VM。 Azure CLI 2.0 将创建 ssh 密钥并使用它们设置 VM。 <br>
`az vm create -n my_vm_name -g MyRG --image UbuntuLTS --generate-ssh-keys`

> [!NOTE]
> 默认情况下，用于对 VM 进行身份验证的公钥和私钥由 Azure CLI 2.0 置于 `/User/.ssh/id_rsa` 和 `/User/.ssh/id_rsa.pub` 中。 .ssh 文件夹保存在附加的 Azure 文件共享的 5 GB 映像中。

此 VM 中的用户名将是你在 Cloud Shell 中使用的用户名 ($User@Azure:)。

### <a name="ssh-into-your-linux-vm"></a>通过 SSH 连接到 Linux VM
1. 在 Azure 门户搜索栏中搜索 VM 名称
2. 单击“连接”并运行：`ssh username@ipaddress`

![](media/sshcmd-copy.png)

建立 SSH 连接后，你应看到 Ubuntu 欢迎提示。
![](media/ubuntu-welcome.png)

## <a name="cleaning-up"></a>清理 
删除资源组及其中的任何资源： <br>
运行 `az group delete -n MyRG`

## <a name="next-steps"></a>后续步骤
[了解如何将存储保存到 Cloud Shell 中](persisting-shell-storage.md) [了解 Azure CLI 2.0] (https://docs.microsoft.com/cli/azure/) [了解 Azure 文件存储] (https://docs.microsoft.com/azure/storage/storage-introduction#file-storage) 

