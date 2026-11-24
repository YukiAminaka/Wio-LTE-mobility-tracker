# Wio LTE mobility tracker

Wio LTE JP Version の開発環境を Arduino CLI で準備し、スケッチの書き込みと LTE 通信を確認する手順です。コマンド例は Linux（bash / zsh）向けです。

## 1. 用意するもの

| 用途 | 必要なもの |
| --- | --- |
| 開発・書き込み | Wio LTE JP Version、PC、データ通信対応 microUSB ケーブル |
| LTE 通信 | 付属の LTE アンテナ 2 本、登録済みで利用可能な SORACOM IoT SIM（nano サイズ） |
| データの確認 | SORACOM アカウント |

SORACOM の公式セットアップに合わせ、通信確認は **plan-D** を前提にします。他の SIM を使う場合は、対応回線と APN を別途確認してください。[SORACOM セットアップガイド](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/)

SIM やアンテナは電源を切った状態で取り付けます。LTE 通信前に「LTE」と記載されたコネクタへアンテナ 2 本を接続してください。電源不足やケーブルの電圧降下は再起動の原因になるため、付属品など品質のよいケーブルを使います。AC 電源で動かす場合は 2 A 以上を供給できる USB 電源アダプターを使用します。[本体の準備](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/setup/)

## 2. Arduino CLI をインストールする

Arduino IDE 2.x をインストールしていれば、arduino-cli もその中に入っています。
すでに `arduino-cli version` が実行できる場合はインストール不要です。未導入の場合は、公式インストーラーでユーザー用ディレクトリに配置します。

```bash
mkdir -p "$HOME/.local/bin"
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh \
  | BINDIR="$HOME/.local/bin" sh
```

PATH環境変数に追加
```
export PATH="$HOME/.local/bin:$PATH"
```

バージョンの確認
```
arduino-cli version
```

初回だけ設定ファイルを作成します。既存の設定がある場合は作成し直す必要はありません。

```bash
arduino-cli config init
```
↓vscodeを使う場合Arduino CLI拡張機能を入れると便利

https://github.com/tanakamasayuki/vscode-arduino-cli-wrapper

## 3. JP 版用のボード定義とライブラリを入れる

Wio LTE JP Version では次の組み合わせを使います。[SeeedJP のライブラリ案内](https://github.com/SeeedJP/WioLTEforArduino)

| 項目 | 値 |
| --- | --- |
| ボードマネージャ URL | `http://www.seeed.co.jp/package_SeeedJP_index.json` |
| ボードパッケージ | SeeedJP STM32 Boards |
| Core ID | `SeeedJP:stm32` |
| ボード名 | Seeed Wio LTE Cat.1 |
| FQBN | `SeeedJP:stm32:WioLTE` |
| ライブラリ | Wio LTE for Arduino |

ボードマネージャ URL を登録し、ボード定義をインストールします。

```bash
# ボードマネージャURLを登録
arduino-cli config add board_manager.additional_urls \
  http://www.seeed.co.jp/package_SeeedJP_index.json

# パッケージインデックスをダウンロード・更新
arduino-cli core update-index

# Coreと依存ツールをインストール
arduino-cli core install SeeedJP:stm32@1.9
```

ライブラリ「Wio LTE for Arduino」が必要なのでインストールします。

```bash
# ライブラリのインデックスをアップデート
arduino-cli lib update-index
# "wio lte"でインストールするライブラリ名を検索
arduino-cli lib search "wio lte"
# ライブラリをインストール
arduino-cli lib install "Wio LTE for Arduino@2.13.0"
```

インストール結果とボード識別子を確認します。ボードを PC に接続していなくても実行できます。

```bash
# インストール済みのCoreを確認
arduino-cli core list
# インストールされているライブラリの一覧を表示
arduino-cli lib list
# インストールされているボードと、それに対応するFQBNを一覧表示
arduino-cli board listall "Wio LTE"
```

`Seeed Wio LTE Cat.1` の FQBN が `SeeedJP:stm32:WioLTE` であることを確認します。検索結果に表示される `WioLTEM1BG96` は別の機種です。

| 用語 | 意味 |
| --- | --- |
| ボードマネージャ URL | パッケージのバージョンや配布先が記載された JSON の URL |
| Core | マイコン向けの Arduino API の実装 |
| ボードパッケージ | Core、ボード定義、ビルド設定などをまとめたもの |
| FQBN | コンパイル・書き込み対象のボードを指定する識別子 |

## 4. USB 接続と動作モードを確認する

Wio LTE は、書き込みと実行でモードを切り替えます。[公式のモード切り替え手順](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/check-operation-mode/)

| モード | 操作（USB 接続済みの場合） | 用途 |
| --- | --- | --- |
| DFU モード | BOOT を押したまま RST を押し、RST → BOOT の順に離す | スケッチの書き込み |
| 通常モード | BOOT を押さずに RST を押して離す | スケッチの実行・シリアル出力 |

BOOT を押しながら USB を接続する方法でも DFU モードになります。

通常モードでは次のコマンドでPCに接続しているボードとシリアルポートを一覧表示します。

```bash
arduino-cli board list
```

Linux では `/dev/ttyACM0` などが表示されます。ボード名が `Unknown` でも、コンパイル対象は FQBN で指定できます。

DFU モードではシリアルポートが消えることがあります。`board list` に表示されないことだけで接続失敗と判断せず、Linux では USB デバイスを確認します（`lsusb` は `usbutils` パッケージに含まれます）。

```bash
lsusb -d 0483:df11
```

`0483:df11` はこのボードの書き込み設定で指定されている STM32 DFU デバイスの ID です。

### Linux でアクセス権限エラーが出る場合

DFU の USB アクセス権限と通常モードのシリアルポート権限は別です。以下は systemd / udev を使うデスクトップ Linux 向けの設定例です。

DFU デバイスを開けない場合は、`/etc/udev/rules.d/70-wio-lte.rules` を管理者権限で作り、次の 1 行を保存します。

```udev
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=="df11", TAG+="uaccess"
```

```bash
sudo udevadm control --reload-rules
```

USB を抜き差しし、再度 DFU モードにします。この設定はローカルでログインしているユーザー向けです。SSH 接続やコンテナなどでは、その環境に応じた USB デバイスへの権限付与が必要です。[Linux の udev 設定について](https://support.arduino.cc/hc/en-us/articles/9005041052444-Fix-udev-rules-on-Linux)

シリアルモニターで権限エラーになる場合は、対象ポートの所有グループを確認します。

```bash
ls -l /dev/ttyACM0
```

所有グループが `dialout` の場合は次を実行し、ログアウト・ログインし直します。ポート名とグループ名は環境に合わせてください。[Linux のシリアルポート権限](https://support.arduino.cc/hc/en-us/articles/360016495679-Fix-port-access-on-Linux)

```bash
sudo usermod -aG dialout "$USER"
```

Windows / macOS でセットアップする場合は、[SORACOM の開発環境準備](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/development-environment/)を参照してください。Windows は DFU 用の WinUSB、通常モード用の Virtual COM Port の確認が必要です。macOS は libusb などの必要ソフトウェアも確認してください。

## 5. LED サンプルをコンパイルして書き込む

最初は SIM やセンサーを使わない `basic/LedSetRGB` サンプルで確認します。[公式の LED 動作確認手順](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/led/)

ライブラリに含まれるサンプルの場所を調べます。

```bash
arduino-cli lib examples "Wio LTE for Arduino"
```

以下の `LED_SKETCH` は Linux の標準的な配置例です。上のコマンドで表示された `basic/LedSetRGB` ディレクトリのパスに合わせて変更してください。指定するのは `.ino` ファイルが入ったディレクトリです。

```bash
LED_SKETCH="$HOME/Arduino/libraries/Wio_LTE_for_Arduino/examples/basic/LedSetRGB"
arduino-cli compile --fqbn SeeedJP:stm32:WioLTE "$LED_SKETCH"
```

コンパイルに成功したら、シリアルモニターを閉じ、ボードを **DFU モード** にして書き込みます。

```bash
arduino-cli upload --fqbn SeeedJP:stm32:WioLTE "$LED_SKETCH"
```

このボードの書き込みは USB DFU を使うため、`--port /dev/ttyACM0` は指定しません。`upload` 自体はコンパイルしないので、コード変更後は必ず `compile` を先に実行します。

書き込みログにエラーがないことを確認し、BOOT を押さずに RST を押します。LED の色が順番に変われば成功です。`DFU end` の表示だけでなく、実際の LED の動作まで確認してください。

## 6. SORACOM Harvest Data への送信を確認する

### SIM と Harvest Data の準備

1. SORACOM ユーザーコンソールで対象 SIM が登録済みで、利用開始できる状態であることを確認します。
2. 対象 SIM を SIM グループに所属させ、そのグループの SORACOM Harvest Data を有効にして保存します。
3. Wio LTE の電源を切り、SIM と LTE アンテナ 2 本を取り付けてから USB を接続します。

画面操作は [SORACOM の Harvest Data 送信手順](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/harvest/)を参照してください。SIM の通信や Harvest Data の利用には料金が発生します。

### 公式サンプルを書き込む

手順 5 の `lib examples` の結果から `soracom/soracom-harvest` の場所を確認し、次のパスを合わせます。

```bash
HARVEST_SKETCH="$HOME/Arduino/libraries/Wio_LTE_for_Arduino/examples/soracom/soracom-harvest"
arduino-cli compile --fqbn SeeedJP:stm32:WioLTE "$HARVEST_SKETCH"
```

ボードを DFU モードに切り替えてから書き込みます。

```bash
arduino-cli upload --fqbn SeeedJP:stm32:WioLTE "$HARVEST_SKETCH"
```

このサンプルは APN `soracom.io`、ユーザー名・パスワード `sora` で接続し、`harvest.soracom.io:8514` へ UDP で稼働秒数 `uptime` を送信します。初回はセンサー用の `SENSOR_PIN` をコメントアウトしたまま使います。[公式サンプルのソース](https://github.com/SeeedJP/WioLTEforArduino/blob/main/examples/soracom/soracom-harvest/soracom-harvest.ino)

### シリアル出力と受信データを確認する

RST を押して通常モードに戻し、ポートを調べてシリアルモニターを開きます。`/dev/ttyACM0` は実際に表示されたポート名に置き換えてください。

```bash
arduino-cli board list
arduino-cli monitor --port /dev/ttyACM0 --config baudrate=115200
```

接続後、おおむね 60 秒ごとに次のようなログが出ます。

```text
### Open.
### Send.
Send:{"uptime":559}
### Receive.
Receive:201
### Close.
```

`Receive:201` を確認し、SORACOM ユーザーコンソールで対象 SIM の Harvest Data を開いて `uptime` が保存されていることを確認します。これで書き込みから LTE 通信までの初期確認は完了です。[受信結果の確認](https://users.soracom.io/ja-jp/guides/dev-boards/wio-lte/harvest/)

モニターは `Ctrl+C` で終了できます。確認が終わったら電源を切り、継続して使わない場合は Harvest Data の設定も無効にします。

## 7. スケッチの作成とコンパイル

リポジトリ直下で新しいスケッチを作る場合は、次のようにします。

```bash
arduino-cli sketch new src/MyFirstSketch
```

```text
src/
└── MyFirstSketch/
    └── MyFirstSketch.ino
```

Arduino のスケッチはディレクトリ名と主 `.ino` ファイル名を一致させます。

```bash
arduino-cli compile --fqbn SeeedJP:stm32:WioLTE src/MyFirstSketch
```

DFU モードに切り替え、同じビルド先から書き込みます。

```bash
arduino-cli upload --fqbn SeeedJP:stm32:WioLTE src/MyFirstSketch
```

以降は「編集 → コンパイル → DFU モード → 書き込み → RST → 動作確認」を繰り返します。`/tmp` のビルド結果が削除された場合は、再度コンパイルしてください。

## 困ったとき

| 症状 | 確認すること |
| --- | --- |
| `arduino-cli: command not found` | インストール先が PATH に入っているか |
| `WioLTEforArduino.h: No such file or directory` | `Wio LTE for Arduino` ライブラリをインストールしたか |
| ボードが見つからない・FQBN が不明 | `core list` と `board listall "Wio LTE"` で `SeeedJP:stm32:WioLTE` を確認 |
| `No DFU capable USB device available` | BOOT / RST の操作、データ通信対応ケーブル、USB 認識、ドライバーを確認 |
| `Cannot open DFU device` / `LIBUSB_ERROR_ACCESS` | DFU 用の udev 権限設定を確認 |
| 書き込み終了後に動かない | BOOT を離して RST を押し、通常モードに戻す |
| シリアルポートがない・ログが出ない | 通常モードか、ポート名が変わっていないか、他のモニターが占有していないかを確認 |
| LTE 接続で `ERROR`、または再起動を繰り返す | SIM の状態・挿入方向、アンテナ、電波状況、APN、USB の給電を確認 |
| LTE 接続後も Harvest に届かない | 対象 SIM のグループで Harvest Data が有効か、送信先と応答ログを確認 |

## 確認環境

- Arduino CLI: `1.5.1`
- ボードパッケージ: `SeeedJP:stm32@1.9`
- ライブラリ: `Wio LTE for Arduino@2.13.0`
