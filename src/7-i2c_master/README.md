# 7. I2Cマスター

I2Cは、Inter-Integrated Circuitの略で、2本のワイヤを使用してデバイス間でデータを送受信するための通信プロトコルです。
I2Cは主にセンサーやEEPROMなどの低速デバイスとの通信に使用されます。

I2Cではマスターデバイスがスレーブデバイスを操作します。
スレーブにはアドレスを指定して制御を指示するため、複数のデバイスを並列に接続できます。

筆者は自作キーボード用のモジュールとして、I2Cスレーブデバイスを複数作ってきました。
交換可能なデバイスを作るため、I2Cは非常に便利なプロトコルです。

電子工作でよく使われるI2Cデバイスに、SSD1306のOLEDディスプレイがあります。
このSSD1306のライブラリの使い方は第15章で解説します。
本章では基本的なI2Cの実装方法を解説します。

## 7.1. CH32V003のI2C

CH32V003にはマスターおよびスレーブとして動作可能なI2Cが1つ搭載されています。

ピンのマッピングは以下の通りです。

| マッピング(I2C1REMAP) | SCL | SDA |
| --------------------- | --- | --- |
| Default(0b00)         | PC2 | PC1 |
| Remap 1(0b01)         | PD1 | PD0 |
| Remap 2(0b10, 0b11)   | PC5 | PC6 |

## 7.2. サンプル回路

I2Cのプリミティブな機能の例として、SHT31という温湿度センサーを例に取ります。
このセンサーは秋月電子通商でも販売されています。

I2CのSDA、SCLのラインにはプルアップが必要です。
この回路では1kΩの抵抗を接続しています。

接続の回路図は以下となります。

<figure class="wide">
<img src="./img/sht31.svg" style="background-color: white; padding:10px;" width="70%"/>
<figcaption>SHT31を使った回路</figcaption>
</figure>

## 7.3. Arduino

ArduinoではI2CはWire.hライブラリを使います。
サンプルコードは以下にあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/i2c_master-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/i2c_master-arduino_core_ch32)

まず、初期化はWire.begin()で行います。

```c
Wire.begin();
```

書き込みは以下の3つの関数を使います。

```c
// 送信の開始
// 第1引数: スレーブのアドレス
void Wire.beginTransmission(uint8_t address)
// 1バイトのデータ送信
void Wire.write(uint8_t data)
// 送信の終了
void Wire.endTransmission()
```

SHT31（アドレス0x44）に、温度と湿度の取得のコマンド（0x24、0x00）を送る場合は以下のようになります。

```c
Wire.beginTransmission(0x44);
Wire.write(0x24);
Wire.write(0x00);
Wire.endTransmission();
```

読み込みは以下の2つの関数を使います。

```c
// スレーブからのデータ送信のリクエスト
Wire.requestFrom(uint8_t address, uint8_t num)
// 1バイトのデータ受信
uint8_t Wire.read()
```

前述のコマンドの後、受信する処理は以下のように実装できます。

```c
Wire.requestFrom(SHT31_I2C_ADDR, 6);
for (i = 0; i < 6; i++)
{
  dac[i] = Wire.read();
}

// 最初2バイトが温度データ
int t = (dac[0] << 8) | dac[1];
// 温度データを、℃に変換（小数点切り捨て）
uint32_t temperature = (((uint32_t)(t) * 175) >> 16) - 45;
// 4、5バイトが湿度データ
int h = (dac[3] << 8) | dac[4];
// 湿度データを、%に変換（小数点切り捨て）
uint32_t humidity = ((uint32_t)(h)*100) >> 16;
```

小数点以下の精度がありますが、CPUには浮動小数点演算は機能としてないため、ソフトウェアでの演算が行われプログラムのサイズが大きくなります。
なるべく整数演算かつ、割り算がないように実装すると良いでしょう。
