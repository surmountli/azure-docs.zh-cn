<properties 
	pageTitle="在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器"
	description="本教程指导你在 Azure 中使用内部负载平衡器 (ILB) 创建 AlwaysOn 可用性组侦听器。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />
<tags 
	ms.service="virtual-machines"
	ms.date="08/11/2015"
	wacn.date="" />

# 在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器

> [AZURE.SELECTOR]
- [Internal Listener](/documentation/articles/virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener)
- [External Listener](/documentation/articles/virtual-machines-sql-server-configure-public-alwayson-availability-group-listener)

## 概述

本主题说明如何使用**内部负载平衡器 (ILB)** 为 AlwaysOn 可用性组配置侦听器。

你的可用性组可以仅包含本地副本或 Azure 副本，也可以跨越本地和 Azure 以实现混合配置。Azure 副本可以位于同一区域，也可以跨越使用多个虚拟网络 (VNet) 的多个区域。以下步骤假设你已[配置了一个可用性组](https://msdn.microsoft.com/zh-cn/library/azure/dn249504.aspx)但是没有配置侦听器。

请注意对 Azure 中使用 ILB 的可用性组侦听器的以下限制：

- Windows Server 2008 R2、Windows Server 2012 和 Windows Server 2012 R2 支持可用性组侦听器。

- 客户端应用程序必须位于与包含你的可用性组 VM 的云服务不同的云服务中。Azure 不支持客户端和服务器位于同一个云服务中的直接服务器返回。

- 每个云服务只支持一个可用性组侦听器，因为该侦听器将配置为使用云服务 VIP 地址或内部负载平衡器的 VIP 地址。尽管 Azure 现在支持在给定的云服务中创建多个 VIP 地址，但此限制仍然有效。

>[AZURE.NOTE]本教程重点介绍如何使用 PowerShell 为包括 Azure 副本的可用性组创建侦听器。有关如何使用 SSMS 或 Transact-SQL 配置侦听器的详细信息，请参阅[创建或配置可用性组侦听器](https://msdn.microsoft.com/zh-cn/library/hh213080.aspx)。

## 确定侦听器的可访问性

[AZURE.INCLUDE [ag-listener-accessibility](../includes/virtual-machines-ag-listener-determine-accessibility.md)]

本文重点介绍如何创建使用**内部负载平衡器 (ILB)** 的侦听器。如果你需要一个公共/外部侦听器，请参阅本文的另一个版本，其中提供了有关设置[外部侦听器](/documentation/articles/virtual-machines-sql-server-configure-public-alwayson-availability-group-listener)的步骤

## 创建支持直接服务器返回的负载平衡 VM 终结点

对于 ILB，必须先创建内部负载平衡器。以下脚本将执行此操作。

[AZURE.INCLUDE [load-balanced-endpoints](../includes/virtual-machines-ag-listener-load-balanced-endpoints.md)]

1. 对于 **ILB**，应该分配一个静态 IP 地址。首先，通过运行以下命令检查当前 VNet 配置：

		(Get-AzureVNetConfig).XMLConfiguration

1. 先记下包含副本所在虚拟机的子网的 **Subnet** 名称。脚本中的 **$SubnetName** 参数将要使用此值。

1. 然后，记下包含副本所在虚拟机的子网的 **VirtualNetworkSite** 名称和起始 **AddressPrefix**。再通过将这两个值传递给 **Test-AzureStaticVNetIP** 命令并检查 **AvailableAddresses** 来查找可用的 IP 地址。例如，如果 VNet 名为 *MyVNet*，并包含起始自 *172.16.0.128* 的子网地址范围，则以下命令将列出可用地址：

		(Test-AzureStaticVNetIP -VNetName "MyVNet"-IPAddress 172.16.0.128).AvailableAddresses

1. 选择一个可用地址，并在以下脚本的 **$ILBStaticIP** 参数中使用它。

3. 将以下 PowerShell 脚本复制到文本编辑器中，并根据你的环境设置变量值（注意，这里为某些参数提供了默认值）。请注意，使用地缘组的现有部署不能添加 ILB。有关 ILB 要求的详细信息，请参阅[内部负载平衡器](/documentation/articles/load-balancer-internal-overview)。此外，如果可用性组跨 Azure 区域，则你必须在每个数据中心内对云服务和节点运行该脚本一次。

		# Define variables
		$ServiceName = "<MyCloudService>" # the name of the cloud service that contains the availability group nodes
		$AGNodes = "<VM1>","<VM2>","<VM3>" # all availability group nodes containing replicas in the same cloud service, separated by commas
		$SubnetName = "<MySubnetName>" # subnet name that the replicas use in the VNet
		$ILBStaticIP = "<MyILBStaticIPAddress>" # static IP address for the ILB in the subnet
		$ILBName = "AGListenerLB" # customize the ILB name or use this default value
		
		# Create the ILB
		Add-AzureInternalLoadBalancer -InternalLoadBalancerName $ILBName -SubnetName $SubnetName -ServiceName $ServiceName -StaticVNetIPAddress $ILBStaticIP
		
		# Configure a load balanced endpoint for each node in $AGNodes using ILB
		ForEach ($node in $AGNodes)
		{
			Get-AzureVM -ServiceName $ServiceName -Name $node | Add-AzureEndpoint -Name "ListenerEndpoint" -LBSetName "ListenerEndpointLB" -Protocol tcp -LocalPort $EndpointPort -PublicPort $EndpointPort -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ILBName -DirectServerReturn $true | Update-AzureVM 
		}

1. 设置变量后，将脚本从文本编辑器复制到 Azure PowerShell 会话中运行。如果提示符仍然显示 >>，请再次按 Enter，以确保脚本开始运行。注意：

>[AZURE.NOTE]Azure 管理门户目前不支持内部负载平衡器，因此你在门户中看不到 ILB 或终结点。但是，如果负载平衡器在某个内部 IP 地址上运行，则 **Get-AzureEndpoint** 将返回该地址。否则，将返回 null。

## 如果需要，请验证是否已安装 KB2854082

[AZURE.INCLUDE [kb2854082](../includes/virtual-machines-ag-listener-kb2854082.md)]

## 在可用性组节点中打开防火墙端口

[AZURE.INCLUDE [防火墙](../includes/virtual-machines-ag-listener-open-firewall.md)]

## 创建可用性组侦听器

[AZURE.INCLUDE [防火墙](../includes/virtual-machines-ag-listener-create-listener.md)]

1. 对于 ILB，必须使用前面创建的内部负载平衡器 (ILB) 的 IP 地址。在 PowerShell 中使用以下脚本获取此 IP 地址。

		# Define variables
		$ServiceName="<MyServiceName>" # the name of the cloud service that contains the AG nodes
		(Get-AzureInternalLoadBalancer -ServiceName $ServiceName).IPAddress

1. 在某个 VM 上，将以下 PowerShell 脚本复制到文本编辑器中，将变量设置为之前记下的值。

		# Define variables
		$ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
		$IPResourceName = "<IPResourceName>" # the IP Address resource name 
		$ILBIP = “<X.X.X.X>” # the IP Address of the Internal Load Balancer (ILB)
		
		Import-Module FailoverClusters
		
		# If you are using Windows Server 2012 or higher, use the Get-Cluster Resource command. If you are using Windows Server 2008 R2, use the cluster res command. Both commands are commented out. Choose the one applicable to your environment and remove the # at the beginning of the line to convert the comment to an executable line of code. 
		
		# Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"OverrideAddressMatch"=1;"EnableDhcp"=0}
		# cluster res $IPResourceName /priv enabledhcp=0 overrideaddressmatch=1 address=$ILBIP probeport=59999  subnetmask=255.255.255.255

1. 设置变量之后，打开提升的 Windows PowerShell 窗口，然后从文本编辑器复制脚本，并将其粘贴到 Azure PowerShell 会话中运行。如果提示符仍然显示 >>，请再次按 Enter，以确保脚本开始运行。

2. 在每个 VM 上重复此过程。此脚本将使用云服务的 IP 地址来配置 IP 地址资源，同时设置探测端口等其他参数。在 IP 地址资源联机后，它可以响应我们在本教程前面部分创建的负载平衡终结点在探测端口上的轮询。

## 使侦听器联机

[AZURE.INCLUDE [Bring-Listener-Online](../includes/virtual-machines-ag-listener-bring-online.md)]

## 后续操作项

[AZURE.INCLUDE [Follow-up](../includes/virtual-machines-ag-listener-follow-up.md)]

## 测试可用性组侦听器（在同一 VNet 中）

[AZURE.INCLUDE [Test-Listener-Within-VNET](../includes/virtual-machines-ag-listener-test.md)]

## 后续步骤

[AZURE.INCLUDE [Listener-Next-Steps](../includes/virtual-machines-ag-listener-next-steps.md)]

<!---HONumber=70-->