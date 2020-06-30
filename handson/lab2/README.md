# Azure Sphere + Azure IoT Central で 可視化 / ルール設定 (lab2)

## 想定 : 40分

## ゴール
Azure Sphere と Azure IoT Central でテレメトリーの可視化によるインサイト、ルール設定によるアクション、をそれぞれ非常に早く実現出来ることを体験してみましょう

## 最初に

このハンズオンでは、以下のデバイス、環境が必要です:

### デバイス

- [Azure Sphere MT3620 Development Kit](https://seeedjp.github.io/Wiki/MT3620/)

### 開発環境

- Windows 10 PC (1703 以降/ 1909 または 2004 推奨)
- Azure サブスクリプションに ログイン可能な MS アカウント
- Visutal Studio 2019 (Community Edition 可)

## ステップ 1 : IoT Central アプリケーションの作成

新しい Azure IoT Central アプリケーションを作成してみましょう

1.  [Azure IoT Central](https://azure.microsoft.com/ja-jp/services/iot-central/) に移動して、 **ソリューションの構築** をクリック

    ![image](img/lab2-s1-1.png)

1. 画面をスクロールし、"カスタムアプリを作成する" をクリック

    ![image](img/lab2-s1-2.png)

1. ”アプリケーション名" に "azure sphere iot central hands on" などの **任意の文字** を、"URL" は自動生成のままで構いません、"アプリケーション テンプレート" は **カスタム アプリケーション** を、"料金プラン" は **Standard 2** を選択  

    ![image](img/lab2-s1-3.png)

    > [!TIP]  
    > "URL" は固有(Azure IoT Central 全体で同じものがない)である必要があります

1. 課金情報 はご利用の Azure サブスクリプションにより異なります  
”ディレクトリ" と "Azure サブスクリプション" 選択後、"場所" は **日本** を選択し、"作成" をクリック  

    ![image](img/lab2-s1-4.png)

    > [!TIP]  
    > アプリケーションを日本語にするには、画面右上の歯車をクリックし、"Language" を **日本語** にして "Save" をクリックします

    ![image](img/lab2-s1-5.png)

    しばらくするとアプリケーションがデプロイされ、ダッシュボードが表示されます

## ステップ 2 : 証明書 (X.509) を登録

Azure Sphere のデバイスが自動的にアプリケーションに登録されるよう、証明書を登録します

1. 画面左に並ぶアイコンの一番下 **管理** をクリック
1. 表示されたメニューから **デバイス接続** をクリック
1. 画面をスクロールさせて **プライマリ証明書の管理** をクリック

    ![image](img/lab2-s2-1.png)

1. "プライマリ" の右の (フォルダ) アイコン をクリック

    ![image](img/lab2-s2-2.png)

1. Azure Sphere テナントから出力した証明書を読み込み

    ![image](img/lab2-s2-3.png)

    > [!TIP]  
    > "Azure Sphere Developer Command Prompt" で以下のコマンドを実行します  
    > `azsphere tenant download-CA-certificate --output CAcertificate.cer`

1. "確認コード" の右の (最新の情報に更新) アイコン をクリックして確認コードを表示、そしてその右のアイコンでコピー

    ![image](img/lab2-s2-4.png)

1. 確認コードを Azure Sphere テナントに入力、署名済み検証証明書 を出力

    > [!TIP]  
    > "Azure Sphere Developer Command Prompt" で以下のコマンドを実行します  
    > `azsphere tenant download-validation-certificate --output ValidationCertification.cer --verificationcode (表示された確認コード) `

1. "確認" をクリックし、署名済み検証証明書 を指定  
1. 証明書を読み込み、**確認済み** と表示されたのを確認  
    ![image](img/lab2-s2-5.png)

以上で、お手持ちの Azure Sphere デバイスが (Azure 接続後) 自動的に登録されるようになります

## ステップ 3 : デバイステンプレートの読み込み

続いて Azure Sphere サンプルコード用に用意されたデバイス テンプレートを読み込みます

1. 画面左に並ぶアイコンの下から3番目、**デバイス テンプレート** をクリック
1. **+ 新規** をクリック (画面サイズによっては + のみ表示)

    ![image](img/lab2-s3-1.png)

1. ”おすすめのデバイス テンプレート" から **Azure Sphere Sample Device** をクリック

1. 画面下の **次へ: レビュー** をクリック

    ![image](img/lab2-s3-2.png)

1. **作成** をクリック

    ![image](img/lab2-s3-3.png)

以上で、 Azure Sphere デバイス用のテンプレート登録が完了です

## ステップ 4 : Azure Sphere 用の環境情報の取得

続いて Azure Sphere 用の環境情報を取得します

1. 画面左に並ぶアイコンの一番下、**管理** をクリック
1. 表示されたメニューから **API トークン** をクリック
1. **+ トークンの作成** をクリック (画面サイズによっては + のみ表示)

    ![image](img/lab2-s4-1.png)

1. "azure-sphere-token" など 任意の "トークン名" (スペース不可) を入力し、ロールが **管理者** であることを確認し、**生成** をクリック

    ![image](img/lab2-s4-2.png)

1. 表示された "トークン" を アイコン を クリックして コピー

    ![image](img/lab2-s4-3.png)

    > [!TIP]  
    > 再表示出来ませんので確実にコピーしてください
    > 機能するようになるまで1-2分かかる場合があります

1. ( **APIトークン** の1つ上の ) **デバイス接続** をクリックし、"ID スコープ" をコピー

    ![image](img/lab2-s4-4.png)

1. "Azure Sphere Developer Command Prompt" を開き、"Samples\AzureIoT\Tools\win-x64" に移動
1. **ShowIoTCentralConfig.exe** を実行し、画面の指示に従って項目を入力

    ![image](img/lab2-s4-5.png)

    > [!TIP]  
    > `Are you using a legacy (2018) IoT Central application (Y/N)`  
    > `Enter the IoT Central App URL (e.g. https://myapp.azureiotcentral.com)`  
    > `Enter your Azure IoT Central application API Token`
    > `Enter the ID Scope from the IoT Central App (Administration | Device connection)`  
    > の項目をそれぞれ入力すると、  
    > "CmdArgs": [ "0ne0012937F" ],  
    > "Capabilities": {  
    "AllowedConnections": [ "global.azure-devices-provisioning.net", "iotc-a4f135ec-b513-496e-8cb6-bbfe6428d03a.azure-devices.net", "iotc-afca1e33-60e2-44a4-98f9-e66932def4a9.azure-devices.net", "iotc-aaebc44f-5958-4faf-9065-fffcc12af73f.azure-devices.net", "iotc-245d9692-10dd-4527-a974-538a650cbcf1.azure-devices.net", "iotc-d3c56ca8-b27e-4aa1-afd5-313b1bc7a721.azure-devices.net", "iotc-c362d2af-6fdc-4944-a9de-7692801692d8.azure-devices.net", "iotc-ae08a6ff-72de-44f1-896e-8756c61cc7b3.azure-devices.net", "iotc-bc27b508-88cc-408d-af87-fb13cf3d0aaf.azure-devices.net", "iotc-c4ef51e2-5cff-475a-9180-00a576938312.azure-devices.net", "iotc-5acce193-dc96-4301-86b2-68ebab2f017d.azure-devices.net" ],  
    "DeviceAuthentication": "--- YOUR AZURE SPHERE TENANT ID---",}  
    > この情報を保持します  

1. "Azure Sphere Developer Command Prompt" で "Azure Sphere Tenant ID" を取得
    > `azsphere tenant show-selected`  
    > Default Azure Sphere tenant ID is '9a935dc6-a3ad-47e1-8632-c7d8430b765c'.  

## ステップ 5 : Azure Sphere サンプルアプリをビルド＆デプロイ

1. "Visutal Studio 2019" を実行、**コード無しで続行 (W)** をクリック  

    ![image](img/lab2-s5-1.png)

1. "ファイル" > "開く" > "CMake" をクリック、"Samples\AzureIoT" の "CMakeLists.txt" をクリックして **開く (O)** をクリック

    ![image](img/lab2-s5-2.png)

    > [!TIP]  
    > 最初に表示された "CMake の概要ページ" の "Device Tab" で PC に接続された Azure Sphere デバイスの詳細が確認出来ます

    ![image](img/lab2-s5-3.png)

1. 画面右の **ソリューションエクスプローラー** に表示されたファイルから **app_manifest.json** をダブルクリック

1. 開いた **app_manifest.json** に対して、"ステップ 4" で表示された内容を入力して保存  
    > [!TIP]  
    >   
    > 

    ![image](img/lab2-s5-4.png)

1. 画面右に表示されたファイルから **main.c** をダブルクリック  
1. 開いた **main.c** の624行目を以下のように修正、保存 (バグ対策)  
    > [!TIP]  
    > `\"%3.2f\"` の前後から `\"` を削除します

    ![image](img/lab2-s5-5.png)

1. "Visutal Studio" 画面中央上の **ARM-Debug** になっていることを確認、その横の "スタートアップ アイテムの選択" を **GDB Debugger (HLCore)** に変更  

    ![image](img/lab2-s5-6.png)

1. **GDB Debugger (HLCore)** をクリックし、デバッグ開始 

    ![image](img/lab2-s5-7.png)

    > [!TIP]  
    > 出力 ウィンドウ で Azure IoT Central (IoT Hub) への送信状況が確認出来ます

## ステップ 6 : Azure IoT Central で テレメトリー の可視化

Azure Sphere デバイス と デバイステンプレートを紐付け、可視化します

1. 画面左に並ぶアイコンの上から3つめ デバイス をクリック  
1. すべてのデバイス に Azure Sphere デバイス があるのを確認、"移行" をクリック

    ![image](img/lab2-s6-1.png)

1. "Azure Sphere Sample Device" をクリック、"移行" をクリック  

    ![image](img/lab2-s6-2.png)

1. Azure Sphere デバイスが (デバイス テンプレート へ移行しているので)、デバイスをクリック  

    ![image](img/lab2-s6-3.png)

1. Overview をクリックしてしばらく待つと、温度データ (シミュレート) が表示

    ![image](img/lab2-s6-4.png)


以上で、テレメトリーの可視化は完了です

## ステップ 7 : Azure IoT Central で ルール 実行

1. 画面左に並ぶアイコンの上から5番目、**規則** をクリック
1. **+ 新規** をクリック (画面サイズによっては + のみ表示)

    ![image](img/lab2-s7-1.png)

1. "温度フラグ" など、任意の規則名を入力  
1. "ターゲット デバイス" > "デバイス テンプレート" で **Azure Sphere Sample Device** を選択  

    ![image](img/lab2-s7-2.png)

1. "条件" > "テレメトリ" で **Temperature** を選択
1. "次の値より大きい" など任意の演算子を選択
1. "値" に "50" など、任意の値を入力
1. "アクション" > **+ 電子メール** をクリック

    ![image](img/lab2-s7-3.png)

1. 任意の表示名を入力 (標準のままでも可)
1. "宛先" にメールアドレスを入力

    ![image](img/lab2-s7-4.png)

    > [!TIP]    
    > この宛先アドレスは Azure IoT Central アプリ登録済みでかつ、最低1回はサインインしている必要があります

1. **作業完了** をクリック
    
1. 画面上の方にある **保存** をクリック

    ![image](img/lab2-s7-5.png)  

    しばらくして条件を満たすと、メールが送信されます  

    ![image](img/lab2-s7-6.png)  


## まとめ

Azure Sphere と Azure IoT Central によるテレメトリー可視化とルール設定の基本的な体験が完了しました  

## EXTRA : Azure CLI で Azure IoT Central をモニタリング

開発中、Azure Sphere デバイスから Azure IoT Central にテレメトリーを送信しているように見える一方で、Azure IoT Central のダッシュボードに反映されず、で クラウド側をモニタリングしたい場合があります  

その際は以下の手順にてモニタリングが可能です  

0. [Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest) をインストール

1. Azure CLI に Azure IoT 拡張機能をインストール  
`az extension add --name azure-iot`

1. "Application ID" を IoT Central アプリで確認 (管理 > お客様のアプリケーション) 

    ![image](img/lab2-s8-1.png)

1. Azure CLI でモニタリングを開始  
`az iot central app monitor-events --app-id (Application ID)`

    > [!TIP]    
    > 初回起動時に uamqp のアップデートが求められる場合があります

    ![image](img/lab2-s8-2.png)

