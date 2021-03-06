---
title: "Windows VM に対する計画的なメンテナンス | Microsoft Docs"
description: "Azure の計画的なメンテナンスの概要と、それが Azure で実行されている Windows 仮想マシンに及ぼす影響について説明します。"
services: virtual-machines-windows
documentationcenter: 
author: drewm
manager: timlt
editor: 
tags: azure-service-management,azure-resource-manager
ms.assetid: eb4b92d8-be0f-44f6-a6c3-f8f7efab09fe
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/26/2016
ms.author: drewm
translationtype: Human Translation
ms.sourcegitcommit: 5919c477502767a32c535ace4ae4e9dffae4f44b
ms.openlocfilehash: f81d4a407e738aa8133e91a24f06feba5af75e63


---
# <a name="planned-maintenance-for-virtual-machines-in-azure"></a>Azure での仮想マシンに対する計画的なメンテナンス
Azure の計画的なメンテナンスの概要と、それが Windows 仮想マシンの可用性に及ぼす影響について説明します。 この記事は、 [Linux 仮想マシン](virtual-machines-linux-planned-maintenance.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)にも利用できます。 

この記事では、Azure の計画的なメンテナンス プロセスに関する基本情報を提供します。 VM が再起動した原因についてトラブルシューティングを行う場合は、 [VM の再起動ログの表示について詳述しているこのブログ記事をご覧ください](https://azure.microsoft.com/blog/viewing-vm-reboot-logs/)。

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

## <a name="why-azure-performs-planned-maintenance"></a>Azure が計画的なメンテナンスを実行する理由
Microsoft Azure は、世界各地で定期的に更新を行い、仮想マシンの基盤となるホスト インフラストラクチャの信頼性、パフォーマンス、セキュリティの向上に努めています。 これらの更新の多くは、仮想マシンや Cloud Services に影響を及ぼさずに実行されます (たとえば、メモリ保護更新など)。

ただし、一部の更新では、インフラストラクチャに必須の更新を適用するときに仮想マシンの再起動が必要になります。 その場合、インフラストラクチャにパッチを適用する間、仮想マシンはシャットダウンされ、その後、仮想マシンの再起動が行われます。

仮想マシンの可用性に影響を及ぼす可能性があるメンテナンスには、計画的なメンテナンスと計画外のメンテナンスの 2 種類があります。 このページでは、Microsoft Azure による計画的なメンテナンスの実行について説明します。 計画外のメンテナンスの詳細については、「[計画済み、または計画外メンテナンスについて理解する](virtual-machines-windows-manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)」を参照してください。

[!INCLUDE [virtual-machines-common-planned-maintenance](../../includes/virtual-machines-common-planned-maintenance.md)]




<!--HONumber=Nov16_HO3-->


