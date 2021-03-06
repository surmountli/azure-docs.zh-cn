---
title: "教程：Azure Active Directory 与 New Relic 集成 | Microsoft 文档"
description: "了解如何使用 New Relic 与 Azure Active Directory 以启用单一登录、自动化预配和其他功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 3186b9a8-f4d8-45e2-ad82-6275f95e7aa6
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/24/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 84f9c9745cc0c95fc5134dcc7e659e7ace11b188
ms.lasthandoff: 04/03/2017


---
# <a name="tutorial-azure-active-directory-integration-with-new-relic"></a>教程：Azure Active Directory 与 New Relic 集成
本教程的目的是介绍如何在 Azure Active Directory 和 New Relic 之间设置单一登录 (SSO)。

在本教程中概述的方案假定您已具有以下各项：

* 一个有效的 Azure 订阅
* 启用 New Relic 单一登录 (SSO) 的订阅

完成本教程后，已向 New Relic 分配的 Azure Active Directory 用户将能够使用 AAD 访问面板进行 SSO。

1. 为 New Relic 启用应用程序集成
2. 配置单一登录 (SSO)
3. 配置用户设置
4. 分配用户

![方案](./media/active-directory-saas-new-relic-tutorial/IC797030.png "方案")

## <a name="enabling-the-application-integration-for-new-relic"></a>为 New Relic 启用应用程序集成
本部分的目的是概述如何为 New Relic 启用应用程序集成。

**若要为 New Relic 启用应用程序集成，请执行以下步骤：**

1. 在 Azure 经典门户的左侧导航窗格中，单击“Active Directory”。
   
   ![Active Directory](./media/active-directory-saas-new-relic-tutorial/IC700993.png "Active Directory")
2. 在“目录”列表中，选择要启用目录集成的目录。
3. 若要打开应用程序视图，请在目录视图的顶部菜单中，单击“应用程序”。
   
   ![应用程序](./media/active-directory-saas-new-relic-tutorial/IC700994.png "应用程序")
4. 在页面底部单击“添加”。
   
   ![添加应用程序](./media/active-directory-saas-new-relic-tutorial/IC749321.png "添加应用程序")
5. 在“要执行什么操作”对话框中，单击“从库中添加应用程序”。
   
   ![从库添加应用程序](./media/active-directory-saas-new-relic-tutorial/IC749322.png "从库添加应用程序")
6. 在**搜索框**中，键入 **New Relic**。
   
   ![应用程序库](./media/active-directory-saas-new-relic-tutorial/IC797031.png "应用程序库")
7. 在结果窗格中，选择“New Relic”，然后单击“完成”添加该应用程序。
   
   ![New Relic](./media/active-directory-saas-new-relic-tutorial/IC797032.png "New Relic")
   
## <a name="configure-single-sign-on"></a>配置单一登录

此部分概述如何让用户使用基于 SAML 协议的联合身份验证通过其在 Azure Active Directory 中的帐户向 New Relic 进行身份验证。

**若要配置 SSO，请执行以下步骤：**

1. 在 Azure 经典门户的“New Relic”应用程序集成页上，单击“配置单一登录”打开“配置单一登录”对话框。
   
   ![配置单一登录](./media/active-directory-saas-new-relic-tutorial/IC769534.png "配置单一登录")
2. 在“你希望用户如何登录 New Relic”页上，选择“Microsoft Azure AD 单一登录”，然后单击“下一步”。
   
   ![配置单一登录](./media/active-directory-saas-new-relic-tutorial/IC797033.png "配置单一登录")
3. 在“配置应用 URL”页的“New Relic 登录 URL”文本框中，键入用户登录到 New Relic 应用程序时使用的 URL，然后单击“下一步”。 
   
   应用 URL 是 New Relic 租户 URL（例如 *https://rpm.newrelic.com*）：
   
   ![配置应用 URL](./media/active-directory-saas-new-relic-tutorial/IC797034.png "配置应用 URL")
4. 在“配置 New Relic 的单一登录”页面上，若要下载证书，请单击“下载证书”，然后在计算机上本地保存该证书文件。
   
   ![配置单一登录](./media/active-directory-saas-new-relic-tutorial/IC797035.png "配置单一登录")
5. 在其他 Web 浏览器窗口中，以管理员身份登录 **New Relic** 公司站点。
6. 在顶部菜单中，单击“帐户设置”。
   
   ![帐户设置](./media/active-directory-saas-new-relic-tutorial/IC797036.png "帐户设置")
7. 单击“安全性和身份验证”选项卡，然后单击“单一登录”选项卡。
   
   ![单一登录](./media/active-directory-saas-new-relic-tutorial/IC797037.png "单一登录")
8. 在 SAML 对话框页上执行以下步骤：
   
   ![SAML](./media/active-directory-saas-new-relic-tutorial/IC797038.png "SAML")
   
   1. 单击“选择文件”，上载已下载的 Azure Active Directory 证书。
   2. 在 Azure 经典门户的“配置 New Relic 的单一登录”页面上，复制“远程登录 URL”值，然后将其粘贴到“远程登录 URL”文本框中。
   3. 在 Azure 经典门户的“配置 New Relic 的单一登录”页面上，复制“远程注销 URL”值，然后将其粘贴到“注销登录 URL”文本框中。
   4. 单击“保存所做的更改”。
9. 在 Azure 经典门户中，选择“单一登录配置确认”，然后单击“完成”，关闭“配置单一登录”对话框。
   
   ![配置单一登录](./media/active-directory-saas-new-relic-tutorial/IC797039.png "配置单一登录")
   
## <a name="configure-user-provisioning"></a>配置用户设置

若要让 Azure Active Directory 用户登录到 New Relic，必须将其预配到 New Relic 中。 使用 New Relic 时，预配属手动任务。

**若要将用户帐户预配到 New Relic，请执行以下步骤：**

1. 以管理员身份登录到 **New Relic** 公司站点。
2. 在顶部菜单中，单击“帐户设置”。
   
   ![帐户设置](./media/active-directory-saas-new-relic-tutorial/IC797040.png "帐户设置")
3. 在左侧的“帐户”窗格中，单击“摘要”，然后单击“添加用户”。
   
   ![帐户设置](./media/active-directory-saas-new-relic-tutorial/IC797041.png "帐户设置")
4. 在“活动用户”对话框中，执行以下步骤：
   
   ![活动用户](./media/active-directory-saas-new-relic-tutorial/IC797042.png "活动用户")
   
   1. 在“电子邮件”文本框中，键入要预配的有效 Azure Active Directory 用户的电子邮件地址。
   2. 选择“用户”作为“角色”。
   3. 单击“添加此用户”。

>[!NOTE]
>可以使用任何其他 New Relic 用户帐户创建工具或 New Relic 提供的 API 来预配 AAD 用户帐户。
> 
> 

## <a name="assign-users"></a>分配用户
若要测试配置，需要通过分配权限的方式向希望其使用应用程序的 Azure AD 用户授予该配置的访问权限。

**若要将用户分配到 New Relic，请执行以下步骤：**

1. 在 Azure 经典门户中，创建测试帐户。
2. 在 **New Relic** 应用程序集成页上，单击“分配用户”。
   
   ![分配用户](./media/active-directory-saas-new-relic-tutorial/IC797043.png "分配用户")
3. 选择测试用户，单击“分配”，然后单击“是”确认分配。
   
   ![是](./media/active-directory-saas-new-relic-tutorial/IC767830.png "是")

如果要测试 SSO 设置，请打开访问面板。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](active-directory-saas-access-panel-introduction.md)（访问面板简介）。

## <a name="additional-resources"></a>其他资源

* [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](active-directory-saas-tutorial-list.md)
* [Azure Active Directory 的应用程序访问与单一登录是什么？](active-directory-appssoaccess-whatis.md)
