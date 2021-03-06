---
title: "在 Azure 中采用 Linux VM 上的策略强制执行安全措施 | Microsoft Docs"
description: "如何将策略应用到 Azure Resource Manager Linux 虚拟机"
services: virtual-machines-linux
documentationcenter: 
author: singhkays
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 06778ab4-f8ff-4eed-ae10-26a276fc3faa
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 04/13/2016
ms.author: singhkay
translationtype: Human Translation
ms.sourcegitcommit: 303cb9950f46916fbdd58762acd1608c925c1328
ms.openlocfilehash: b48a4e2fa913b865cf4b57693ef281e446541328
ms.lasthandoff: 04/04/2017


---
# <a name="apply-security-and-policies-to-linux-vms-with-azure-resource-manager"></a>使用 Azure Resource Manager 将安全措施和策略应用至 Linux VM
通过使用策略，组织可以在整个企业中强制实施各种约定和规则。 强制实施所需行为有助于消除风险，同时为组织的成功做出贡献。 在本文中，我们将介绍如何使用 Azure Resource Manager 策略来为组织中的虚拟机定义所需行为。

下面概述了实现此目的的步骤

1. Azure Resource Manager 策略 101
2. 为虚拟机定义策略
3. 创建策略
4. 应用策略

## <a name="azure-resource-manager-policy-101"></a>Azure Resource Manager 策略 101
若要开始使用 Azure Resource Manager 策略，我们建议你先阅读以下文章，然后继续执行本文中的步骤。 以下文章介绍了策略的基本定义和结构以及如何评估策略，并提供了策略定义的各种示例。

* [使用策略来管理资源和控制访问](../../azure-resource-manager/resource-manager-policy.md)

## <a name="define-a-policy-for-your-virtual-machine"></a>为虚拟机定义策略
企业中常用的一种方案可能是，只允许其用户在经测试可与 LOB 应用程序兼容的特定操作系统中创建虚拟机。 使用 Azure Resource Manager 策略可以通过几个步骤完成此任务。
在此策略示例中，我们将只允许创建 Ubuntu 14.04.2-LTS 虚拟机。 策略定义如下所示

```
"if": {
  "allOf": [
    {
      "field": "type",
      "equals": "Microsoft.Compute/virtualMachines"
    },
    {
      "not": {
        "allOf": [
          {
            "field": "Microsoft.Compute/virtualMachines/imagePublisher",
            "equals": "Canonical"
          },
          {
            "field": "Microsoft.Compute/virtualMachines/imageOffer",
            "equals": "UbuntuServer"
          },
          {
            "field": "Microsoft.Compute/virtualMachines/imageSku",
            "equals": "14.04.2-LTS"
          }
        ]
      }
    }
  ]
},
"then": {
  "effect": "deny"
}
```

可以轻松修改上述策略，以允许在虚拟机部署中使用经过以下更改的任何 Ubuntu LTS 映像

```
{
  "field": "Microsoft.Compute/virtualMachines/imageSku",
  "like": "*LTS"
}
```

#### <a name="virtual-machine-property-fields"></a>虚拟机属性字段
下表描述了可在策略定义中用作字段的虚拟机属性。 有关策略字段的详细信息，请参阅[使用策略来管理资源和控制访问](../../resource-manager-policy.md)。

| 字段名称 | 说明 |
| --- | --- |
| imagePublisher |指定映像的发布者 |
| imageOffer |指定所选映像发布者的产品 |
| imageSku |指定所选产品的 SKU |
| imageVersion |指定所选 SKU 的映像版本 |

## <a name="create-the-policy"></a>创建策略
可以直接使用 REST API 或 PowerShell cmdlet 轻松创建策略。 可以详细了解如何[创建和分配策略](../../resource-manager-policy.md)。

## <a name="apply-the-policy"></a>应用策略
创建策略后，需要根据定义的范围来应用它。 范围可以是订阅、资源组甚至资源。 可以详细了解如何[创建和分配策略](../../resource-manager-policy.md)。

