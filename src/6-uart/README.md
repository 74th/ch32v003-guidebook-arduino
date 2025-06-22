# 6. UART

UARTとは、2つのデバイス間でシリアル通信を行うためのインターフェースです。
送信側と受信側で通信速度を合わせて動くため、同期のためのクロック信号は必要ありません。
そのため非常にシンプルな配線で通信できます。
また、マスターとスレーブといった親子関係はありません。

MCUとしては同期通信もできるとして、データシートにはUSARTと書かれていますが、本サイトではUARTで統一します。

## 6.1. CH32V003のUART

CH32V003にはUARTが1つ搭載されています。

以下のポートに接続されています。
AFIOのリマップを使うことで、標準のポート以外でも接続できます。
ただし、ArduinoではAFIOのリマップの機能の動作を確認できませんでした。

| UART1マッピング(USART1_RM) | TX  | RX  | CK  | CTS | RTS |
| -------------------------- | --- | --- | --- | --- | --- |
| Default (0b00)             | PD5 | PD6 | PD4 | PD3 | PC2 |

UARTによるログ出力でも説明しましたが、SOP-8パッケージのCH32V003J4M6のUART TXは、SWDIOと同じピンを利用しています。
そのため、UART TXを有効にするとSWDIOが無効になり、書き込みを受け付けなくなります。

そのときには、12.4.節で説明する「電源オン時のフラッシュ消去」を行って、一度フラッシュを消去してから、書き込みを行ってください。

## 6.2. サンプル回路

UARTで動かすツールとして、秋月電子通商でも販売されているCO2センサーMH-Z19Cを使います。
筆者の利用したセンサーは壊れているようでしたが、UARTでの通信の確認に用いました。

<figure class="wide">
<img src="./img/uart.svg" style="background-color: white; padding:10px;" width="50%"/>
<figcaption>MH-Z19Cを読み取るサンプル回路</figcaption>
</figure>

## 6.3. Arduino

UARTは、ArduinoではSerialオブジェクトを使用して簡単に利用できます。
サンプルコードは以下のリポジトリにあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/uart-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/uart-arduino_core_ch32)

まず、`Serial.begin()`で初期化として、ボーレートを指定します。

```c
Serial.begin(9600);
```

バイト列を書き込むには`Serial.write()`を使います。
この関数は、バイト値1つの送信や、複数バイトの送信、文字列の送信ができます。

```c
// 1バイトの送信
size_t Serial.write(uint8_t val)
// テキストの送信
size_t Serial.write(char *str)
// バイト列の送信
// 第1引数にバッファ、第2引数にバッファの長さを指定する
size_t Serial.write(uint8_t buf[], size_t len)
```

MH-Z19CのCO2読み取り命令を送るには、以下のように実装できます。

```c
uint8_t CMD_READ_CO2_CONNECTION[9] =⏎
  { 0xFF, 0x01, 0x86, 0x00, 0x00, 0x00, 0x00, 0x00, 0x79 };

Serial.write(CMD_READ_CO2_CONNECTION, sizeof(CMD_READ_CO2_CONNECTION));
```

受信には以下のAPIが利用できます。

```c
// 1バイトの受信
// 受信できるまでブロックする
uint8_t Serial.read()
// 複数バイトの受信
// 第1引数にバッファ、第2引数にバッファの長さを指定する
// 戻り値は受信したバイト数
// タイムアウト機能があり、受信できない場合は0を返す
size_t readBytes(uint8_t *buf, size_t len)
// 複数バイトの受信
// readBytesの機能に加え、第1引数に終了文字を指定できる
size_t readBytesUntil(uint8_t terminator, uint8_t *buf, size_t len)
```

MH-Z19CのCO2読み取り命令を受信するには、以下のように実装できます。

```c
uint8_t read_buf[9] = { 0 };
uint32_t len = Serial.readBytes(read_buf, sizeof(read_buf));
// MH-Z19Cの応答からデータの抽出
uint16_t co2 = read_buf[2] * 256 + read_buf[3];
```
