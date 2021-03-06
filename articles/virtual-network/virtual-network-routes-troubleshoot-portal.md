---
title: ルートのトラブルシューティング | Microsoft Docs
description: Azure Resource Manager のデプロイメント モデルで、Azure ポータルを使用してルートをトラブルシューティングする方法について説明します。
services: virtual-network
documentationcenter: na
author: AnithaAdusumilli
manager: narayan
editor: ''
tags: azure-resource-manager

ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/23/2016
ms.author: anithaa

---
# <a name="troubleshoot-routes-using-the-azure-portal"></a>Azure ポータルを使用してルートのトラブルシューティングを行う
> [!div class="op_single_selector"]
> * [Azure ポータル](virtual-network-routes-troubleshoot-portal.md)
> * [PowerShell](virtual-network-routes-troubleshoot-powershell.md)
> 
> 

Azure 仮想マシン (VM) とのネットワーク接続に問題が発生した場合、VM トラフィック フローがルートの影響を受けている場合があります。 この記事では、詳細なトラブルシューティングを行うためのルートの診断機能の概要を説明します。

ルート テーブルはサブネットに関連付けられ、そのサブネットのすべてのネットワーク インターフェイス (NIC) に対して有効になります。 各ネットワーク インターフェイスには、次の種類のルートを適用できます。

* **システム ルート:** 既定では、Azure Virtual Network (VNet) で作成されたすべてのサブネットが、ローカル VNet トラフィック、VPN ゲートウェイ経由のオンプレミスのトラフィック、インターネット トラフィックを利用できるシステム ルート テーブルを持ちます。 ピアリングされる VNet にもシステム ルートが存在します。
* **BGP ルート:** ExpressRoute またはサイト間 VPN 接続経由のネットワーク インターフェイスに反映されます。 BGP ルーティングの詳細については、「[Azure VPN ゲートウェイを使用した BGP の概要](../vpn-gateway/vpn-gateway-bgp-overview.md)」と「[ExpressRoute の技術概要](../expressroute/expressroute-introduction.md)」の記事を参照してください。
* **ユーザー定義のルート (UDR):** ネットワーク仮想アプライアンスまたは強制トンネリングを使用して、オンプレミス ネットワークにサイト間 VPN 経由でトラフィックをルートしている場合は、サブネット ルート テーブルにユーザー定義のルート (UDR) が関連付けられていることがあります。 UDR に慣れていない場合は、「 [ユーザー定義のルート](virtual-networks-udr-overview.md#user-defined-routes) 」の記事をご覧ください。

ネットワーク インターフェイスに適用できるさまざまなルートを使用する場合、有効な集約ルートの特定が難しくなることがあります。 VM ネットワーク接続をトラブルシューティングするため、Azure Resource Manager デプロイメント モデルのネットワーク インターフェイスのすべての有効なルートを表示できます。

## <a name="using-effective-routes-to-troubleshoot-vm-traffic-flow"></a>有効なルートを使用した VM トラフィック フローのトラブルシューティング
この記事では、例として次のシナリオを使用し、ネットワーク インターフェイスの有効な規則のトラブルシューティング方法を説明します。

VNet (*VNet1*、プレフィックス: 10.9.0.0/16) に接続する VM (*VM1*) が、新たにピアリングされた VNet (*VNet3*、プレフィックス 10.10.0.0/16) の VM (VM3) に接続できません。 VM に接続する VM1-NIC1 ネットワーク インターフェイスには UDR や BGP ルートが適用されておらず、システム ルートのみが適用されています。

この記事では、Azure Resource Management デプロイ モデルの有効なルート機能を使用した、接続エラーの原因の特定方法を説明します。
この例ではシステム ルートのみを使用していますが、次の手順は任意の種類のルート経由での受信/送信接続の失敗の確認に使用できます。

> [!NOTE]
> VM に複数の NIC が接続されているときは、各 NIC の有効なルートを確認して、VM とのネットワーク接続の問題を診断します。
> 
> 

### <a name="view-effective-routes-for-a-virtual-machine"></a>仮想マシンの有効なルートを表示する
VM に適用されている集約ルートを表示するには、次の手順を実行します。

1. https://portal.azure.com で Azure ポータルにログインします。
2. **[その他のサービス]** をクリックし、表示される一覧で **[仮想マシン]** をクリックします。
3. 表示される一覧からトラブルシューティングを実行する VM を選択すると、VM のブレードとオプションが表示されます。
4. **[問題の診断と解決]** をクリックし、一般的な問題を選択します。 例として、 **[Windows VM に接続できません]** を選択します。 
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image1.png)
5. 次の図のように、問題の下に手順が表示されます。 
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image2.png)
   
    推奨される手順の一覧で、 *[effective routes (有効なルート)]* をクリックします。
6. 次の図のように、 **[Effective routes (有効なルート)]** ブレードが表示されます。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image3.png)
   
    VM に NIC が 1 つしかない場合は、既定でその NIC が選択されます。 複数の NIC がある場合は、表示する有効なルートの NIC を選択します。
   
   > [!NOTE]
   > NIC に関連付けられている VM が実行中でない場合は、有効なルートは表示されません。 ポータルでは、効果的なルートのうち最初の 200 件のみが表示されます。 完全な一覧が必要な場合は、 **[ダウンロード]**をクリックします。 ダウンロードした .csv ファイルの結果をさらにフィルター処理できます。
   > 
   > 
   
    出力内の次の情報にご注意ください。
   
   * **Source**: ルートのタイプを示します。 システム ルートは *Default*、UDR は *User*、ゲートウェイ ルート (静的または BGP) は *VPNGateway* と表示されます。
   * **State**: 有効なルートの状態を示します。 *Active* か *Invalid* のいずれかの値になります。
   * **AddressPrefixes**: CIDR 表記で有効なルートのアドレス プレフィックスを指定します。 
   * **nextHopType**: 特定のルートの次のホップを示します。 *VirtualAppliance*、*Internet*、*VNetLocal*、*VNetPeering*、*Null* のいずれかの値になります。 UDR の *nextHopType* の値が **Null** の場合、ルートが無効であることを表します。 たとえば、 **nextHopType** が *VirtualAppliance* で、ネットワーク仮想アプライアンス VM がプロビジョニング/実行中の状態でない場合です。 **nextHopType** が *VPNGateway* で、特定の VNet のゲートウェイがプロビジョニング/実行中の状態でない場合も、ルートが無効になることがあります。
7. 前の手順の図には、*WestUS-VNET3* (プレフィックス 10.9.0.0/16) から *WestUS-VNet1* VNet (プレフィックス 10.10.0.0/16) へのルートはリストされていません。 次の図では、ピアリング リンクの状態が *Disconnected* になっています。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image4.png)
   
    ピアリングの双方向リンクが切断されています。VM1 が *WestUS-VNet3* VNet の VM3 に接続できないのはそのためです。
8. 次の図は、双方向のピアリング リンクが確立された後のルートを示しています。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image5.png)

強制トンネリングとルート評価のためのトラブルシューティング シナリオについては、この記事の「 [考慮事項](virtual-network-routes-troubleshoot-portal.md#Considerations) 」セクションをご覧ください。

### <a name="view-effective-routes-for-a-network-interface"></a>ネットワーク インターフェイスの有効なルートを表示する
特定のネットワーク インターフェイス (NIC) のネットワーク トラフィック フローが影響を受ける場合は、有効なルートの完全な一覧を NIC で直接表示できます。 NIC に適用されている集約ルートを表示するには、次の手順を実行します。

1. https://portal.azure.com で Azure ポータルにログインします。
2. **[その他のサービス]** をクリックし、**[ネットワーク インターフェイス]** をクリックします。
3. NIC の名前の一覧を検索するか、表示された一覧から選択します。 この例では、 **VM1-NIC1** が選択されています。
4. 次の図にあるように、**[ネットワーク インターフェイス]** ブレードで **[Effective routes]** (有効なルート) を選択します。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image6.png)
   
    **[スコープ]** の既定値が選択したネットワーク インターフェイスに設定されています。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image7.png)

### <a name="view-effective-routes-for-a-route-table"></a>ルート テーブルの有効なルートを表示する
ルート テーブル内のユーザー定義のルート (UDR) を変更する際には、特定の VM に追加されるルートの影響を確認できます。 ルート テーブルは、任意の数のサブネットに関連付けることができます。 特定のルート テーブルが適用されているすべての NIC の有効なルートの完全な一覧を表示できるようになります。特定のルート テーブル ブレードからコンテキストを切り替える必要はありません。

この例では、ルート テーブル (*UDRouteTable*) で UDR (*UDRoute*) が指定されています。 このルートでは、*WestUS-VNet1* VNet の *Subnet1* からのすべてのインターネット トラフィックが、同じ VNet の *Subnet2* のネットワーク仮想アプライアンス (NVA) 経由で送信されます。 次の図にはルートが示されています。

![](./media/virtual-network-routes-troubleshoot-portal/image8.png)

ルート テーブルの集約ルートを表示するには、次の手順を実行します。

1. https://portal.azure.com で Azure ポータルにログインします。
2. **[その他のサービス]** をクリックし、**[ルート テーブル]** をクリックします。
3. 集約ルートを表示するルート テーブルの一覧を検索し、選択します。 この例では **UDRouteTable** が選択されています。 次の図に示すように、選択したルート テーブルのブレードが表示されます。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image9.png)
4. **[ルート テーブル]** ブレードで **[Effective Routes]** (有効なルート) を選択します。 **[スコープ]** は、選択したルート テーブルに設定されます。
5. ルート テーブルは複数のサブネットに適用できます。 一覧から、確認する **サブネット** を選択します。 この例では、 **Subnet1** が選択されています。
6. **ネットワーク インターフェイス**を選択します。 選択したサブネットに接続されているすべての NIC の一覧が表示されます。 この例では、 **VM1-NIC1** が選択されています。
   
    ![](./media/virtual-network-routes-troubleshoot-portal/image10.png)
   
   > [!NOTE]
   > NIC が実行中の VM に関連付けられていない場合は、有効なルートは表示されません。
   > 
   > 

## <a name="considerations"></a>考慮事項
返されたルートの一覧を確認する際は、次の点にご注意ください。

* UDR、BGP、システム ルートにおいては、ルーティングは最長プレフィックス一致 (LPM) に基づいて行われます。 同じ LPM マッチの複数のルートが存在する場合は、そのルートが検出された経緯に応じて次の順序でルートが選択されます。
  
  * ユーザー定義のルート
  * BGP のルート
  * システム (既定) のルート
    
    有効なルートを使用すると、使用可能なすべてのルートに対して LPM マッチの有効なルートのみが表示されます。 特定の NIC に対してルートが実際に評価される方法を表示することによって、VM との接続に影響を与える可能性のある特定のルートのトラブルシューティングをより簡単に行うことができます。
* UDR があり、**nextHopType** の *VirtualAppliance* を使用して、ネットワーク仮想アプライアンス (NVA) にトラフィックを送信している場合は、トラフィックを受信する NVA で IP 転送が有効であることを確認してください。有効になっていない場合、パケットが破棄されます。 
* 強制トンネリングが有効になっている場合、発信されるインターネット トラフィックはオンプレミスにルーティングされます。 オンプレミスによるトラフィックの処理方法によっては、インターネットから VM への RDP/SSH は、この設定では機能しない場合があります。 
  強制トンネリングは、次の方法で有効にできます。
  * VPN ゲートウェイとして nextHopType を使用するユーザー定義のルート (UDR) を設定して、サイト間 VPN を使用する場合
  * 既定のルートが BGP 経由で提供される場合
* VNet のピアリング トラフィックが正常に機能するためには、ピアリングされる VNet のプレフィックスの範囲に **nextHopType** *VNetPeering* を使用するシステム ルートを追加する必要があります。 そのようなルートが存在せず、VNet ピアリングのリンクに問題がないと考えられる場合:
  * 新しく確立されたピアリングのリンクの場合は、数秒待ってから再度実行してください。 サブネット内のすべてのネットワーク インターフェイスにルートを反映させるのに時間がかかる場合がります。
  * ネットワーク セキュリティ グループ (NSG) の規則が、トラフィック フローに影響を与える可能性があります。 詳細については、「 [ネットワーク セキュリティ グループのトラブルシューティング](virtual-network-nsg-troubleshoot-portal.md) 」をご覧ください。

<!--HONumber=Oct16_HO2-->


