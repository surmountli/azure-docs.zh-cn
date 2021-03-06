---
title: "使用 DMV 监视工作负荷 | Microsoft 文档"
description: "了解如何使用 DMV 监视工作负荷。"
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jhubbard
editor: 
ms.assetid: 69ecd479-0941-48df-b3d0-cf54c79e6549
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.custom: performance
ms.date: 10/31/2016
ms.author: barbkess
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 3735d656429da1f1fe7569f640b272b099382032
ms.lasthandoff: 04/03/2017


---
# <a name="monitor-your-workload-using-dmvs"></a>使用 DMV 监视工作负荷
本文介绍如何使用动态管理视图 (DMV) 在 Azure SQL 数据仓库中监视工作负荷及调查查询执行情况。

## <a name="permissions"></a>权限
若要查询本文中的 DMV，需具有 VIEW DATABASE STATE 或 CONTROL 权限。 通常情况下，首选授予 VIEW DATABASE STATE 权限，因为该权限的限制要大得多。

```sql
GRANT VIEW DATABASE STATE TO myuser;
```

## <a name="monitor-connections"></a>监视连接
所有登录到 SQL 数据仓库的操作都记录到 [sys.dm_pdw_exec_sessions][sys.dm_pdw_exec_sessions]。  此 DMV 包含最后 10,000 个登录。  session_id 是主键，每次进行新的登录时按顺序分配。

```sql
-- Other Active Connections
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed' and session_id <> session_id();
```

## <a name="monitor-query-execution"></a>监视查询执行
在 SQL 数据仓库上执行的所有查询都记录到 [sys.dm_pdw_exec_requests][sys.dm_pdw_exec_requests]。  此 DMV 包含最后 10,000 个执行的查询。  request_id 对每个查询进行唯一标识，并且为此 DMV 的主键。  request_id 在每次进行新的查询时按顺序分配，并会加上前缀 QID，代表查询 ID。  针对给定 session_id 查询此 DMV 将显示给定登录的所有查询。

> [!NOTE]
> 存储过程使用多个请求 ID。  按先后顺序分配请求 ID。 
> 
> 

以下是调查特定查询的查询执行计划和时间所要遵循的步骤。

### <a name="step-1-identify-the-query-you-wish-to-investigate"></a>步骤 1：确定想要调查的查询
```sql
-- Monitor active queries
SELECT * 
FROM sys.dm_pdw_exec_requests 
WHERE status not in ('Completed','Failed','Cancelled')
  AND session_id <> session_id()
ORDER BY submit_time DESC;

-- Find top 10 queries longest running queries
SELECT TOP 10 * 
FROM sys.dm_pdw_exec_requests 
ORDER BY total_elapsed_time DESC;

-- Find a query with the Label 'My Query'
-- Use brackets when querying the label column, as it it a key word
SELECT  *
FROM    sys.dm_pdw_exec_requests
WHERE   [label] = 'My Query';
```

从前面的查询结果中，记下想要调查的查询的**请求 ID**。

处于**已暂停**状态的查询是指因并发限制而排队的查询。 这些查询也出现在类型为 UserConcurrencyResourceType 的 sys.dm_pdw_waits 等待查询中。 有关并发限制的详细信息，请参阅[并发和工作负荷管理][Concurrency and workload management]。 查询也可能因其他原因（如对象锁定）处于等待状态。  如果查询正在等待资源，请参阅本文后面的[调查等待资源的查询][Investigating queries waiting for resources]。

为了简化在 sys.dm_pdw_exec_requests 表中查找查询的过程，请使用 [LABEL][LABEL] 将注释分配给可在 sys.dm_pdw_exec_requests 视图中查找的查询。

```sql
-- Query with Label
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query')
;
```

### <a name="step-2-investigate-the-query-plan"></a>步骤 2：调查查询计划
使用请求 ID 从 [sys.dm_pdw_request_steps][sys.dm_pdw_request_steps] 检索查询的分布式 SQL (DSQL) 计划。

```sql
-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID####'
ORDER BY step_index;
```

当 DSQL 计划的执行时间超出预期时，原因可能是计划很复杂，包含许多 DSQL 步骤，也可能是一个步骤占用很长的时间。  如果计划有很多步骤，包含多个移动操作，可考虑优化表分布，减少数据移动。 [表分布][Table distribution]一文说明了为何必须移动数据才能解决查询问题，并说明了如何使用某些分布策略，尽量减少数据移动。

若要进一步调查单个步骤的详细信息，可检查长时间运行的查询步骤的 *operation_type* 列并记下**步骤索引**：

* 针对以下 **SQL 操作**继续执行步骤 3a：OnOperation、RemoteOperation、ReturnOperation。
* 针对以下**数据移动操作**继续执行步骤 3b：ShuffleMoveOperation、BroadcastMoveOperation、TrimMoveOperation、PartitionMoveOperation、MoveOperation、CopyOperation。

### <a name="step-3a-investigate-sql-on-the-distributed-databases"></a>步骤 3a：查看分布式数据库上的 SQL
使用请求 ID 和步骤索引从 [sys.dm_pdw_sql_requests][sys.dm_pdw_sql_requests] 中检索详细信息，其中包含所有分布式数据库上的查询步骤的执行信息。

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID####' AND step_index = 2;
```

当查询步骤正在运行时，可以使用 [DBCC PDW_SHOWEXECUTIONPLAN][DBCC PDW_SHOWEXECUTIONPLAN] 从 SQL Server 计划缓存中检索 SQL Server 估计计划，了解在特定分布基础上运行的步骤。

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);
```

### <a name="step-3b-investigate-data-movement-on-the-distributed-databases"></a>步骤 3b：查看在分布式数据库上进行的数据移动
使用请求 ID 和步骤索引检索在 [sys.dm_pdw_dms_workers][sys.dm_pdw_dms_workers] 中的每个分布上运行的数据移动步骤的相关信息。

```sql
-- Find the information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID####' AND step_index = 2;
```

* 检查 *total_elapsed_time* 列，以查看是否有特定分布在数据移动上比其他分布花费了更多时间。
* 对于长时间运行的分布，请检查 *rows_processed* 列，以查看从该分布移动的行数是否远远多于其他分布。 如果是这样，这可能表示底层数据的偏斜。

如果查询正在运行，则可以使用 [DBCC PDW_SHOWEXECUTIONPLAN][DBCC PDW_SHOWEXECUTIONPLAN] 检索特定分发中当前正在运行的 SQL 步骤的 SQL Server 计划高速缓存中的 SQL Server 估计计划。

```sql
-- Find the SQL Server estimated plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);
```

<a name="waiting"></a>

## <a name="monitor-waiting-queries"></a>监视正在等待的查询
如果查询未取得进展（因其正在等待资源），下面是显示查询正在等待的所有资源的查询。

```sql
-- Find queries 
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,
      waits.state,
      waits.object_type,
      waits.object_name
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID####'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

如果查询正在主动等待另一个查询中的资源，则状态将为 **AcquireResources**。  如果查询具有全部所需资源，则状态将为 **Granted**。

## <a name="next-steps"></a>后续步骤
请参阅[系统视图][System views]，了解 DMV 的详细信息。
有关最佳实践的详细信息，请参阅 [SQL 数据仓库最佳实践][SQL Data Warehouse best practices]

<!--Image references-->

<!--Article references-->
[Manage overview]: ./sql-data-warehouse-overview-manage.md
[SQL Data Warehouse best practices]: ./sql-data-warehouse-best-practices.md
[System views]: ./sql-data-warehouse-reference-tsql-system-views.md
[Table distribution]: ./sql-data-warehouse-tables-distribute.md
[Concurrency and workload management]: ./sql-data-warehouse-develop-concurrency.md
[Investigating queries waiting for resources]: ./sql-data-warehouse-manage-monitor.md#waiting

<!--MSDN references-->
[sys.dm_pdw_dms_workers]: http://msdn.microsoft.com/library/mt203878.aspx
[sys.dm_pdw_exec_requests]: http://msdn.microsoft.com/library/mt203887.aspx
[sys.dm_pdw_exec_sessions]: http://msdn.microsoft.com/library/mt203883.aspx
[sys.dm_pdw_request_steps]: http://msdn.microsoft.com/library/mt203913.aspx
[sys.dm_pdw_sql_requests]: http://msdn.microsoft.com/library/mt203889.aspx
[DBCC PDW_SHOWEXECUTIONPLAN]: http://msdn.microsoft.com/library/mt204017.aspx
[DBCC PDW_SHOWSPACEUSED]: http://msdn.microsoft.com/library/mt204028.aspx
[LABEL]: https://msdn.microsoft.com/library/ms190322.aspx

