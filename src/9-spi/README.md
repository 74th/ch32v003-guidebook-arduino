# 9. SPI

SPIとは、Serial Peripheral Interfaceの略で、デバイス間の通信を行うためのプロトコルです。
SPIは、マスターとスレーブデバイス間でデータを同期的に転送するために使用されます。

FLASHなど、I2Cよりも高速な通信が必要なデバイスに使用されます。

CS（Chip Select）信号を使用して、複数のスレーブデバイスを制御できます。
CH32V003では、PC1をCS信号として使用しますが、複数のデバイスを接続する場合には、デバイスごとにCS信号を用意する必要があります。
その場合、GPIOとしてHigh/Low出力を設定し、対象デバイスのCS信号を制御する必要があります。

本章では、SPIデバイスとして、温度センサADT7310をサンプルに、制御の実装方法を紹介します。

## 9.1. CH32V003のSPI

CH32V003にはSPIが1つ搭載されています。
SPIは、フルデュプレックス通信とデータの送受信を同時に行えます。

リマップで使えるピンは以下の通りです。
MISO、MOSIを入れ替える用途くらいにしか使えません。

| マッピング(SPI1RM) | NSS | SCK | MISO | MOSI |
| ------------------ | --- | --- | ---- | ---- |
| Default            | PC1 | PC5 | PC7  | PC6  |
| Remapping          | PC0 | PC5 | PC7  | PC6  |

あまり利用しないと思われるため、本書ではマッピング方法の紹介は省略します。

また、SPIではPC5、PC6、PC7のピンを使用しますが、これらのピンはSOP-8パッケージのCH32V003J4M6には割り当てられていません。
したがって、CH32V003J4M6ではSPIを使用できません。

## 9.2. サンプル回路

温度センサADT7310とCH32V003の接続を回路図にすると以下となります。

<figure class="wide">
<img src="./img/spi.svg" style="background-color: white; padding:10px;" width="60%"/>
<figcaption>サンプル回路</figcaption>
</figure>

## 9.3. Arduino

Arduinoでの実装を解説します。
サンプルコードは以下のリポジトリにあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/spi_master-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/spi_master-arduino_core_ch32)

ArduinoではSPIオブジェクトからSPIを使えます。
SPIは全二重通信であり、書き込みと読み込みを同時に行えます。
次のように、SPI.beginTransaction()で通信を開始し、SPI.transfer()の引数と戻り値でデータを送受信を実装します。

```c
#include <SPI.h>

void loop() {
  uint16_t raw;
  int32_t raw_int;

  // 連続読み取りモードの開始
  SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
  // コマンドの送信
  SPI.transfer(0x54);
  SPI.endTransaction();

  delay(500);

  // 2バイト読み取り
  SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
  uint8_t b1 = SPI.transfer(0);
  uint8_t b2 = SPI.transfer(0);
  uint16_t raw = b1 << 8 | b2;
  SPI.endTransaction();
}
```

SPISettingsは、SPIの設定を行うためのクラスです。
dataOrderとdataModeは、使用するデバイスに合わせて変更してください。

```h
// speedMaximum: 通信速度
// dataOrder: データの順序（MSBFIRST or LSBFIRST）
// dataMode: SPIモード（SPI_MODE0, SPI_MODE1, SPI_MODE2, SPI_MODE3）
SPISettings mySetting(speedMaximum, dataOrder, dataMode)
```

データの送受信に利用した関数の仕様は次の通りです。

```h
// トランザクションの開始
void beginTransaction(SPISettings settings);
// トランザクションの終了
void endTransaction();
// データの送受信
uint8_t transfer(uint8_t data);
```
