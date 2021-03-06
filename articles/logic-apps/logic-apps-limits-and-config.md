---
title: "逻辑应用限制和配置 | Microsoft Docs"
description: "适用于逻辑应用的服务限制和配置值的概述。"
services: logic-apps
documentationcenter: .net,nodejs,java
author: jeffhollan
manager: anneta
editor: 
ms.assetid: 75b52eeb-23a7-47dd-a42f-1351c6dfebdc
ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/23/2016
ms.author: jehollan
ms.translationtype: Human Translation
ms.sourcegitcommit: f6006d5e83ad74f386ca23fe52879bfbc9394c0f
ms.openlocfilehash: 2a270ba8ae17077c55c6b1473d4955dfb5f79ca1
ms.contentlocale: zh-cn
ms.lasthandoff: 05/03/2017


---
# <a name="logic-app-limits-and-configuration"></a>逻辑应用限制和配置

以下是有关适用于 Azure 逻辑应用的当前限制和配置详细信息的信息。

## <a name="limits"></a>限制

### <a name="http-request-limits"></a>HTTP 请求限制

这些是针对单个 HTTP 请求和/或连接器调用的限制

#### <a name="timeout"></a>超时

|Name|限制|说明|
|----|----|----|
|请求超时|120 秒|[异步模式](../logic-apps/logic-apps-create-api-app.md)或 [Until 循环](logic-apps-loops-and-scopes.md)可以根据需要进行补偿|

#### <a name="message-size"></a>消息大小

|Name|限制|说明|
|----|----|----|
|消息大小|100 MB|某些连接器和 API 可能不支持 100MB |
|表达式计算限制|131,072 个字符|`@concat()`、`@base64()`、`string` 不能超过此长度|

#### <a name="retry-policy"></a>重试策略

|Name|限制|说明|
|----|----|----|
|重试次数|4|可以使用[重试策略参数](https://msdn.microsoft.com/en-us/library/azure/mt643939.aspx)进行配置|
|重试最大延迟|1 小时	|可以使用[重试策略参数](https://msdn.microsoft.com/en-us/library/azure/mt643939.aspx)进行配置|
|重试最小延迟|5 秒|可以使用[重试策略参数](https://msdn.microsoft.com/en-us/library/azure/mt643939.aspx)进行配置|

### <a name="run-duration-and-retention"></a>运行持续时间和保留期

这些是针对单个逻辑应用运行的限制。

|Name|限制|说明|
|----|----|----|
|运行持续时间|90 天||
|存储保留期|90 天|这从运行开始时间开始计算|
|最小重复间隔|1 秒|| 对于带有应用服务计划的逻辑应用为 15 秒
|最大重复间隔|500 天||


### <a name="looping-and-debatching-limits"></a>循环和解除批处理限制

这些是针对单个逻辑应用运行的限制。

|Name|限制|说明|
|----|----|----|
|ForEach 项|100,000|可以使用[查询操作](../connectors/connectors-native-query.md)根据需要筛选更大数组|
|Until 迭代|5,000||
|SplitOn 项|100,000||
|ForEach 并行度|20|可以通过将 `"operationOptions": "Sequential"` 添加到 `foreach` 操作来设置为顺序 foreach|


### <a name="throughput-limits"></a>吞吐量限制

这些是针对单个逻辑应用实例的限制。 

|Name|限制|说明|
|----|----|----|
|每 5 分钟执行的操作数 |100,000|可以根据需要在多个应用之间分配工作负荷|
|运行时终结点读取每 5 分钟调用一次 |60,000|可以根据需要在多个应用之间分配工作负荷|
|运行时终结点调用每 5 分钟调用一次 |45,000|可以根据需要在多个应用之间分配工作负荷|
|阻止并发调用的运行时终结点 |~1,000|减少并发请求数，或根据需要减少持续时间|

如果你在正常处理中需要超过此限制，或想要运行在一段时间内可能超过此限制的负载测试，请[与我们联系](mailto://logicappsemail@microsoft.com)，以便我们可以帮助满足你的需求。

### <a name="definition-limits"></a>定义限制

这些是针对单个逻辑应用定义的限制。

|Name|限制|说明|
|----|----|----|
|每个工作流的操作数|250|可以添加嵌套工作流以根据需要对此进行扩展|
|允许操作嵌套深度|5|可以添加嵌套工作流以根据需要对此进行扩展|
|每个订阅每个区域的工作流数|1000||
|每个工作流的触发数|10||
|每个表达式的最大字符数|8,192||
|最大 `trackedProperties` 大小（以字符为单位）|16,000|
|`action`/`trigger` 名称限制|80||
|`description` 长度限制|256||
|`parameters` 限制|50||
|`outputs` 限制|10||

### <a name="integration-account-limits"></a>集成帐户限制

这些是对添加到集成帐户的项目的限制

|Name|限制|说明|
|----|----|----|
|架构|8MB|可以使用 [blob URI](logic-apps-enterprise-integration-schemas.md) 上传大于 2 MB 的文件 |
|映射（XSLT 文件）|2MB| |
|运行时终结点读取每 5 分钟调用一次 |60,000|可以根据需要在多个帐户之间分配工作负荷|
|运行时终结点调用每 5 分钟调用一次 |90,000|可以根据需要在多个帐户之间分配工作负荷|
|阻止并发调用的运行时终结点 |~1,000|减少并发请求数，或根据需要减少持续时间|

### <a name="b2b-protocols-as2-x12-edifact-message-size"></a>B2B 协议（AS2、X12、EDIFACT）消息大小

这些是对 B2B 协议的限制

|Name|限制|说明|
|----|----|----|
|AS2|50MB|适用于解码和编码|
|X12|50MB|适用于解码和编码|
|EDIFACT|50MB|适用于解码和编码|

## <a name="configuration"></a>配置

### <a name="ip-address"></a>IP 地址

#### <a name="logic-app-service"></a>逻辑应用服务

直接从逻辑应用（即通过 [HTTP](../connectors/connectors-native-http.md) 或 [HTTP + Swagger](../connectors/connectors-native-http-swagger.md)）或从其他 HTTP 请求进行的调用来自下面指定的 IP 地址：

|逻辑应用区域|出站 IP|
|-----|----|
|澳大利亚东部|13.75.153.66、104.210.89.222、104.210.89.244、13.75.149.4、104.210.91.55、104.210.90.241|
|澳大利亚东南部|13.73.115.153、40.115.78.70、40.115.78.237、13.73.114.207、13.77.3.139、13.70.159.205|
|巴西南部|191.235.86.199、191.235.95.229、191.235.94.220、191.235.82.221、191.235.91.7、191.234.182.26|
|加拿大中部|52.233.29.92,52.228.39.241,52.228.39.244|
|加拿大东部|52.232.128.155,52.229.120.45,52.229.126.25|
|印度中部|52.172.157.194、52.172.184.192、52.172.191.194、52.172.154.168、52.172.186.159、52.172.185.79|
|美国中部|13.67.236.76、40.77.111.254、40.77.31.87、13.67.236.125、104.208.25.27、40.122.170.198|
|东亚|168.63.200.173、13.75.89.159、23.97.68.172、13.75.94.173、40.83.127.19、52.175.33.254|
|美国东部|137.135.106.54、40.117.99.79、40.117.100.228、13.92.98.111、40.121.91.41、40.114.82.191|
|美国东部 2|40.84.25.234、40.79.44.7、40.84.59.136、40.84.30.147、104.208.155.200、104.208.158.174|
|日本东部|13.71.146.140、13.78.84.187、13.78.62.130、13.71.158.3、13.73.4.207、13.71.158.120|
|日本西部|40.74.140.173、40.74.81.13、40.74.85.215、40.74.140.4、104.214.137.243、138.91.26.45|
|美国中北部|168.62.249.81、157.56.12.202、65.52.211.164、168.62.248.37、157.55.210.61、157.55.212.238|
|欧洲北部|13.79.173.49、52.169.218.253、52.169.220.174、40.113.12.95、52.178.165.215、52.178.166.21|
|美国中南部|13.65.98.39、13.84.41.46、13.84.43.45、104.210.144.48、13.65.82.17、13.66.52.232|
|亚洲东南部|52.163.93.214、52.187.65.81、52.187.65.155、13.76.133.155、52.163.228.93、52.163.230.166|
|印度南部|52.172.9.47、52.172.49.43、52.172.51.140、52.172.50.24、52.172.55.231、52.172.52.0|
|欧洲西部|13.95.155.53、52.174.54.218、52.174.49.6、40.68.222.65、40.68.209.23、13.95.147.65|
|印度西部|104.211.164.112、104.211.165.81、104.211.164.25、104.211.164.80、104.211.162.205、104.211.164.136|
|美国西部|52.160.90.237、138.91.188.137、13.91.252.184、52.160.92.112、40.118.244.241、40.118.241.243|

#### <a name="connectors"></a>连接器

从[连接器](../connectors/apis-list.md)进行的调用来自下面指定的 IP 地址：

|逻辑应用区域|出站 IP|
|-----|----|
|澳大利亚东部|40.126.251.213|
|澳大利亚东南部|40.127.80.34|
|巴西南部|191.232.38.129|
|加拿大中部|52.233.31.197,52.228.42.205,52.228.33.76,52.228.34.13|
|加拿大东部|52.229.123.98,52.229.120.178,52.229.126.202,52.229.120.52|
|印度中部|104.211.98.164|
|美国中部|40.122.49.51|
|东亚|23.99.116.181|
|美国东部|191.237.41.52|
|美国东部 2|104.208.233.100|
|日本东部|40.115.186.96|
|日本西部|40.74.130.77|
|美国中北部|65.52.218.230|
|欧洲北部|104.45.93.9|
|美国中南部|104.214.70.191|
|亚洲东南部|13.76.231.68|
|印度南部|104.211.227.225|
|欧洲西部|40.115.50.13|
|印度西部|104.211.161.203|
|美国西部|104.40.51.248|


## <a name="next-steps"></a>后续步骤  

- 若要开始使用逻辑应用，请按照[创建逻辑应用](../logic-apps/logic-apps-create-a-logic-app.md)教程进行操作。  
- [查看常见示例和方案](../logic-apps/logic-apps-examples-and-scenarios.md)
- [使用逻辑应用可以自动执行业务流程](http://channel9.msdn.com/Events/Build/2016/T694) 
- [了解如何将系统与逻辑应用集成](http://channel9.msdn.com/Events/Build/2016/P462)

