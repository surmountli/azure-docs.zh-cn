---
title: "创建非交互式身份验证 .NET HDInsight 应用程序 | Microsoft Docs"
description: "了解如何创建非交互式身份验证 .NET HDInsight 应用程序。"
editor: cgronlun
manager: jhubbard
services: hdinsight
documentationcenter: 
tags: azure-portal
author: mumian
ms.assetid: 8e32430f-6404-498a-9fcd-f20338d964af
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/22/2017
ms.author: jgao
translationtype: Human Translation
ms.sourcegitcommit: db7cb109a0131beee9beae4958232e1ec5a1d730
ms.openlocfilehash: 2b8638ffc3287346a71f591370367655c450e376
ms.lasthandoff: 04/19/2017


---
# <a name="create-non-interactive-authentication-net-hdinsight-applications"></a>创建非交互式身份验证 .NET HDInsight 应用程序
可以在应用程序自身的标识（非交互式）或应用程序的已登录用户标识（交互式）下运行 .NET Azure HDInsight 应用程序。 有关交互式应用程序的示例，请参阅[连接到 Azure HDInsight](hdinsight-administer-use-dotnet-sdk.md#connect-to-azure-hdinsight)。 本文介绍了如何创建非交互式身份验证 .NET 应用程序以连接到 Azure 并管理 HDInsight。

需要从非交互式 .NET 应用程序中获取：

* Azure 订阅租户 ID（A.K.A 目录 ID）。 请参阅[获取租户 ID](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-tenant-id)。
* Azure Active Directory 应用程序客户端 ID。 请参阅[创建 Azure Active Directory 应用程序](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)和[获取应用程序 ID](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-application-id-and-authentication-key)
* Azure Active Directory 应用程序密钥。 请参阅[获取应用程序身份验证密钥](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-application-id-and-authentication-key)

## <a name="prerequisites"></a>先决条件
* HDInsight 群集。 请参阅[获取入门教程](hdinsight-hadoop-linux-tutorial-get-started.md#create-cluster)。



## <a name="assign-azure-ad-application-to-role"></a>将 Azure AD 应用程序分配到角色
必须将应用程序分配到某个[角色](../active-directory/role-based-access-built-in-roles.md)，以授予它执行操作的权限。 可将作用域设置为订阅、资源组或资源级别。 作用域的较低级别将继承权限（例如，将某个应用程序添加到资源组的“读取者”角色意味着该应用程序可以读取该资源组及其包含的所有资源）。 在本教程中，你将在资源组级别设置作用域。 更多详细信息，请参阅[使用角色分配，管理对 Azure 订阅资源的访问权限](../active-directory/role-based-access-control-configure.md)

**将“所有者”角色添加到 Azure AD 应用程序**

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 单击左侧窗格中的“资源组”。
3. 单击包含 HDInsight 群集（在本教程中，将在其中运行 Hive 查询）的资源组。 如果有过多资源组，可以使用筛选器。
4. 从“资源组”菜单中单击“访问控制 (IAM)”。
5. 单击“用户”边栏选项卡上的“添加”。
6. 按照说明，将“所有者”角色添加到上一个过程中创建的 Azure AD 应用程序。 成功完成后，应该可在具有“所有者”角色的“用户”边栏选项卡中看到列出的应用程序。

## <a name="develop-hdinsight-client-application"></a>开发 HDInsight 客户端应用程序

1. 创建 C# 控制台应用程序。
2. 添加以下 Nuget 包：

        Install-Package Microsoft.Azure.Common.Authentication -Pre
        Install-Package Microsoft.Azure.Management.HDInsight -Pre
        Install-Package Microsoft.Azure.Management.Resources -Pre

3. 使用以下代码示例：

        using System;
        using System.Security;
        using Microsoft.Azure;
        using Microsoft.Azure.Common.Authentication;
        using Microsoft.Azure.Common.Authentication.Factories;
        using Microsoft.Azure.Common.Authentication.Models;
        using Microsoft.Azure.Management.Resources;
        using Microsoft.Azure.Management.HDInsight;
        
        namespace CreateHDICluster
        {
            internal class Program
            {
                private static HDInsightManagementClient _hdiManagementClient;
        
                private static Guid SubscriptionId = new Guid("<Enter Your Azure Subscription ID>");
                private static string tenantID = "<Enter Your Tenant ID (A.K.A. Directory ID)>";
                private static string applicationID = "<Enter Your Application ID>";
                private static string secretKey = "<Enter the Application Secret Key>";
        
                private static void Main(string[] args)
                {
                    var key = new SecureString();
                    foreach (char c in secretKey) { key.AppendChar(c); }

                    var tokenCreds = GetTokenCloudCredentials(tenantID, applicationID, key);
                    var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
        
                    var resourceManagementClient = new ResourceManagementClient(subCloudCredentials);
                    resourceManagementClient.Providers.Register("Microsoft.HDInsight");
        
                    _hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);
        
                    var results = _hdiManagementClient.Clusters.List();
                    foreach (var name in results.Clusters)
                    {
                        Console.WriteLine("Cluster Name: " + name.Name);
                        Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
                        Console.WriteLine("\t Cluster location: " + name.Location);
                        Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
                    }
                    Console.WriteLine("Press Enter to continue");
                    Console.ReadLine();
                }

                /// Get the access token for a service principal and provided key                
                public static TokenCloudCredentials GetTokenCloudCredentials(string tenantId, string clientId, SecureString secretKey)
                {
                    var authFactory = new AuthenticationFactory();
                    var account = new AzureAccount { Type = AzureAccount.AccountType.ServicePrincipal, Id = clientId };
                    var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
                    var accessToken =
                        authFactory.Authenticate(account, env, tenantId, secretKey, ShowDialog.Never).AccessToken;
        
                    return new TokenCloudCredentials(accessToken);
                }
        
                public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(SubscriptionCloudCredentials creds, Guid subId)
                {
                    return new TokenCloudCredentials(subId.ToString(), ((TokenCloudCredentials)creds).Token);
                }
            }
        }

## <a name="next-steps"></a>后续步骤
* [使用门户创建 Azure Active Directory 应用程序和服务主体](../azure-resource-manager/resource-group-create-service-principal-portal.md)
* [通过 Azure Resource Manager 对服务主体进行身份验证](../azure-resource-manager/resource-group-authenticate-service-principal.md)
* [Azure 基于角色的访问控制](../active-directory/role-based-access-control-configure.md)

