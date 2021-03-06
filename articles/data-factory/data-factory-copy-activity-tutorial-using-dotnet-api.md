---
title: "チュートリアル: コピー アクティビティがあるパイプラインを .NET API で作成する | Microsoft Docs"
description: "このチュートリアルでは、.NET API を使用して、コピー アクティビティがある Azure Data Factory パイプラインを作成します。"
services: data-factory
documentationcenter: 
author: spelluru
manager: jhubbard
editor: monicar
ms.assetid: 58fc4007-b46d-4c8e-a279-cb9e479b3e2b
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/27/2016
ms.author: spelluru
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 629ff68b11df0d17629ca101e5a80a396cfd0fb9


---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-net-api"></a>チュートリアル: コピー アクティビティがあるパイプラインを .NET API で作成する
> [!div class="op_single_selector"]
> * [概要と前提条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [コピー ウィザード](data-factory-copy-data-wizard-tutorial.md)
> * [Azure ポータル](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager テンプレート](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

このチュートリアルでは、.NET API を使用して Azure データ ファクトリを作成および監視する方法について説明しています。 データ ファクトリのパイプラインは、コピー アクティビティを使用して、Azure Blob Storage から Azure SQL Database にデータをコピーします。

コピー アクティビティにより、Azure Data Factory でデータ移動が実行されます。 このアクティビティは、安全で信頼性の高いスケーラブルな方法によってさまざまなデータ ストア間でデータをコピーできる、グローバルに利用可能なサービスによって動作します。 コピー アクティビティの詳細については、「 [データ移動アクティビティ](data-factory-data-movement-activities.md) 」をご覧ください。   

> [!NOTE]
> この記事では、すべての Data Factory .NET API を取り上げているわけではありません。 Data Factory .NET SDK の詳細については、 [Data Factory .NET API リファレンス](https://msdn.microsoft.com/library/mt415893.aspx) のページを参照してください。 
> 
> 

## <a name="prerequisites"></a>前提条件
* [チュートリアルの概要と前提条件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) のページに目を通して、チュートリアルの概要を理解し、 **前提条件** の手順を完了します。 
* Visual Studio 2012 または 2013 または 2015
*  [Azure .NET SDK](http://azure.microsoft.com/downloads/)
* Azure PowerShell。 「 [Azure PowerShell のインストールと構成の方法](../powershell-install-configure.md) 」に記載されている手順に従って、コンピューターに Azure PowerShell をインストールします。 Azure PowerShell を使用して、Azure Active Directory アプリケーションを作成します。

### <a name="create-an-application-in-azure-active-directory"></a>Azure Active Directory にアプリケーションを作成する
Azure Active Directory アプリケーションを作成し、アプリケーションのサービス プリンシパルを作成して、 **Data Factory 共同作成者** ロールに割り当てます。  

1. **PowerShell**を起動します。 
2. 次のコマンドを実行して、Azure ポータルへのサインインに使用するユーザー名とパスワードを入力します。
   
        Login-AzureRmAccount   
3. 次のコマンドを実行して、このアカウントのすべてのサブスクリプションを表示します。
   
        Get-AzureRmSubscription 
4. 次のコマンドを実行して、使用するサブスクリプションを選択します。 **&lt;NameOfAzureSubscription**&gt; を自分の Azure サブスクリプションの名前で置き換えます。 
   
        Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext
   
   > [!IMPORTANT]
   > このコマンドの出力から、**SubscriptionId** と **TenantId** をメモしておきます。 
   > 
   > 
5. PowerShell で次のコマンドを実行して、 **ADFTutorialResourceGroup** という名前の Azure リソース グループを作成します。  
   
        New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
   
    リソース グループが既に存在する場合は、それを更新するか (Y) そのまま保持するか (N) を指定します。 
   
    異なるリソース グループを使用する場合は、このチュートリアルで ADFTutorialResourceGroup の代わりにそのリソース グループの名前を使用する必要があります。
6. Azure Active Directory アプリケーションを作成します。 
   
        $azureAdApplication = New-AzureRmADApplication -DisplayName "ADFCopyTutotiralApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfcopytutorialapp.org/example" -Password "Pass@word1"
   
    次のエラーが表示された場合は、別の URL を指定して、コマンドをもう一度実行します。 
   
        Another object with the same value for property identifierUris already exists.
7. AD サービス プリンシパルを作成します。 
   
        New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
8. **Data Factory 共同作成者** ロールにサービス プリンシパルを追加します。 
   
        New-AzureRmRoleAssignment -RoleDefinitionName "Data Factory Contributor" -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
9. アプリケーション ID を取得します。
   
        $azureAdApplication
   
    アプリケーション ID (出力の**applicationID** ) をメモします。

これらの手順で、次の 4 つの値がわかりました。 

* テナント ID
* サブスクリプション ID
* アプリケーション ID 
* パスワード (最初のコマンドで指定)   

## <a name="walkthrough"></a>チュートリアル
1. Visual Studio 2012/2013/2015 を使用して、C# .NET コンソール アプリケーションを作成します。
   1. **Visual Studio** 2012/2013/2015 を起動します。
   2. **[ファイル]** をクリックし、**[新規作成]** をポイントして、**[プロジェクト]** をクリックします。
   3. **[テンプレート]** を展開し、**[Visual C#]** を選択します。 このチュートリアルでは C# を使用しますが、どの .NET 言語でも使用できます。
   4. 右側にあるプロジェクトの種類の一覧から **[コンソール アプリケーション]** を選択します。
   5. [名前] に「 **DataFactoryAPITestApp** 」と入力します。
   6. [場所] で **[C:\ADFGetStarted]** を選択します。
   7. **[OK]** をクリックしてプロジェクトを作成します。
2. **[ツール]** をクリックし、**[NuGet パッケージ マネージャー]** をポイントして、**[パッケージ マネージャー コンソール]** をクリックします。
3. **パッケージ マネージャー コンソール**で、次の手順を実行します。 
   1. 次のコマンドを実行して、Data Factory パッケージをインストールします: `Install-Package Microsoft.Azure.Management.DataFactories`        
   2. 次のコマンドを実行して、Azure Active Directory パッケージをインストールします (コードで Active Directory API を使用します): `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213`
4. 次の **appSetttings** セクションを **App.config** ファイルに追加します。 これらの設定は、ヘルパー メソッド **GetAuthorizationHeader**によって使用されます。 
   
    **&lt;Application ID&gt;**、**&lt;Password&gt;**、**&lt;Subscription ID&gt;**、**&lt;tenant ID&gt;** の値を実際の値に置き換えます。 
   
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <startup> 
                <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
            </startup>
            <appSettings>
                <add key="ActiveDirectoryEndpoint" value="https://login.windows.net/" />
                <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
                <add key="WindowsManagementUri" value="https://management.core.windows.net/" />
   
                <add key="ApplicationId" value="your application ID" />
                <add key="Password" value="Password you used while creating the AAD application" />
                <add key="SubscriptionId" value= "Subscription ID" />
                <add key="ActiveDirectoryTenantId" value="Tenant ID" />
            </appSettings>
        </configuration>
5. 次の **using** ステートメントをプロジェクト内のソース ファイル (Program.cs) に追加します。
   
        using System.Threading;
        using System.Configuration;
        using System.Collections.ObjectModel;
   
        using Microsoft.Azure.Management.DataFactories;
        using Microsoft.Azure.Management.DataFactories.Models;
        using Microsoft.Azure.Management.DataFactories.Common.Models;
   
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure;
6. **DataPipelineManagementClient** クラスのインスタンスを作成する次のコードを **Main** メソッドに追加します。 このオブジェクトを使用して、データ ファクトリ、リンクされたサービス、入力データセットと出力データセット、およびパイプラインを作成します。 また、実行時にデータセットのスライスを監視する際にもこのオブジェクトを使用します。    
   
            // create data factory management client
            string resourceGroupName = "ADFTutorialResourceGroup";
            string dataFactoryName = "APITutorialFactory";
   
            TokenCloudCredentials aadTokenCredentials =
                new TokenCloudCredentials(
                    ConfigurationManager.AppSettings["SubscriptionId"],
                    GetAuthorizationHeader());
   
            Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);
   
            DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
   
   > [!IMPORTANT]
   > **resourceGroupName** の値を、使用する Azure リソース グループの名前で置き換えます。 
   > 
   > データ ファクトリの名前 (**dataFactoryName**) が一意になるように更新します。 データ ファクトリの名前はグローバルに一意にする必要があります。 Data Factory アーティファクトの名前付け規則については、 [Data Factory - 名前付け規則](data-factory-naming-rules.md) に関するトピックを参照してください。 
   > 
   > 
7. **データ ファクトリ**を作成する次のコードを **Main** メソッドに追加します。
   
            // create a data factory
            Console.WriteLine("Creating a data factory");
            client.DataFactories.CreateOrUpdate(resourceGroupName,
                new DataFactoryCreateOrUpdateParameters()
                {
                    DataFactory = new DataFactory()
                    {
                        Name = dataFactoryName,
                        Location = "westus",
                        Properties = new DataFactoryProperties() { }
                    }
                }
            );
8. **Azure Storage のリンクされたサービス**を作成する次のコードを **Main** メソッドに追加します。 
   
   > [!IMPORTANT]
   > **storageaccountname** と **accountkey** を Azure ストレージ アカウントの名前とキーで置き換えます。 
   > 
   > 
   
            // create a linked service for input data store: Azure Storage
            Console.WriteLine("Creating Azure Storage linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureStorageLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accountkey>")
                        )
                    }
                }
            );
9. **Azure SQL のリンクされたサービス**を作成する次のコードを **Main** メソッドに追加します。
   
   > [!IMPORTANT]
   > **servername**、**databasename**、**username**、**password** を Azure SQL のサーバー名、データベース名、ユーザー名、パスワードで置き換えます。  
   > 
   > 
   
            // create a linked service for output data store: Azure SQL Database
            Console.WriteLine("Creating Azure SQL Database linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureSqlLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            new AzureSqlDatabaseLinkedService("Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30")
                        )
                    }
                }
            );
10. **入力データセットと出力データセット**を作成する次のコードを **Main** メソッドに追加します。 
    
            // create input and output datasets
            Console.WriteLine("Creating input and output datasets");
            string Dataset_Source = "DatasetBlobSource";
            string Dataset_Destination = "DatasetAzureSqlDestination";
    
            Console.WriteLine("Creating input dataset of type: Azure Blob");
            client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,

            new DatasetCreateOrUpdateParameters()
            {
                Dataset = new Dataset()
                {
                    Name = Dataset_Source,
                    Properties = new DatasetProperties()
                    {
                        Structure = new List<DataElement>()
                        {
                            new DataElement() { Name = "FirstName", Type = "String" },
                            new DataElement() { Name = "LastName", Type = "String" }
                        },
                        LinkedServiceName = "AzureStorageLinkedService",
                        TypeProperties = new AzureBlobDataset()
                        {
                            FolderPath = "adftutorial/",
                            FileName = "emp.txt"
                        },
                        External = true,
                        Availability = new Availability()
                        {
                            Frequency = SchedulePeriod.Hour,
                            Interval = 1,
                        },

                        Policy = new Policy()
                        {
                            Validation = new ValidationPolicy()
                            {
                                MinimumRows = 1
                            }
                        }
                    }
                }
            });


            Console.WriteLine("Creating output dataset of type: Azure SQL");
            client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new DatasetCreateOrUpdateParameters()
                {
                    Dataset = new Dataset()
                    {
                        Name = Dataset_Destination,
                        Properties = new DatasetProperties()
                        {
                            Structure = new List<DataElement>()
                            {
                                new DataElement() { Name = "FirstName", Type = "String" },
                                new DataElement() { Name = "LastName", Type = "String" }
                            },
                            LinkedServiceName = "AzureSqlLinkedService",
                            TypeProperties = new AzureSqlTableDataset()
                            {
                                TableName = "emp"
                            },

                            Availability = new Availability()
                            {
                                Frequency = SchedulePeriod.Hour,
                                Interval = 1,
                            },
                        }
                    }
                });

1. **パイプラインを作成してアクティブにする**次のコードを **Main** メソッドに追加します。 このパイプラインには、ソースとして **BlobSource**、シンクとして **BlobSink** を使用する **CopyActivity** があります。
   
           // create a pipeline
           Console.WriteLine("Creating a pipeline");
           DateTime PipelineActivePeriodStartTime = new DateTime(2016, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
           DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
           string PipelineName = "ADFTutorialPipeline";
   
           client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
               new PipelineCreateOrUpdateParameters()
               {
                   Pipeline = new Pipeline()
                   {
                       Name = PipelineName,
                       Properties = new PipelineProperties()
                       {
                           Description = "Demo Pipeline for data transfer between blobs",
   
                   // Initial value for pipeline's active period. With this, you won't need to set slice status
                   Start = PipelineActivePeriodStartTime,
                           End = PipelineActivePeriodEndTime,
   
                           Activities = new List<Activity>()
                           {
                               new Activity()
                               {
                                   Name = "BlobToAzureSql",
                                   Inputs = new List<ActivityInput>()
                                   {
                                       new ActivityInput() {
                                           Name = Dataset_Source
                                       }
                                   },
                                   Outputs = new List<ActivityOutput>()
                                   {
                                       new ActivityOutput()
                                       {
                                           Name = Dataset_Destination
                                       }
                                   },
                                   TypeProperties = new CopyActivity()
                                   {
                                       Source = new BlobSource(),
                                       Sink = new BlobSink()
                                       {
                                           WriteBatchSize = 10000,
                                           WriteBatchTimeout = TimeSpan.FromMinutes(10)
                                       }
                                   }
                               }
                           },
                       }
                   }
               });    
2. 次のコードを **Main** メソッドに追加して、出力データセットのデータ スライスのステータスを取得します。 この例で予想されるスライスのみが存在します。   
   
           // Pulling status within a timeout threshold
           DateTime start = DateTime.Now;
           bool done = false;
   
           while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
           {
               Console.WriteLine("Pulling the slice status");
               // wait before the next status check
               Thread.Sleep(1000 * 12);
   
               var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Dataset_Destination,
                   new DataSliceListParameters()
                   {
                       DataSliceRangeStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                       DataSliceRangeEndTime = PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString()
                   });
   
               foreach (DataSlice slice in datalistResponse.DataSlices)
               {
                   if (slice.State == DataSliceState.Failed || slice.State == DataSliceState.Ready)
                   {
                       Console.WriteLine("Slice execution is done with status: {0}", slice.State);
                       done = true;
                       break;
                   }
                   else
                   {
                       Console.WriteLine("Slice status is: {0}", slice.State);
                   }
               }
           }
3. データ スライスの実行の詳細を取得する次のコードを **Main** メソッドに追加します。
   
           Console.WriteLine("Getting run details of a data slice");
   
           // give it a few minutes for the output slice to be ready
           Console.WriteLine("\nGive it a few minutes for the output slice to be ready and press any key.");
           Console.ReadKey();
   
           var datasliceRunListResponse = client.DataSliceRuns.List(
                   resourceGroupName,
                   dataFactoryName,
                   Dataset_Destination,
                   new DataSliceRunListParameters()
                   {
                       DataSliceStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString()
                   }
               );
   
           foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
           {
               Console.WriteLine("Status: \t\t{0}", run.Status);
               Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
               Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
               Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
               Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
               Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
               Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
           }
   
           Console.WriteLine("\nPress any key to exit.");
           Console.ReadKey();
4. **Main** メソッドで使用される次のヘルパー メソッドを **Program** クラスに追加します。  
   
       public static string GetAuthorizationHeader()
       {
           AuthenticationResult result = null;
           var thread = new Thread(() =>
           {
               try
               {
                   var context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
   
                   ClientCredential credential = new ClientCredential(ConfigurationManager.AppSettings["ApplicationId"], ConfigurationManager.AppSettings["Password"]);
                   result = context.AcquireToken(resource: ConfigurationManager.AppSettings["WindowsManagementUri"], clientCredential: credential);
               }
               catch (Exception threadEx)
               {
                   Console.WriteLine(threadEx.Message);
               }
           });
   
           thread.SetApartmentState(ApartmentState.STA);
           thread.Name = "AcquireTokenThread";
           thread.Start();
           thread.Join();
   
           if (result != null)
           {
               return result.AccessToken;
           }
   
           throw new InvalidOperationException("Failed to acquire token");
       }  
5. ソリューション エクスプローラーで、プロジェクト (**DataFactoryAPITestApp**) を展開し、**[参照]** を右クリックして **[参照の追加]** をクリックします。 "**System.Configuration**" アセンブリのチェック ボックスをオンにして、**[OK]** をクリックします。 
6. コンソール アプリケーションをビルドします。 メニューから **[ビルド]** をクリックし、**[ソリューションのビルド]** をクリックします。 
7. Azure BLOB ストレージ内の **adftutorial** コンテナーに少なくとも 1 つのファイルが存在することを確認します。 存在しない場合は、以下の内容を記述した **Emp.txt** ファイルをメモ帳で作成し、adftutorial コンテナーにアップロードします。
   
       John, Doe
       Jane, Doe
8. メニューの **[デバッグ]** -> **[デバッグの開始]** の順にクリックして、サンプルを実行します。 **[Getting run details of a data slice (データ スライスの実行の詳細を取得中)]** が表示されている場合は、数分待機して、**Enter** を押します。 
9. Azure ポータルを使用して、データ ファクトリの **APITutorialFactory** が次のアーティファクトで作成されることを確認します。 
   * リンクされたサービス: **LinkedService_AzureStorage** 
   * データセット: **DatasetBlobSource** と **DatasetBlobDestination**
   * パイプライン: **PipelineBlobSample** 
10. 指定した Azure SQL データベースの "**emp**" テーブルに 2 つの従業員レコードが作成されることを確認します。

## <a name="next-steps"></a>次のステップ
* [データ移動アクティビティ](data-factory-data-movement-activities.md)に関する記事に目を通してください。この記事には、このチュートリアルで使用したコピー アクティビティの詳細が記載されています。
* Data Factory .NET SDK の詳細については、 [Data Factory .NET API リファレンス](https://msdn.microsoft.com/library/mt415893.aspx) のページを参照してください。 この記事では、すべての Data Factory .NET API を取り上げているわけではありません。 




<!--HONumber=Nov16_HO2-->


