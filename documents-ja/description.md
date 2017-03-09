# TFS・VSTS・ローカルで動作するデータベースを使った単体テストの作り方・量産方法 (Visual Studio 2017・2015・2013対応)
## 概要
TFSプロジェクトを作成して開発を開始するとき、まず最初にやるべき事がいくつかあります。特に 単体テスト をどうするか？を後回しにして「とりあえず」開発を始めてしまうと、後で取り返しがつかなくなることがあります（例えば、単体テストを作るのに時間がかかったり、後から単体テストを作ることがそもそも不可能、など）。

これを解決するために、「単体テストの基本的な作り方」と「量産方法」をサンプルとしてまとめました。

これを先に意識した上で、目的のメソッドを設計・実装することで、単体テストを作りやすい開発を行うことが出来ます。また、これを意識せずに開発してしまったメソッドについても、Fakes を使って無理矢理単体テストを行う方法も示しています。

最も頭を悩ませる、データベースへの接続を単体テスト時に切り替える方法も示しています。

これらのテストは全て ローカルの Visual Studio 2017, 2015, 2013, Team Foundation Server, Visual Studio Team Services 上で動作します。ビルド定義を作成して、「手動」あるいは「日次」で自動ビルド・自動テストを行う方法も解説します。これにより、チームの誰かがある日突然ソースコードを壊しても、すぐに発見することが出来るようになります。

このサンプルと同様に実際の案件のプロジェクトを構成することで、効率的に単体テストの「保守」を行うことが出来ます。

## サンプルの使い方
サンプルは テスト対象のプロジェクト（ SampleClassLibrary）と、単体テストのサンプルのプロジェクト（ SampleClassLibrary.Test ）の二つで構成されています。

![ソリューション・プロジェクト](description/1.png)

Test で終わる名称は、TFS/VSTS の自動ビルド・自動テストのために必要な命名規則です（変更も可能ですが、面倒です）。単体テストプロジェクトは必ず、対象の名前 .Test という名前をつけるようにします。

Visual Studio 2017 (あるいは 2015 や 2013) でソリューションを開き、[ テスト(S) > 実行(R) > 全てのテスト(A) ] を実行してください。[ テスト(S) > ウィンドウ(W) > テスト エクスプローラー(T) ] にテスト結果が表示されます。全ての結果が成功（緑のチェックマークのアイコン）になれば動作成功です。

![テストエクスプローラー](description/2.png)

テスト結果を選択して [出力] を押すと、標準出力に詳細な情報が出力されているのが確認できます。

![テスト結果 詳細](description/3.png)

このテストではコンストラクタ引数、プロパティ、テストしたメソッドの引数、期待値と結果がすべて表示されてます。

![テスト結果 出力](description/4.png)

一見、冗長に感じるかもしれませんが、実際に日次・週次・月次で単体テストを自動実行させるときには、最低限これらの情報を出力しておかないと、いざテストが失敗になった時になぜ失敗になったのか？すぐにはわかりません。

この出力を簡単にするためのライブラリをサンプルでは使用しています。

## サンプルの説明
ソリューションに含まれるサンプルの内容は次の通りです。順に参照することで、単体テストの基礎・量産方法・Fakes の使用方法（あるいはFakesしなくてもいいクラスの設計）・DBを使ったテストの方法が学べるようになっています。

![ソリューションエクスプローラー](description/5.png)
### Simpleフォルダ
基本的な単体テストの例です。

- Static メソッドのテストの例（単体テストの基本・量産方法） 
- インスタンスのメソッドのテストの例 

### サンプルで使用する拡張機能 ( Visual Studio ギャラリー )
単体テストの量産には EditorPlus 拡張機能の Format Text 機能を使用しています。

#### EditorPlus
[https://visualstudiogallery.msdn.microsoft.com/af8f350c-b992-464f-b9f3-580b51545f67](https://visualstudiogallery.msdn.microsoft.com/af8f350c-b992-464f-b9f3-580b51545f67)

使い方説明とテンプレートファイルを Simple/TestCodeTemplate に用意しました。

### SystemFakes フォルダ
Microsoft Fakes フレームワークを使う単体テストの例です。
 - 日付を指定できるメソッドのテストの例（Fakes は未使用） 
 - Fakes で System.DateTime.Now を置き換えなければならないテストの例（Fakes 使用） 

Microsoft Fakes は Visual Studio 2017, 2015 Enterprise あるいは Visual Studio 2013 Premium 以上でしか使用できません。Fakes を使用するか、使用しなくても動作するクラスを実装するか？（後者をおすすめしますが）最初に検討する必要があるでしょう。

### UseDatabaseフォルダ
データベースを使ったテストの例です。
 - DB接続文字列を指定できるメソッドのテストの例（Fakesは未使用） 
 - 接続文字列を App.config から参照するメソッドのテストの例（Fakesを使用） 

データベースは mdfファイル のアタッチ、DACPAC、BACPAC からのインポートの3種類の方法で初期化しています。初期データが入っているべきか？それともスキーマだけで良いのか？また、テスト時の実行速度は mdf が最高速で、対してDACPAC,　BACPAC はそれぞれ遅く、Visual Studio Team Services のビルド時間を多く消費してしまいます。それらを考慮した上で、データベースをどのように使用するか？検討します。

### サンプルで使用するライブラリ(NuGet)
このテストのサンプルには、Surviveplus.UnitTestPlus というライブラリを使用しています。

#### Surviveplus.UnitTestPlus
[https://www.nuget.org/packages/Surviveplus.UnitTestPlus/](https://www.nuget.org/packages/Surviveplus.UnitTestPlus/)

このライブラリの役割は以下の点です。
 - 標準出力に出力する内容の標準化 
 - テスト実行時の DB の切り替え（MDF、DACPAC、BACPAC の自動アタッチ・インポート） 
 
## TFS・VSTS 自動ビルドの構成
単体テストを Team Foundation Server/Visual Studio Team Services で実行するには、TFSプロジェクトを作成して、サンプルをソース管理に追加してください。

チェックイン後、TFS・VSTSの Build & Release を開きます。

![VSTS Build & Release](description/6.png)

Builds の [New definition] を押すとビルド定義を作成出来ます。

![Create new build definition](description/7.png)

![既定のビルド定義](description/8.png)

「Visual Studioビルド」を選択して作成すると、既定で全てのソースがビルドされ、 *test*.dll という名称でビルドされるプロジェクトのテストが実行されます。

![Visual Studio Versionの選択](description/9.png)

ひとつだけ注意しなければならないのは「Visual Studio Version」をプロジェクトに合わせて変更することです。[Save] を押して保存します。名前を適切に指定して保存してください。

![ビルド キューに追加](description/10.png)

保存したビルド定義の [Queue new build...] をクリックするか、Builds画面の一覧で [...]をクリックして表示されるメニューから [Queue new build...] を右クリックして、キューに追加することで新しいビルドを TFS/VSTS で実行できます。

![ビルド開始](description/11.png)

ローカルでは数秒で終わるこのサンプルのテストですが、Visual Studio Team Services での実行にはおおよそ10分程度かかります。キューに積まれてから実際に実行されるまで時間がかかる場合があります。

![ビルド結果 1](description/12.png)

テストが成功するとこのように確認することが出来ます。

![ビルド結果 2](description/13.png)

ビルドの成功率が VSTS の最初の画面の Activity （右中央）に表示されます。

![成功率 Activity](description/14.png)

また HOME Overview（ホーム・概要）にタイルとして「Chart for Build History」や「Test results tend」を追加して表示しておくことが出来ます。

![ビルド結果 Home](description/15.png)

ビルド定義の Triggers を変更すると、毎日自動ビルド・自動テストを行うように出来ます。（BUILD には無料時間枠がありますが、それを超えると有料になるか、ビルドが出来なくなるので注意が必要してください）


## ステップアップのための練習課題
- 実際の運用をイメージして、テストが失敗するようにメソッドを変更してみましょう。出力 から内容を確認することが出来るでしょうか。 
- ローカルでDBテストに失敗したとき、テストフォルダに MDF ファイルが残ったままになります。ログにファイルのパスが書かれているので、参照し、SQL Server Management Studio などでアタッチして内容を確認することが出来るでしょうか。 
- Visual Studio Team Services で失敗したときも、詳細に内容が表示されます。Visual Studio Team Services の日次ビルドを構成して、ホーム画面にピンしましょう。 
- DBを使った単体テストを EditorPlus を使って量産してみましょう。 
- サンプルを参考に、Surviveplus.UnitTestPlus を使って新しい単体テストを作成してみましょう。 
