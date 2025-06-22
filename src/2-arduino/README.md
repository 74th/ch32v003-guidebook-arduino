# 2. Arduinoでの開発

Arduinoは、Arduino IDEとArduino APIを実装したArduinoライブラリを使った開発環境です
Arduino APIでCH32Vが開発できる環境をWCH社が提供しています。

本章ではArduino IDEでの開発方法と、各機能の使い方を説明します。
各Peripheralの章でArduino APIを使った実装方法とその注意点について解説します。

CH32V003用のArduinoライブラリには、WCH社が提供するArduino Core CH32と、arduino-wch32v003があります。
本書ではArduino Core CH32を利用する場合の説明を記述します。

## 2.1. Arduinoでの開発の準備

本書ではArduino IDE 2.xを例に説明します。
Arduino IDEのインストールについては、Webで「Arduino IDE」と検索するとArduino公式サイトが見つかり、ダウンロードできますので、それをPCにインストールしてください。

Arduino IDEを起動し、メニューバーから基本設定(Settings)を開きます。
"追加のボードマネージャのURL（Additional board manager URLs）"から横のボタンを開き、URL入力のテキストエリアを開きます。

<figure class="wide">
<img src="./img/arduinoide-settings.drawio.svg" width="70%"/>
<figcaption>Arduino基本設定</figcaption>
</figure>

ここに以下のURLを1行追加します。

**一般的なCH32V003の場合**

```
https://raw.githubusercontent.com/robinjanssens/WCH32V_board_manager_files/main/package_ch32v_index.json
```

**UIAPduinoの場合**

```
https://github.com/YuukiUmeta-UIAP/board_manager_files/raw/main/package_uiap.jp_index.json
```

ここには、公式リポジトリ [https://github.com/openwch/arduino_core_ch32](https://github.com/openwch/arduino_core_ch32) ではなく、サードパーティのリポジトリを指定しています。
これは、公式リポジトリに取り込まれたすべてのPRが2025年3月時点ではボードマネージャから利用できるパッケージにまだ反映されておらず、一部の機能が使えないためです。
UIAPduinoの場合、専用のパッケージが用意されています。
現状、サードパーティが提供する上記のパッケージを使用するようにしましょう。

次に左のアクティビティバーから、ボードマネージャータブを開きます。
タブ内の検索欄に、図の様に入力してパッケージを検索し、インストールを押します。

<figure class="wide">
<img src="./img/arduinoide-board_manager.drawio.svg" width="50%"/>
<figcaption>Board Managerからパッケージのインストール</figcaption>
</figure>

Windowsの場合、`Error: 13 INTERNAL: Cannot install tool WCH:beforeinstall@1.0.0: extracting archive: Create link`というエラーが出ることがあります。
これはインストール時に管理者権限を利用しているために発生することがあります。
このエラーが出た場合には、一度Arduino IDEを終了し、Arduino IDEのアイコンを右クリックして「管理者として実行」を選択して起動してください。
一度インストールを行った後は、再度Arduino IDEを再起動し、通常ユーザで利用できます。

パッケージがインストールされ、CH32V003のボードが選択可能になりました。
次にタイトルメニューバーから、利用する環境に合わせて、以下の通りに選択します。

- 公式Arduinoの場合: 「ツール」→「ボード」→「CH32 MCU EVT Boards」→「CH32V00x」
- UIAPduinoの場合: 「ツール」→「ボード」→「UIAPduino」→「Pro Micro CH32V003」

さらに、メニューバーの「ツール」から以下の設定もします。

- Clock Select: 48MHz Internal（内部クロックを利用します）
- Upload method:
  - 公式Arduinoの場合: WCH-SWD（WCH-LinkE経由で書き込みます）
  - UIAPduinoの場合: minichlink（UIAPduinoのUSB経由で書き込みます）

## 2.2. まずLチカを成功させる

ArduinoでのLチカをしてみましょう。
Lチカとは、LEDを点滅する簡単なコードのことですが、これでビルドと書き込みができることで、開発環境が整ったことを確認できます。
タイトルメニューから「ファイル」→「スケッチ例」→「01.Basics」→「Blink」を選択すると、サンプルコードが開きます。
このサンプルコードは`LED_BUILTIN`に定義されたピンに繋がれたLEDを点滅させる内容となっています。
CH32V003では`LED_BUILTIN`は未定義となっているため、代わりに`PC0`と置換します。
以下のように書き換えます。

```c
void setup() {
  pinMode(PC0, OUTPUT);
}

void loop() {
  digitalWrite(PC0, HIGH);
  delay(1000);
  digitalWrite(PC0, LOW);
  delay(1000);
}
```

次にWCH-LinkEとCH32V003において、以下の配線を行います。
なお、UIAPduinoの場合にはWCH-LinkEは不要です。

| CH32V003 | WCH-LinkE |
| -------- | --------- |
| VCC      | 3V3       |
| GND      | GND       |
| D1       | SWDIO     |

そして、PC0に抵抗（電流制限に利用）とLEDを直列に接続します。
回路図にすると以下のような形になります。

<figure class="wide">
<img src="./img/lchika.svg" style="background-color: white; padding:10px;" width="70%"/>
<figcaption>Lチカのテストの配線</figcaption>
</figure>

配線後に、WCH-LinkEをPCに接続しましょう。
UIAPduinoの場合には、ボード上のボタンを押しながらUSBケーブルをPCに接続します。

さて、Arduino IDEでファームウェアを書き込み、上部のツールバーアイコンの右矢印（→）マーク「書き込み」をクリックします。
このときエラーが表示されても、出力タブに「Verified OK」と表示されていれば成功です。
UIAPduinoの場合には、「Image written.」と表示されれば成功です。

<figure class="wide">
<img src="./img/arduinoide-upload.drawio.svg" width="75%"/>
<figcaption>ビルドと書き込み</figcaption>
</figure>

## 2.3. Arduinoでのファームウェアの開発

### Arduinoでのピンの名前の定義

Arduino IDE内では、ピンの名前として`PC0`や、`PD2`といったように`P<GPIO名><ピン番号>`という形で定義されています。

なお、`PC_0`という定義も存在しますが、こちらはArduinoライブラリの内部定数のため、使用しないようにしてください。

### ログ出力

CH32VシリーズにはSDI-PrintというSWIO経由でのログ出力の仕組みがあるのですが、CH32V003のArduinoでは未実装となっています。
UART経由でログ出力するしかありません。

UARTのTX（送信）は`PD5`が使われます。
`PD5`と、WCH-LinkEのRXを接続してください。

また、SOP-8パッケージのCH32V003J4M6ではUARTのデフォルトのTX（送信）ピン `PD5` がSWDIOピンの `PD1` と共用されているため、一度UARTを有効にするとSWIOが無効になりSWIO経由での書き込みができなくなります。
詳しくは[「12.6. CH32V003開発で知っておくと良いこと」の「SOP-8のCH32V003J4M6のUSART TXとSWIOのピンの共用」](../12-ch32v003/README.md#sop-8のch32v003j4m6のusart-txとswioのピンの共用)に書かれています。そちらを参照してください。

UARTでログ出力の例を示します。
print命令の詳しい使い方は、Arduinoのドキュメントを参照してください。

```c
void setup()
{
  Serial.begin(115200);
  Serial.println("start");
}

int count = 0;

void loop()
{
  Serial.print("loop ");
  Serial.println(count++);
  delay(500);
}
```

Arduino IDEでシリアルモニタを開くには、まずツールバー中のポートの選択から、WCH-LinkEのポートを選択します。
その次のツールバー右側の虫眼鏡のアイコンをクリックします。
すると下部パネル内にSerial.printで送った内容が表示されます。

<figure class="wide">
<img src="./img/arduinoide-serialmonitor.drawio.svg" width="60%"/>
<figcaption>Serial Monitor</figcaption>
</figure>

なお、UIAPduinoでは、一般的なArduinoのようにUSB経由でシリアルモニタを開く機能は現状はありません。

## 2.4. Arduinoのコードからレジスタを操作する

（本節は上級トピックのため、未掲載です。元本を参照ください。）

## 2.5. まとめ

以上、Arduino IDEでの開発方法を説明しました。
後の各Peripheralの解説では、GPIOやADCなどのArduinoのAPIの使い方について詳しく説明します。

本書ではArduino APIで各Peripheralを動かしています。
予想よりもきちんと実装されていて、動くことを確認できました。
そのため、Arduinoを使った開発は人に勧めるのに有用だと感じました。
