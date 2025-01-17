---
lab:
    title: '05 - サイト間接続を実装する'
    module: 'モジュール 05 - サイト間接続'
---

# ラボ 05 - サイト間接続を実装する
# 受講者用ラボ マニュアル

## ラボ シナリオ

Contoso 社は、ボストン、ニューヨーク、シアトルの各オフィスにデータセンターを持ち、メッシュ型のワイドエリア ネットワーク リンクで接続されており、それらの間では完全な接続性があります。Contoso社 のオンプレミス ネットワークのトポロジを反映したラボ環境を実装して、その機能を検証します。 

## 目標

このラボでは、次の内容を学習します。

+ タスク 1: ラボ環境をプロビジョニングする
+ タスク 2: ローカルとグローバルの仮想ネットワークのピアリングを構成する
+ タスク 3: サイト間の接続性をテストする 

## 想定時間: 30分間

### 手順

#### タスク 1: ラボ環境をプロビジョニングする

このタスクでは、3 つの仮想マシンをそれぞれ別の仮想ネットワークにデプロイし、そのうちの 2 つを同じ Azure リージョンに配置し、3 つ目の仮想マシンを別の Azure リージョンにデプロイします。 

1. [Azure portal](https://portal.azure.com) にログインします。

1. Azure portal で右上にあるアイコンをクリックして **Azure Cloud Shell** を開きます。

1. **Bash** または **PowerShell** のいずれかを選択するように求められた場合、**PowerShell**を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 をクリックします。 

1. 「Cloud Shell」 ペインのツール バーで 「**ファイルのアップロード/ダウンロード**」 アイコンをクリックし、ドロップダウン メニューで 「**アップロードする**」 をクリックして、「**\\Allfiles\\Labs\\05\\az104-05-vnetvm-template.json**」 ファイルと 「**\\Allfiles\\Labs\\05\\az104-05-vnetvm-parameters.json**」 ファイルを 「Cloud Shell」 のホーム ディレクトリにアップロードします。

1. 「Cloud Shell」 ペインで次のコマンドを実行し、最初の仮想ネットワークと仮想マシンのペアをホストする最初のリソース グループを作成します。プレースホルダー `[Azure_region_1]` は、これらのAzure 仮想マシンをデプロイする **可用性ゾーンに対応した**Azure リージョンの名前に置き換えます。

   ```pwsh
   $location = '[Azure_region_1]'

   $rgName = 'az104-05-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
   >**注**: Azure リージョンを識別するには、クラウド シェルの PowerShell セッションから **(Get-AzLocation).Location** を実行します。
   
1. 「Cloud Shell」 ペインから、次のコマンドを実行して最初の仮想ネットワークを作成し、アップロードしたテンプレートとパラメーター ファイルを使用して仮想マシンをデプロイします。

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 0 `
      -AsJob
   ```
1. 「Cloud Shell」 ペインで次を実行して、2 つ目の仮想ネットワークと 2 つ目の仮想マシンをホストする 2 つ目のリソースグループを作成します。

   ```pwsh
   $rgName = 'az104-05-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して 2 つ目の仮想ネットワークを作成し、アップロードしたテンプレートとパラメーター ファイルを使用して仮想マシンをデプロイします。

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 1 `
      -AsJob
   ```
1. 「Cloud Shell」 ペインで次のコマンドを実行して、3 つ目の仮想ネットワークと 3 つ目の仮想マシンをホストする 3 つ目のリソース グループを作成します。プレースホルダー `[Azure_region_2]` は、他の 2 つのデプロイで使用した Azure リージョンとは異なる Azure 仮想マシンをデプロイできる、別の可用性ゾーンに対応した Azure リージョンの名前に置き換えます。

   ```pwsh
   $location = '[Azure_region_2]'

   $rgName = 'az104-05-rg2'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 「Cloud Shell」 ペインで、次のコマンドを実行して 3 つ目の仮想ネットワークを作成し、アップロードしたテンプレートとパラメーター ファイルを使用して仮想マシンをデプロイします。

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 2 `
      -AsJob
   ```
    >**注意**: 次のタスクを進める前に、スクリプトが完了するのを待ちます。これには約 2 分かかります。

    >**注**: デプロイの状態を確認するには、このタスクで作成したリソース グループのプロパティを調べます。

1. 「Cloud Shell」 ペインを閉じます。

#### タスク 2: ローカルとグローバルの仮想ネットワークのピアリングを構成する

このタスクでは、前のタスクでデプロイした仮想ネットワーク間で、ローカル ピアリングとグローバル ピアリングを構成します。

1. Azure ポータルで、 「**仮想ネットワーク**」を検索して選択します。 

1. 前のタスクで作成した仮想ネットワークを確認し、最初の 2 つが同じ Azure リージョンに配置され、3 つ目が別の Azure リージョンに存在することを確認します。 

    >**注**: 3 つの仮想ネットワークのデプロイに使用したテンプレートにより、3 つの仮想ネットワークの IP アドレス範囲が重複しないようにしています。

1. 仮想ネットワークの一覧で、「**az104-05-vnet0**」 をクリックします。

1. 「**az104-05-vnet0** 仮想ネットワーク」 ブレードの 「**設定**」 セクションで 「**ピアリング(Peerings)**」 をクリックしてから、「**+ 追加(Add)**」 をクリックします。
    >**注**: この手順では、**az104-05-vnet0** から **az104-05-vnet1**、**az104-05-vnet1** から **az104-05-vnet0** までの双方向の 2 つのローカル ピアリングを確立します。

1. 次の設定で **This virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet0_to_az104-05-vnet1** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. 次の設定で **Remote virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet1_to_az104-05-vnet0** |
    | Virtual network deployment model | **Resource Manager** |
    | Subscription | このラボで使用している Azure サブスクリプションの名前 |
    | Virtual Network | **az104-05-vnet1 (az104-05-rg1)** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. **Add** をクリックして保存します。

1. 再度「**az104-05-vnet0** 仮想ネットワーク」 ブレードの 「**設定**」 セクションで 「**ピアリング**」 をクリックしてから、「**+ 追加**」 をクリックします。
    >**注**: この手順では、**az104-05-vnet0** から **az104-05-vnet2**、**az104-05-vnet2** から **az104-05-vnet0** までの双方向の 2 つのローカル ピアリングを確立します。

1. 次の設定で **This virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet0_to_az104-05-vnet2** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. 次の設定で **Remote virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet2_to_az104-05-vnet0** |
    | Virtual network deployment model | **Resource Manager** |
    | Subscription | このラボで使用している Azure サブスクリプションの名前 |
    | Virtual Network | **az104-05-vnet2 (az104-05-rg2)** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. **Add** をクリックして保存します。

1. 「**仮想ネットワーク**」 ブレードに戻り、仮想ネットワークの一覧で 「**az104-05-vnet1**」 をクリックします。

1. 「**az104-05-vnet1** 仮想ネットワーク」 ブレードの 「**設定**」 セクションで 「**ピアリング**」 をクリック し、「**+ 追加**」 をクリックします。
    >**注**: この手順では、**az104-05-vnet1** から **az104-05-vnet2**、**az104-05-vnet2** から **az104-05-vnet1** までの双方向の 2 つのローカル ピアリングを確立します。

1. 次の設定で **This virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet1_to_az104-05-vnet2** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. 次の設定で **Remote virtual network** の設定を行います (その他の設定は既定値のままにします)。

    | 設定 | 値|
    | --- | --- |
    | ピアリングの名前(Peering link name) | **az104-05-vnet2_to_az104-05-vnet1** |
    | Virtual network deployment model | **Resource Manager** |
    | Subscription | このラボで使用している Azure サブスクリプションの名前 |
    | Virtual Network | **az104-05-vnet2 (az104-05-rg2)** |
    | Traffic to remote virtual network | **Allow(default)** |
    | Traffic forwarded from remote virtual network | **Block traffic that originates from outside this virtual network** |
    | Virtual network gateway | **None (default)** |

1. **Add** をクリックして保存します。

#### タスク 3: サイト間の接続性をテストする 

このタスクでは、前のタスクでローカル ピアリングとグローバル ピアリングを介して接続した 3 つの仮想ネットワーク上の仮想マシン間の接続性をテストします。

1. Azure portal で、「**仮想マシン**」 を検索して選択します。

1. 仮想マシンの一覧で、**az104-05-vm0**をクリックします。 

1. 「**az104-05-vm0**」 ブレードで 「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP で接続**」 ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、プロンプトに従ってリモート デスクトップ セッションを開始します。

    >**注**: この手順は、Windows コンピューターからリモート デスクトップ経由で接続することを指します。Mac では、Mac App Store からリモート デスクトップ クライアントを使用でき、Linux コンピューターでは、オープンソースの RDP クライアント ソフトウェアを使用できます。

    >**注**: ターゲットの仮想マシンに接続する際は、警告メッセージを無視できます。

1. プロンプトが表示されたら、ユーザー名 「**Student**」 とパスワード 「**Pa55w.rd1234**」 を使用してログインします。

1. **az104-05-vm0** へのリモート デスクトップ セッション内で、「**スタート**」 ボタンを右クリックし、右クリック メニューで 「**Windows PowerShell (管理者)**」 をクリックします。

1. Windows PowerShell コンソール ウィンドウで次のコマンドを実行して 、TCP ポート 3389 での 「**az104-05-vm1**」 (プライベート IP アドレスが **10.51.0.4** ) への接続性をテストします。

   ```pwsh
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**メモ**: このテストで TCP 3389 を使用するのは、このポートがオペレーティング システムのファイアウォールによって既定で許可されているためです。 

1. コマンドの出力を調べて、接続が正常に行われたことを確認します。

1. Windows PowerShell コンソール ウィンドウで次のコマンドを実行して、「**az104-05-vm2**」 (プライベート IP アドレスが **10.52.0.4**) への接続性をテストします。

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
     
1. ラボ コンピューターの Azure portal に戻り、「**仮想マシン**」 ブレードに戻ります。 

1. 仮想マシンの一覧で、「**az104-05-vm1**」 をクリックします。

1. 「**az104-05-vm1**」 ブレードで 「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP で接続**」 ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、プロンプトに従ってリモート デスクトップ セッションを開始します。

    >**注**: この手順は、Windows コンピューターからリモート デスクトップ経由で接続することを指します。Mac では、Mac App Store からリモート デスクトップ クライアントを使用でき、Linux コンピューターでは、オープンソースの RDP クライアント ソフトウェアを使用できます。

    >**注**: ターゲットの仮想マシンに接続する際は、警告メッセージを無視してください。

1. プロンプトが表示されたら、ユーザー名 「**Student**」 とパスワード 「**Pa55w.rd1234**」 を使用してログインします。

1. 「**az104-05-vm1** へのリモート デスクトップ」 セッション内で 「**スタート**」 ボタンを右クリックし、右クリック メニューで 「**Windows PowerShell (管理者)**」 をクリックします。

1. Windows PowerShell コンソール ウィンドウで次のコマンドを実行して、TCP ポート 3389 での 「**az104-05-vm2**」 (プライベート IP アドレスが **10.52.0.4** ) への接続性をテストします。

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**注意**: このテストで TCP 3389 を使用するのは、このポートがオペレーティング システムのファイアウォールによって既定で許可されているためです。 

1. コマンドの出力を調べて、接続が正常に行われたことを確認します。

#### リソースのクリーンアップ

   >**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用していないリソースを削除することで、予期しないコストが発生しなくなります。

1. Azure portalの 「**Cloud Shell**」 ウインドウで、**PowerShell** セッションを開始します。

1. 次のコマンドを実行して、このモジュールのラボ全体で作成されたすべてのリソース グループを一覧表示します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*'
   ```

1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループを削除します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**注意**: コマンドは非同期に実行されるため (-AsJob パラメーターによって決まります)、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボで学習した内容は以下のとおりです。

- ラボ環境の構成
- ローカル仮想ネットワーク ピアリングおよびグローバル仮想ネットワーク ピアリングの構成
- サイト間の接続性のテスト
