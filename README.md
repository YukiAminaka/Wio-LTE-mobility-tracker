## 開発手順
Arduino CLIを使った基本的な開発手順

### 1. Arduino CLIのインストール
```
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
```
バージョンの確認
```bash
arduino-cli version
# arduino-cli  Version: 1.5.1 Commit: 01f3d4f2ba7c2eaafb5dc710c8a1903af7762fea Date:
```

### 2. 設定ファイルを作成する
arduino-cli の全体設定ファイルを初期化。初回だけ実行
```
arduino-cli config init
```

### 3. ボードをPCに接続
最初に、利用可能なプラットフォームとライブラリのローカルキャッシュを更新
```bash
arduino-cli core update-index
```

USBケーブルを使ってボードをPCに接続した後、以下のコマンドを実行して認識されているかどうかを確認できる
```
arduino-cli board list
```


> [!NOTE]
> ボードマネージャURL：開発環境の入手先を記載したインデックスファイルのURL
> Core：Arduino APIを特定のマイコンで動作させるための実装
> プラットフォーム：Core、ボード定義、ビルド設定などをまとめたもの
> FQBN：コンパイル対象のボードを識別するための名前


### 4. ボード用のコアをインストール
ボードの動作にサードパーティ製のコアパッケージが必要な場合は、Arduino CLI 設定ファイルに、追加のパッケージインデックスへの ボードマネージャURL をリストすることができます。
ボードマネージャURLは、Arduino CLIに対して、ボード用の開発環境をどこから入手できるか教えるためのURLです。
URLの先には、利用可能なCoreのバージョンや、必要なコンパイラ、ダウンロード先などを記載した**パッケージインデックス（JSONファイル）**があります。

ボード「Seeed Wio LTE Cat.1」を使うためには、次のようにボードマネージャURLを登録します。

```
arduino-cli config add \
  board_manager.additional_urls \
  http://www.seeed.co.jp/package_SeeedJP_index.json
```

パッケージインデックスをダウンロード・更新
```
arduino-cli core update-index
```

インストールできるCoreを検索。
```
arduino-cli core search SeeedJP
```

該当するCore IDを確認してインストールします。SeeedJP STM32 Boardsをインストールするので以下のようにします。
実行すると、対応する複数のボードを開発できる環境がインストールされます。
```
arduino-cli core install SeeedJP:stm32
```

実際のFQBNは次のコマンドで確認します。
```
arduino-cli board listall "Wio LTE"
```

### 4. 新しいスケッチの作成
新しいスケッチを作成するには、現在のディレクトリで、次のコマンドを実行する。
```
arduino-cli sketch new MyFirstSketch
```
以下のような構成が生成されます。
```
MyProject/
└── MyProject.ino
```