---
title: Azure SQL Data Warehouse の復元 (ポータル) | Microsoft Docs
description: Azure SQL Data Warehouse を復元するための Azure ポータル タスク。
services: sql-data-warehouse
documentationcenter: NA
author: Lakshmi1812
manager: barbkess
editor: ''

ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 09/21/2016
ms.author: lakshmir;barbkess;sonyama

---
# <a name="restore-an-azure-sql-data-warehouse-(portal)"></a>Azure SQL Data Warehouse の復元 (ポータル)
> [!div class="op_single_selector"]
> * [概要][概要]
> * [ポータル][ポータル]
> * [PowerShell][PowerShell]
> * [REST ()][REST ()]
> 
> 

この記事では、Azure ポータルを使用して Azure SQL Data Warehouse を復元する方法について説明します。

## <a name="before-you-begin"></a>開始する前に
**DTU 容量を確認します。**  各 SQL Data Warehouse は、既定の DTU クォータが割り当てられている SQL サーバー (例: myserver.database.windows.net) でホストされます。  SQL Data Warehouse を復元する前に、データベースの復元に必要な量の DTU クォータがSQL server に残っていることを確認してください。 必要な DTU を計算する方法と DTU を要求する方法については、 [DTU クォータの変更の要求][DTU クォータの変更の要求]に関するトピックをご覧ください。

## <a name="restore-an-active-or-paused-database"></a>アクティブまたは一時停止中のデータベースを復元する
データベースを復元するには:

1. [Azure ポータル][Azure ポータル]
2. 画面の左側にある **[参照]** をクリックし、**[SQL Server]** を選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/01-browse-for-sql-server.png)
3. 目的のサーバーに移動して選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/01-select-server.png)
4. 復元する SQL Data Warehouse を見つけて選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/01-select-active-dw.png)
5. Data Warehouse ブレードの上部にある **[復元]**
   
    ![](./media/sql-data-warehouse-restore-database-portal/01-select-restore-from-active.png)
6. 新しい **データベース名**
7. 最新の **復元ポイント**
   
   1. 最新の復元ポイントを選択していることを確認します。  復元ポイントは UTC で表示されるため、表示される既定のオプションが最新の復元ポイントではない場合があります。
      
      ![](./media/sql-data-warehouse-restore-database-portal/01-restore-blade-from-active.png)
8. **[OK]**
9. データベースの復元処理が開始され、 **[通知]**

> [!NOTE]
> 復元が完了したら、「 [復旧後のデータベースの構成][復旧後のデータベースの構成]」の手順に従って、復旧されたデータベースを構成できます。
> 
> 

## <a name="restore-a-deleted-database"></a>削除されたデータベースの復元
削除されたデータベースを復元するには:

1. [Azure ポータル][Azure ポータル]
2. 画面の左側にある **[参照]** をクリックし、**[SQL Server]** を選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/01-browse-for-sql-server.png)
3. 目的のサーバーに移動して選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/02-select-server.png)
4. サーバーのブレードで下にスクロールして [操作] セクションに移動します
5. **[削除済みデータベース]** タイルをクリックします
   
    ![](./media/sql-data-warehouse-restore-database-portal/02-select-deleted-dws.png)
6. 復元する削除済みデータベースを選択します
   
    ![](./media/sql-data-warehouse-restore-database-portal/02-select-deleted-dw.png)
7. 新しい **データベース名**
   
    ![](./media/sql-data-warehouse-restore-database-portal/02-restore-blade-from-deleted.png)
8. **[OK]**
9. データベースの復元処理が開始され、 **[通知]**

> [!NOTE]
> 復元が完了した後にデータベースを構成する方法については、「 [復旧後のデータベースの構成][復旧後のデータベースの構成]」を参照してください。 
> 
> 

## <a name="next-steps"></a>次のステップ
Azure SQL Database の各エディションのビジネス継続性機能については、 [Azure SQL Database のビジネス継続性の概要][Azure SQL Database のビジネス継続性の概要]に関するページをご覧ください。

<!--Image references-->

<!--Article references-->
[Azure SQL Database のビジネス継続性の概要]: ./sql-database-business-continuity.md
[概要]: ./sql-data-warehouse-restore-database-overview.md
[ポータル]: ./sql-data-warehouse-restore-database-portal.md
[PowerShell]: ./sql-data-warehouse-restore-database-powershell.md
[REST ()]: ./sql-data-warehouse-restore-database-rest-api.md
[復旧後のデータベースの構成]: ./sql-database-disaster-recovery.md#configure-your-database-after-recovery
[DTU クォータの変更の要求]: ./sql-data-warehouse-get-started-create-support-ticket.md#request-quota-change

<!--MSDN references-->

<!--Blog references-->

<!--Other Web references-->
[Azure ポータル]: https://portal.azure.com/



<!--HONumber=Oct16_HO2-->


