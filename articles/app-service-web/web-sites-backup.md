---
title: "在 Azure 中备份应用"
description: "了解如何在 Azure App Service 中创建应用备份。"
services: app-service
documentationcenter: 
author: cephalin
manager: erikre
editor: jimbe
ms.assetid: 6223b6bd-84ec-48df-943f-461d84605694
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2016
ms.author: cephalin
translationtype: Human Translation
ms.sourcegitcommit: b1a633a86bd1b5997d5cbf66b16ec351f1043901
ms.openlocfilehash: b00c320eeb1e8b30a0fad9634844ceeca60d1c29
ms.lasthandoff: 01/20/2017


---
# <a name="back-up-your-app-in-azure"></a>在 Azure 中备份应用
利用 [Azure App Service](../app-service/app-service-value-prop-what-is.md) 中的备份和还原功能，可以轻松地手动或自动创建应用备份。 你可以将应用还原到以前的状态，或者基于原始应用的备份之一创建新的应用。 

有关从备份中还原应用的信息，请参阅[在 Azure 中还原应用](web-sites-restore.md)。

<a name="whatsbackedup"></a>

## <a name="what-gets-backed-up"></a>备份的内容
应用服务可备份以下信息：

* 应用配置
* 文件内容
* 任何连接到你的应用的 Azure SQL 数据库或 Azure MySQL (ClearDB) 数据库（你可以选择备份中要包括哪些数据库）

此信息会备份到你指定的 Azure 存储帐户和容器。 

> [!NOTE]
> 每个备份都是你的应用的完整脱机副本，而不是增量更新。
> 
> 

<a name="requirements"></a>

## <a name="requirements-and-restrictions"></a>要求和限制
* 备份和还原功能要求应用服务计划位于**标准**层或更高层。 有关缩放应用服务计划以使用更高层的详细信息，请参阅[增加 Azure 中的应用](web-sites-scale.md)。 请注意，与**标准**层相比，**高级**层每日允许更多备份量。
* 在与要备份的应用相同的订阅中，需要有一个 Azure 存储帐户和容器。 有关 Azure 存储帐户的详细信息，请参阅本文结尾处的[链接](#moreaboutstorage)。
* 最多可备份 10GB 的应用和数据库内容。 如果备份大小超过此限制，将会出错。 

<a name="manualbackup"></a>

## <a name="create-a-manual-backup"></a>创建手动备份
1. 在 [Azure 门户](https://portal.azure.com)中，导航到应用的边栏选项卡，选择“设置”，然后选择“备份”。 将显示“备份”边栏选项卡。
   
    ![“备份”页面][ChooseBackupsPage]
   
   > [!NOTE]
   > 若显示以下消息，请单击该消息升级应用服务计划，然后才能继续备份。
   > 有关详细信息，请参阅[增加 Azure 中的应用](web-sites-scale.md)。  
   > ![选择存储帐户](./media/web-sites-backup/01UpgradePlan.png)
   > 
   > 
2. 在“备份”边栏选项卡中，单击“存储: 未配置”来配置存储帐户。
   
    ![选择存储帐户][ChooseStorageAccount]
3. 选择“存储帐户”和“容器”来选择备份目标。 该存储帐户必须与要备份的应用属于同一订阅。 也可在各自的边栏选项卡中新建存储帐户或容器。 完成后，单击“选择”。
   
    ![选择存储帐户](./media/web-sites-backup/02ChooseStorageAccount1.png)
4. 在仍处于打开状态的“配置备份设置”边栏选项卡中，单击“数据库设置”、选择备份要包含的数据库（SQL 数据库或 MySQL），然后单击“确定”。  
   
    ![选择存储帐户](./media/web-sites-backup/03ConfigureDatabase.png)
   
   > [!NOTE]
   > 若要使数据库显示在此列表中，其连接字符串必须位于应用中“应用程序设置”边栏选项卡的“连接字符串”部分中。
   > 
   > 
5. 在“配置备份设置”边栏选项卡中，单击“保存”。    
6. 在“备份”边栏选项卡的命令栏中，单击“立即备份”。
   
    ![BackUpNow 按钮][BackUpNow]
   
    备份过程中将显示进度消息。

配置用于备份的存储帐户和容器后，可随时进行手动备份。  

<a name="automatedbackups"></a>

## <a name="configure-automated-backups"></a>配置自动执行的备份
1. 在“备份”边栏选项卡中，单击“计划: 未配置”。 
   
    ![选择存储帐户](./media/web-sites-backup/05ScheduleBackup.png)
2. 在“备份计划设置”边栏选项卡上，将“计划备份”设为“开”，然后根据需要配置备份计划，并单击“确定”。
   
    ![启用自动化的备份][SetAutomatedBackupOn]
3. 在仍处于打开状态的“配置备份设置”边栏选项卡上单击“存储设置”，然后通过选择“存储帐户”和“容器”来选择备份目标。 该存储帐户必须与要备份的应用属于同一订阅。 也可在各自的边栏选项卡中新建存储帐户或容器。 完成后，单击“选择”。
   
    ![选择存储帐户](./media/web-sites-backup/02ChooseStorageAccount1.png)
4. 在“配置备份设置”边栏选项卡中，单击“数据库设置”，选择备份要包含的数据库（SQL 数据库或 MySQL），然后单击“确定”。  
   
    ![选择存储帐户](./media/web-sites-backup/03ConfigureDatabase.png)
   
   > [!NOTE]
   > 若要使数据库显示在此列表中，其连接字符串必须位于应用中“应用程序设置”边栏选项卡的“连接字符串”部分中。
   > 
   > 
5. 在“配置备份设置”边栏选项卡中，单击“保存”。    

<a name="partialbackups"></a>

## <a name="backup-just-part-of-your-app"></a>备份只是应用的一部分
有时你不想备份应用中的所有内容。 以下是一些示例：

* [设置每周备份](web-sites-backup.md#configure-automated-backups)应用，其中包含永远不会更改的静态内容，例如旧的博客文章或图像。
* 应用的内容超过 10GB（这是一次可以备份的最大数量）。
* 你不想备份日志文件。

使用部分备份可以精确选择想要备份的文件。

### <a name="exclude-files-from-your-backup"></a>从备份中排除文件
若要从备份中排除文件和文件夹，在应用的 D:\home\site\wwwroot 文件夹中创建 `_backup.filter` 文件，并指定想要在这里排除的文件和文件夹列表。 通过 [Kudu 控制台](https://github.com/projectkudu/kudu/wiki/Kudu-console)，即可轻松访问它。 

假设你的应用中包含永远不会更改的历年的日志文件和静态映像。 你已有包括旧映像的应用的完整备份。 现在你想要每天备份应用，但不想为永远不会更改的存储日志文件或静态映像文件支付费用。

![日志文件夹][LogsFolder]
![图像文件夹][ImagesFolder]

以下步骤展示了如何从备份中排除这些文件。

1. 转到 `http://{yourapp}.scm.azurewebsites.net/DebugConsole` 并确定想要从备份中排除的文件夹。 在此示例中，你想要排除该用户界面中所示的以下文件和文件夹：
   
        D:\home\site\wwwroot\Logs
        D:\home\LogFiles
        D:\home\site\wwwroot\Images\2013
        D:\home\site\wwwroot\Images\2014
        D:\home\site\wwwroot\Images\brand.png
   
    [AZURE.NOTE] 最后一行显示了你可以排除单个文件以及文件夹。
2. 创建名为 `_backup.filter` 的文件并将上述列表放在文件中，但删除 `D:\home`。 每行列出一个目录或文件。 文件的内容应为：
   
    \site\wwwroot\Logs  \LogFiles  \site\wwwroot\Images\2013  \site\wwwroot\Images\2014  \site\wwwroot\Images\brand.png
3. 使用 [ftp](web-sites-deploy.md#ftp) 或任何其他方法，将此文件上载到站点 `D:\home\site\wwwroot\` 目录。 如果你愿意，可以直接在 `http://{yourapp}.scm.azurewebsites.net/DebugConsole` 中创建文件并在那里插入内容。
4. 采用通常使用的相同方式运行备份，即[手动](#create-a-manual-backup)或[自动](#configure-automated-backups)。

现在，`_backup.filter` 中指定的所有文件和文件夹都会从备份中排除。 在此示例中，将不再对日志文件，2013 年和 2014 年的映像文件以及 brand.png 进行备份。

> [!NOTE]
> 采用与[还原定期备份](web-sites-restore.md)相同的方式，还原站点的部分备份。 还原过程会执行正确的操作。
> 
> 还原完整备份后，站点上的所有内容都被替换为备份中的任何内容。 如果文件在站点上但不在备份中，则会将其删除。 但是，还原部分备份时，位于其中一个黑名单目录或任何黑名单文件中的任何内容都保持不变。
> 
> 

<a name="aboutbackups"></a>

## <a name="how-backups-are-stored"></a>如何存储备份
对应用进行了一次或多次备份后，将在存储帐户的“容器”边栏选项卡中看到备份以及应用。 在存储帐户中，每个备份都由一个 .zip 文件和一个 .xml 文件组成，前者包含备份数据，后者包含 .zip 文件内容的清单。 如果你想要在无需实际执行应用还原的情况下访问备份，则可以解压缩并浏览这些文件。

应用的数据库备份存储在 .zip 文件的根目录中。 对于 SQL 数据库，这是 BACPAC 文件（无文件扩展名），并且可以导入。 若要基于 BACPAC 导出创建新的 SQL 数据库，请参阅[导入 BACPAC 文件以创建新的用户数据库](http://technet.microsoft.com/library/hh710052.aspx)。

> [!WARNING]
> 改动 **websitebackups** 容器中的任何文件都导致备份无效，进而无法还原。
> 
> 

<a name="nextsteps"></a>

## <a name="next-steps"></a>后续步骤
有关从备份中还原应用的信息，请参阅[在 Azure 中还原应用](web-sites-restore.md)。 还可使用 REST API 备份和还原应用服务应用（请参阅[使用 REST 备份和还原应用服务应用](websites-csm-backup.md)）。

> [!NOTE]
> 如果您想要在注册 Azure 帐户之前开始使用 Azure App Service，请转到 [试用 App Service](https://azure.microsoft.com/try/app-service/)，您可以在 App Service 中立即创建一个生存期较短的入门 Web 应用。 你不需要使用信用卡，也不需要做出承诺。
> 
> 

<!-- IMAGES -->
[ChooseBackupsPage]: ./media/web-sites-backup/01ChooseBackupsPage.png
[ChooseStorageAccount]: ./media/web-sites-backup/02ChooseStorageAccount.png
[IncludedDatabases]: ./media/web-sites-backup/03IncludedDatabases.png
[BackUpNow]: ./media/web-sites-backup/04BackUpNow.png
[BackupProgress]: ./media/web-sites-backup/05BackupProgress.png
[SetAutomatedBackupOn]: ./media/web-sites-backup/06SetAutomatedBackupOn.png
[Frequency]: ./media/web-sites-backup/07Frequency.png
[StartDate]: ./media/web-sites-backup/08StartDate.png
[StartTime]: ./media/web-sites-backup/09StartTime.png
[SaveIcon]: ./media/web-sites-backup/10SaveIcon.png
[ImagesFolder]: ./media/web-sites-backup/11Images.png
[LogsFolder]: ./media/web-sites-backup/12Logs.png
[GhostUpgradeWarning]: ./media/web-sites-backup/13GhostUpgradeWarning.png


