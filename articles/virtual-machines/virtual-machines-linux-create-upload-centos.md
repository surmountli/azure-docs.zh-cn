---
title: "在 Azure 中创建和上载基于 CentOS 的 Linux VHD"
description: "了解如何创建和上载包含基于 CentOS 的 Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
services: virtual-machines-linux
documentationcenter: 
author: szarkos
manager: timlt
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: 0e518e92-e981-43f4-b12c-9cba1064c4bb
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/09/2016
ms.author: szark
translationtype: Human Translation
ms.sourcegitcommit: ee34a7ebd48879448e126c1c9c46c751e477c406
ms.openlocfilehash: 6919a8c2147db4f8c86230e336540c8c88c714e1


---
# <a name="prepare-a-centos-based-virtual-machine-for-azure"></a>为 Azure 准备基于 CentOS 的虚拟机
* [为 Azure 准备 CentOS 6.x 虚拟机](#centos-6x)
* [为 Azure 准备 CentOS 7.0+ 虚拟机](#centos-70)

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

## <a name="prerequisites"></a>先决条件
本文假设你已在虚拟硬盘中安装了 CentOS（或类似的衍生产品）Linux 操作系统。 存在多个用于创建 .vhd 文件的工具，例如 Hyper-V 等虚拟化解决方案。 有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/library/hh846766.aspx)。

**CentOS 安装说明**

* 另请参阅[常规 Linux 安装说明](virtual-machines-linux-create-upload-generic.md#general-linux-installation-notes)，获取更多有关如何为 Azure 准备 Linux 的提示。
* Azure 不支持 VHDX 格式，仅支持**固定大小的 VHD**。  可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。
* 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。 这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。  如果需要，可以在数据磁盘上使用 [LVM](virtual-machines-linux-configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或 [RAID](virtual-machines-linux-configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 由于低于 2.6.37 的 Linux 内核版本中的 bug，更大的 VM 不支持 NUMA。 此问题主要影响使用上游 Red Hat 2.6.32 内核的分发。 手动安装的 Azure Linux 代理 (waagent) 将自动在 Linux 内核的 GRUB 配置中禁用 NUMA。 可以在下面的步骤中找到有关此内容的详细信息。
* 不要在操作系统磁盘上配置交换分区。 可以配置 Linux 代理，以在临时资源磁盘上创建交换文件。  可以在下面的步骤中找到有关此内容的详细信息。
* 所有 VHD 的大小必须是 1 MB 的倍数。

## <a name="centos-6x"></a>CentOS 6.x
1. 在 Hyper-V 管理器中，选择虚拟机。
2. 单击“连接”打开该虚拟机的控制台窗口。
3. 通过运行以下命令卸载 NetworkManager：
   
        # sudo rpm -e --nodeps NetworkManager
   
    **注意：**如果尚未安装此包，则此命令将会失败，并显示一条错误消息。 这是正常情况。
4. 在包含以下文本的 `/etc/sysconfig/` 目录中创建一个名为 **network** 的文件：
   
     NETWORKING=yes   HOSTNAME=localhost.localdomain
5. 在包含以下文本的 `/etc/sysconfig/network-scripts/` 目录中创建一个名为 **ifcfg-eth0** 的文件：
   
     DEVICE=eth0   ONBOOT=yes   BOOTPROTO=dhcp   TYPE=Ethernet   USERCTL=no   PEERDNS=yes   IPV6INIT=no
6. 修改 udev 规则，以避免产生以太网接口的静态规则。 在 Microsoft Azure 或 Hyper-V 中克隆虚拟机时，这些规则可能会引发问题：
   
   # <a name="sudo-ln--s-devnull-etcudevrulesd75-persistent-net-generatorrules"></a>sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
   # <a name="sudo-rm--f-etcudevrulesd70-persistent-netrules"></a>sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
7. 通过运行以下命令，确保网络服务将在引导时启动：
   
        # sudo chkconfig network on
8. **仅限 CentOS 6.3**：安装适用于 Linux Integration Services (LIS) 的驱动程序。
   
    **重要说明：该步骤仅适用于 CentOS 6.3 及更低版本。**  在 CentOS 6.4+ 中，*标准内核中已提供* Linux 集成服务。
   
   * 按照 [LIS 下载页](https://www.microsoft.com/en-us/download/details.aspx?id=46842)上的安装说明操作并将 RPM 安装到映像中。  
9. 通过运行以下命令安装 python-pyasn1 包：
   
        # sudo yum install python-pyasn1
10. 如果你想要使用 Azure 数据中心托管的 OpenLogic 镜像，请使用以下存储库替换 /etc/yum.repos.d/CentOS-Base.repo 文件。  这还会添加包含 Azure Linux 代理包的 **[openlogic]** 存储库：
    
        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
    
        [base]
        name=CentOS-$releasever - Base
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    
        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    
        #contrib - packages by Centos Users
        [contrib]
        name=CentOS-$releasever - Contrib
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    
    **注意：**本指南的余下部分假设你至少会使用 [openlogic] 存储库，下面将使用该存储库安装 Azure Linux 代理。
11. 将下列行添加到 /etc/yum.conf：
    
     http_caching=packages
    
    （**仅在 CentOS 6.3 上**）添加以下行：
    
     exclude=kernel*
12. 通过编辑文件“/etc/yum/pluginconf.d/fastestmirror.conf”禁用 yum 模块“fastestmirror”，并在 `[main]` 下面键入以下内容
    
        set enabled=0
13. 运行以下命令以便清除当前 yum 元数据：
    
    # <a name="yum-clean-all"></a>yum clean all
14. （**仅在 CentOS 6.3 上**）使用以下命令更新内核：
    
        # sudo yum --disableexcludes=all install kernel
15. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 为此，请在文本编辑器中打开“/boot/grub/menu.lst”，并确保默认内核包含以下参数：
    
     console=ttyS0 earlyprintk=ttyS0 rootdelay=300 numa=off
    
    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 由于 CentOS 6 所使用的内核版本中存在 Bug，因此这将禁用 NUMA。
    
    除此之外，建议*删除*以下参数：
    
     rhgb quiet crashkernel=auto
    
    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。
    
    根据需要可以配置 `crashkernel` 选项，但请注意此参数会使虚拟机中的可用内存量减少 128MB 或更多，这在较小的虚拟机上可能会出现问题。
16. 请确保已安装 SSH 服务器且已将其配置为在引导时启动。  这通常是默认设置。
17. 通过运行以下命令来安装 Azure Linux 代理：
    
        # sudo yum install WALinuxAgent
    
    请注意，如果没有如步骤 2 中所述删除 NetworkManager 包和 NetworkManager-gnome 包，则安装 WALinuxAgent 包将删除它们。
18. 不要在 OS 磁盘上创建交换空间。
    
    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是*临时*磁盘，并可能在取消预配 VM 时被清空。 在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：
    
     ResourceDisk.Format=y  ResourceDisk.Filesystem=ext4  ResourceDisk.MountPoint=/mnt/resource  ResourceDisk.EnableSwap=y  ResourceDisk.SwapSizeMB=2048    ## 注意：将此项设置为所需的内容。
19. 运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：
    
    # <a name="sudo-waagent--force--deprovision"></a>sudo waagent -force -deprovision
    # <a name="export-histsize0"></a>export HISTSIZE=0
    # <a name="logout"></a>logout
20. 在 Hyper-V 管理器中单击“操作”->“关闭”。 Linux VHD 现已准备好上载到 Azure。

- - -
## <a name="centos-70"></a>CentOS 7.0+
**CentOS 7（和类似衍生产品）中的更改**

为 Azure 准备 CentOS 7 虚拟机非常类似于 CentOS 6，但有几个值得注意的重要区别：

* NetworkManager 包不再与 Azure Linux 代理冲突。 默认情况下将安装此包，建议你不要删除它。
* GRUB2 现在用作默认引导加载程序，因此用于编辑内核参数的过程已更改（请参见下文）。
* XFS 现在是默认文件系统。 如果需要，仍可以使用 ext4 文件系统。

**配置步骤**

1. 在 Hyper-V 管理器中，选择虚拟机。
2. 单击“连接”打开该虚拟机的控制台窗口。
3. 在包含以下文本的 `/etc/sysconfig/` 目录中创建一个名为 **network** 的文件：
   
     NETWORKING=yes   HOSTNAME=localhost.localdomain
4. 在包含以下文本的 `/etc/sysconfig/network-scripts/` 目录中创建一个名为 **ifcfg-eth0** 的文件：
   
     DEVICE=eth0   ONBOOT=yes   BOOTPROTO=dhcp   TYPE=Ethernet   USERCTL=no   PEERDNS=yes   IPV6INIT=no
5. 修改 udev 规则，以避免产生以太网接口的静态规则。 在 Microsoft Azure 或 Hyper-V 中克隆虚拟机时，这些规则可能会引发问题：
   
   # <a name="sudo-ln--s-devnull-etcudevrulesd75-persistent-net-generatorrules"></a>sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
6. 通过运行以下命令，确保网络服务将在引导时启动：
   
        # sudo chkconfig network on
7. 通过运行以下命令安装 python-pyasn1 包：
   
        # sudo yum install python-pyasn1
8. 如果你想要使用 Azure 数据中心托管的 OpenLogic 镜像，请使用以下存储库替换 /etc/yum.repos.d/CentOS-Base.repo 文件。  这还会添加包含 Azure Linux 代理包的 **[openlogic]** 存储库：
   
        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
   
        [base]
        name=CentOS-$releasever - Base
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   
        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   
        #contrib - packages by Centos Users
        [contrib]
        name=CentOS-$releasever - Contrib
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

    **注意：**本指南的余下部分假设你至少会使用 [openlogic] 存储库，下面将使用该存储库安装 Azure Linux 代理。

1. 运行以下命令以清除当前 yum 元数据并安装所有更新：
   
   # <a name="sudo-yum-clean-all"></a>sudo yum clean all
   # <a name="sudo-yum--y-update"></a>sudo yum -y update
2. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 为此，请在文本编辑器中打开“/etc/default/grub”并编辑 `GRUB_CMDLINE_LINUX` 参数，例如：
   
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 此外，还会关闭 NIC 的新 CentOS 7 命名约定。 除此之外，建议*删除*以下参数：
   
    rhgb quiet crashkernel=auto
   
   图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。
   
   根据需要可以配置 `crashkernel` 选项，但请注意此参数会使虚拟机中的可用内存量减少 128MB 或更多，这在较小的虚拟机上可能会出现问题。
3. 完成后，请按照上面所示编辑“/etc/default/grub”，运行以下命令以重新生成 grub 配置：
   
       # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
4. 请确保已安装 SSH 服务器且已将其配置为在引导时启动。  这通常是默认设置。
5. **仅当从 VMWare、VirtualBox 或 KVM 生成映像时：**将 Hyper-V 模块添加到 initramfs 中：
   
   编辑 `/etc/dracut.conf`，添加内容：
   
    add_drivers+=”hv_vmbus hv_netvsc hv_storvsc”
   
   重新生成 initramfs：
   
   # <a name="dracut-f--v"></a>dracut –f -v
6. 通过运行以下命令来安装 Azure Linux 代理：
   
       # sudo yum install WALinuxAgent
       # sudo systemctl enable waagent
7. 不要在 OS 磁盘上创建交换空间。
   
   Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是*临时*磁盘，并可能在取消预配 VM 时被清空。 在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：
   
    ResourceDisk.Format=y  ResourceDisk.Filesystem=ext4  ResourceDisk.MountPoint=/mnt/resource  ResourceDisk.EnableSwap=y  ResourceDisk.SwapSizeMB=2048    ## 注意：将此项设置为所需的内容。
8. 运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：
   
   # <a name="sudo-waagent--force--deprovision"></a>sudo waagent -force -deprovision
   # <a name="export-histsize0"></a>export HISTSIZE=0
   # <a name="logout"></a>logout
9. 在 Hyper-V 管理器中单击“操作”->“关闭”。 Linux VHD 现已准备好上载到 Azure。

## <a name="next-steps"></a>后续步骤
现在，可以使用 CentOS Linux 虚拟硬盘在 Azure 中创建新的 Azure 虚拟机。 如果这是第一次将 .vhd 文件上载到 Azure，请参阅[创建和上载包含 Linux 操作系统的虚拟硬盘](virtual-machines-linux-classic-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)中的步骤 2 和步骤 3。




<!--HONumber=Nov16_HO3-->

