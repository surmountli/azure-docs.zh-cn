---
title: "为 StorSimple 设备设置 Web 代理 | Microsoft 文档"
description: "了解如何使用 Windows PowerShell for StorSimple 为 StorSimple 设备配置 Web 代理设置。"
services: storsimple
documentationcenter: 
author: alkohli
manager: carmonm
editor: 
ms.assetid: 6c2ca351-a7c6-4da6-ab5e-c081e6d08261
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/17/2016
ms.author: alkohli
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 66ee6ce15e51b14366eac0512c899d1c425c6092


---
# <a name="configure-web-proxy-for-your-storsimple-device"></a>为 StorSimple 设备配置 Web 代理
## <a name="overview"></a>概述
本教程介绍如何使用 Windows PowerShell for StorSimple 来配置和查看 StorSimple 设备的 Web 代理设置。 StorSimple 设备与云通信时使用 Web 代理设置。 Web 代理服务器用于添加另一个安全层、筛选内容、提供缓存以缓解带宽要求，甚至可帮助分析信息。

Web 代理是 StorSimple 设备的一项可选配置。 只能通过 Windows PowerShell for StorSimple 配置 Web 代理。 配置过程由两个步骤组成，如下所述：

1. 首先通过安装向导或 Windows PowerShell for StorSimple cmdlet 配置 Web 代理设置。
2. 然后通过 Windows PowerShell for StorSimple cmdlet 启用配置的 Web 代理设置。

完成 Web 代理配置后，可以在 Microsoft Azure StorSimple Manager 服务和 Windows PowerShell for StorSimple 中查看配置的 Web 代理设置。 

阅读本教程之后，你将能够：

* 使用安装向导和 cmdlet 配置 Web 代理
* 使用 cmdlet 启用 Web 代理
* 在 Azure 经典门户中查看 Web 代理设置
* 在 Web 代理配置期间排查错误

## <a name="configure-web-proxy-via-windows-powershell-for-storsimple"></a>通过 Windows PowerShell for StorSimple 配置 Web 代理
可以使用以下方法之一配置 Web 代理设置：

* 使用安装向导来引导完成配置步骤。
* 使用 Windows PowerShell for StorSimple 中的 cmdlet。

以下部分将介绍上述每种方法。

## <a name="configure-web-proxy-via-the-setup-wizard"></a>通过安装向导配置 Web 代理
可以使用安装向导来引导自己完成 Web 代理的配置步骤。 执行以下步骤，在设备上配置 Web 代理。

#### <a name="to-configure-web-proxy-via-the-setup-wizard"></a>通过安装向导配置 Web 代理
1. 在串行控制台菜单中，选择选项 1，**使用完全访问权限登录**，并提供**设备管理员密码**。 键入以下命令启动安装向导会话：
   
    `Invoke-HcsSetupWizard`
2. 如果这是首次为设备注册运行安装向导，则需要配置全部所需的网络设置，直到看到 Web 代理的配置步骤。 如果设备已注册，则可接受所有配置的网络设置，直到看到 Web 代理的配置步骤。 在安装向导中，当系统提示是否配置 Web 代理设置时，请选择“是”。
3. 对于“Web 代理 URL”，请指定 Web 代理服务器的 IP 地址或完全限定的域名 (FQDN)，以及设备在与云通信时要使用的 TCP 端口号。 使用以下格式：
   
    `http://<IP address or FQDN of the web proxy server>:<TCP port number>`
   
    默认情况下，已指定 TCP 端口号 8080。
4. 选择“NTLM”、“基本”或“无”作为身份验证类型。 “基本”是代理服务器配置使用的安全性最低的身份验证。 NT LAN Manager (NTLM) 是高度安全和复杂的身份验证协议，使用三向消息传送系统（在需要更高的数据完整性时，有时还会使用四向）验证用户的身份。 默认的身份验证为 NTLM。 有关详细信息，请参阅[基本身份验证](http://hc.apache.org/httpclient-3.x/authentication.html)和 [NTLM 身份验证](http://hc.apache.org/httpclient-3.x/authentication.html)。 
   
   > [!IMPORTANT]
   > **在 StorSimple Manager 服务中，如果在设备的代理服务器配置中启用“基本”或“NTLM”身份验证，设备监视图表将不起作用。为使监视图表起作用，需确保将身份验证设置为“无”。**
   > 
   > 
5. 如果使用身份验证，请提供 **Web 代理用户名**和 **Web 代理密码**。 还需要确认密码。
   
    ![在 StorSimple 设备上配置 Web 代理 - 1](./media/storsimple-configure-web-proxy/IC751830.png)

如果这是首次注册自己的设备，请继续注册。 如果设备已注册，向导将会退出。 配置的设置将会保存，

同时，Web 代理已启用。 可以跳过[启用 Web 代理](#enable-web-proxy)步骤，直接转到[在 Azure 经典门户中查看 Web 代理设置](#view-web-proxy-settings-in-the-azure-classic-portal)。

## <a name="configure-web-proxy-via-windows-powershell-for-storsimple-cmdlets"></a>通过 Windows PowerShell for StorSimple cmdlet 配置 Web 代理
配置 Web 代理设置的另一种方法是通过 Windows PowerShell for StorSimple cmdlet。 执行以下步骤可配置 Web 代理。

#### <a name="to-configure-web-proxy-via-cmdlets"></a>通过 cmdlet 配置 Web 代理
1. 在串行控制台菜单中，选择选项 1“使用完全访问权限登录”。 出现提示时，请提供**设备管理员密码**。 默认密码为 `Password1`。
2. 在命令提示符处，键入：
   
    `Set-HcsWebProxy -Authentication NTLM -ConnectionURI "<http://<IP address or FQDN of web proxy server>:<TCP port number>" -Username "<Username for web proxy server>"`
   
    出现提示时，请提供并确认密码，如下所示。
   
    ![在 StorSimple 设备上配置 Web 代理 - 3](./media/storsimple-configure-web-proxy/IC751831.png)

现已配置 Web 代理，接下来需要启用它。

## <a name="enable-web-proxy"></a>启用 Web 代理
Web 代理默认已禁用。 在 StorSimple 设备上配置 Web 代理设置后，需要使用 Windows PowerShell for StorSimple 启用 Web 代理设置。

> [!NOTE]
> **如果 Web 代理是使用安装向导配置的，则不需要执行此步骤。安装向导会话结束后，默认会自动启用 Web 代理。**
> 
> 

在 Windows PowerShell for StorSimple 中执行以下步骤，在设备上启用 Web 代理：

#### <a name="to-enable-web-proxy"></a>启用 Web 代理
1. 在串行控制台菜单中，选择选项 1“使用完全访问权限登录”。 出现提示时，请提供**设备管理员密码**。 默认密码为 `Password1`。
2. 在命令提示符处，键入：
   
    `Enable-HcsWebProxy`
   
    现已在 StorSimple 设备上启用 Web 代理配置。
   
    ![在 StorSimple 设备上配置 Web 代理 - 4](./media/storsimple-configure-web-proxy/IC751832.png)

## <a name="view-web-proxy-settings-in-the-azure-classic-portal"></a>在 Azure 经典门户中查看 Web 代理设置
Web 代理设置是通过 Windows PowerShell 界面配置的，无法在经典门户中更改。 但是，可以在经典门户中查看配置的这些设置。 执行以下步骤可查看 Web 代理。

#### <a name="to-view-web-proxy-settings"></a>查看 Web 代理设置
1. 导航到“StorSimple Manager 服务”>“设备”。 选择并单击某个设备，然后转到“配置”。
2. 将“配置”页向下滚动到“Web 代理设置”部分。 可以查看 StorSimple 设备上配置的 Web 代理设置，如下所示。
   
    ![在管理门户中查看 Web 代理](./media/storsimple-configure-web-proxy/ViewWebProxyPortal_M.png)

## <a name="errors-during-web-proxy-configuration"></a>Web 代理配置期间出错
如果 Web 代理设置的配置不正确，Windows PowerShell for StorSimple 中会向用户显示错误消息。 下表描述了其中的某些错误消息、可能的原因和建议的操作。

| 序列号 | HRESULT 错误代码 | 可能的根本原因 | 建议的操作 |
|:--- |:--- |:--- |:--- |
| 1. |0x80070001 |命令从被动控制器运行，但无法与主动控制器通信。 |在主动控制器上运行命令。 若要从被动控制器运行命令，需要修复从被动控制器到主动控制器的连接。 如果此连接中断，需要咨询 Microsoft 支持。 |
| 2. |0x800710dd - 操作标识符无效 |StorSimple 虚拟设备不支持代理设置。 |StorSimple 虚拟设备不支持代理设置。 只能在 StorSimple 物理设备上配置这些设置。 |
| 3. |0x80070057 - 参数无效 |为代理设置提供的一个参数无效。 |提供的 URI 的格式不正确。 使用以下格式：`http://<IP address or FQDN of the web proxy server>:<TCP port number>` |
| 4. |0x800706ba - RPC 服务器不可用 |根本原因是以下项之一：</br></br>群集未启动。</br></br>数据路径服务未运行。</br></br>命令从被动控制器运行，但无法与主动控制器通信。 |请咨询 Microsoft 支持，确保群集可启动，数据路径服务可运行。</br></br>从主动控制器运行命令。 若要从被动控制器运行命令，需确保被动控制器可与主动控制器通信。 如果此连接中断，需要咨询 Microsoft 支持。 |
| 5. |0x800706be - RPC 调用失败 |群集已关闭。 |请咨询 Microsoft 支持，确保群集可启动。 |
| 6. |0x8007138f - 找不到群集资源 |找不到平台服务群集资源。 安装不当时可能会发生此错误。 |可能需要在设备上执行出厂重置。 可能需要创建平台资源。 请联系 Microsoft 支持了解后续措施。 |
| 7. |0x8007138c - 群集资源未联机 |平台或数据路径群集资源未联机。 |请联系 Microsoft 支持，帮助确保数据路径和平台服务资源联机。 |

> [!NOTE]
> * 上面的错误消息列表并不详尽。 
> * Web 代理设置相关的错误不会显示在 StorSimple Manager 服务的 Azure 经典门户中。 如果在完成配置后 Web 代理出现问题，经典门户中的设备状态将更改为“脱机”。|
> 
> 

## <a name="next-steps"></a>后续步骤
* 如果在部署设备或配置 Web 代理设置时遇到任何问题，请参阅[排查 StorSimple 设备的部署问题](storsimple-troubleshoot-deployment.md)。
* 若要了解如何使用 StorSimple Manager 服务，请转到[使用 StorSimple Manager 服务管理 StorSimple 设备](storsimple-manager-service-administration.md)。




<!--HONumber=Nov16_HO3-->


