---
title: "防止关键 Azure 资源中的更改 | Microsoft Docs"
description: "通过对所有用户和角色应用限制，来防止用户更新或删除特定的资源。"
services: azure-resource-manager
documentationcenter: 
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 53c57e8f-741c-4026-80e0-f4c02638c98b
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/14/2016
ms.author: tomfitz
translationtype: Human Translation
ms.sourcegitcommit: 2a9075f4c9f10d05df3b275a39b3629d4ffd095f
ms.openlocfilehash: de8137a69ccc2028a7dcbff491573f36640bdc50


---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>锁定资源以防止意外更改 
管理员可能需要锁定订阅、资源组或资源，以防止组织中的其他用户意外删除或修改关键资源。 可以将锁定级别设置为 **CanNotDelete** 或 **ReadOnly**。 

* **CanNotDelete** 表示经授权的用户仍可读取和修改资源，但不能删除资源。 
* **ReadOnly** 表示经授权的用户可以读取资源，但不能删除或更新资源。 应用此锁类似于将所有经授权的用户限制于使用“读者”角色授予的权限。 

Resource Manager 锁仅适用于管理平面内发生的操作，包括发送到 `https://management.azure.com` 的操作。 锁不会限制资源如何执行各自的函数。 资源更改将受到限制，但资源操作不受限制。 例如，SQL 数据库上的 ReadOnly 锁将阻止删除或修改该数据库，但不会阻止创建、更新或删除该数据库中的数据。 允许数据事务，因为这些操作不会发送到 `https://management.azure.com`。

应用 **ReadOnly** 可能会导致意外结果，因为看起来好像读取操作的某些操作实际上需要其他操作。 例如，在存储帐户上放置 **ReadOnly** 锁将阻止所有用户列出密钥。 列出密钥操作通过 POST 请求进行处理，因为返回的密钥可用于写入操作。 另举一例，在应用服务资源上放置 **ReadOnly** 锁将阻止 Visual Studio 服务器资源管理器显示资源文件，因为该交互需要写访问权限。

与基于角色的访问控制不同，你可以使用管理锁来对所有用户和角色应用限制。 若要了解如何为用户和角色设置权限，请参阅 [Azure 基于角色的访问控制](../active-directory/role-based-access-control-configure.md)。

在父范围应用锁时，该范围内所有资源都将继承相同的锁。 即使是之后添加的资源也会从父作用域继承该锁。 继承中限制性最强的锁优先执行。

## <a name="who-can-create-or-delete-locks-in-your-organization"></a>谁可以在组织中创建或删除锁
若要创建或删除管理锁，必须有权执行 `Microsoft.Authorization/*` 或 `Microsoft.Authorization/locks/*` 操作。 在内置角色中，只有“所有者”和“用户访问管理员”有权执行这些操作。

## <a name="creating-a-lock-through-the-portal"></a>通过门户创建锁
[!INCLUDE [resource-manager-lock-resources](../../includes/resource-manager-lock-resources.md)]

## <a name="creating-a-lock-in-a-template"></a>在模板上创建锁
以下示例演示在存储帐户上创建锁的模板。 要对其应用锁的存储帐户将以参数形式提供。 锁名是通过将包含 **/Microsoft.Authorization/** 的资源名称与锁名连接起来创建的（本例中为 **myLock**）。

提供的类型特定于资源类型。 对于存储，将类型设置为“Microsoft.Storage/storageaccounts/providers/locks”。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "lockedResource": {
          "type": "string"
        }
      },
      "resources": [
        {
          "name": "[concat(parameters('lockedResource'), '/Microsoft.Authorization/myLock')]",
          "type": "Microsoft.Storage/storageAccounts/providers/locks",
          "apiVersion": "2015-01-01",
          "properties": {
            "level": "CannotDelete"
          }
        }
      ]
    }

## <a name="creating-a-lock-with-rest-api"></a>使用 REST API 创建锁
可以使用[管理锁的 REST API](https://docs.microsoft.com/rest/api/resources/managementlocks) 锁定已部署的资源。 REST API 使您可以创建和删除锁，并且检索有关现有锁的信息。

若要创建一个锁，请运行：

    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}

作用域可能是订阅、资源组或资源。 锁名称可以是您想要对该锁使用的任何称谓。 对于 api-version，请使用 **2015-01-01**。

在请求中，包括指定锁属性的 JSON 对象。

    {
      "properties": {
        "level": "CanNotDelete",
        "notes": "Optional text notes."
      }
    } 


## <a name="creating-a-lock-with-azure-powershell"></a>使用 Azure PowerShell 创建一个锁
可以使用 **New-AzureRmResourceLock** 在 Azure PowerShell 中锁定已部署的资源，如以下示例中所示。

    New-AzureRmResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup

Azure PowerShell 提供了其他用于使用锁的命令，例如，**Set-AzureRmResourceLock** 用于更新锁，**Remove-AzureRmResourceLock** 用于删除锁。

## <a name="next-steps"></a>后续步骤
* 有关使用资源锁的详细信息，请参阅[锁定 Azure 资源](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/06/18/lock-down-your-azure-resources.aspx)
* 有关使用逻辑方式组织资源的信息，请参阅[使用标记来组织资源](resource-group-using-tags.md)
* 若要更改资源位于哪个资源组，请参阅[将资源移到新的资源组](resource-group-move-resources.md)
* 可以使用自定义策略对订阅应用限制和约定。 有关详细信息，请参阅 [使用策略来管理资源和控制访问](resource-manager-policy.md)。
* 有关企业可如何使用 Resource Manager 有效管理订阅的指南，请参阅 [Azure 企业基架 - 出于合规目的监管订阅](resource-manager-subscription-governance.md)。




<!--HONumber=Jan17_HO4-->


