# 6. ADC

本章では、ADCの使い方を説明します。

ADCとはAnalog-Digital Converterの略で、アナログ信号をデジタル信号に変換するためのコンポーネントです。
ADCは、センサーからのアナログ信号をデジタル信号に変換するために使用されます。

## 6.1. CH32V003のADC

CH32V003には、10bit分解能のADCが8チャンネル搭載されています。
入力電圧範囲は、電源電圧のVSS(GND)からVDD(VCC)までです。

それぞれのADCは以下のポートに接続されています。

- PA2: ADC_IN0
- PA1: ADC_IN1
- PC4: ADC_IN2
- PD2: ADC_IN3
- PD3: ADC_IN4
- PD5: ADC_IN5
- PD6: ADC_IN6
- PD4: ADC_IN7

## 6.2. サンプルコードの回路

ここでは2軸ジョイスティックの読み取りを行います。
回路図に示すと以下のようになります。

<figure class="wide">
<img src="./img/adc.svg" style="background-color: white;" width="60%"/>
<figcaption>2軸ジョイスティック読み取りの回路図</figcaption>
</figure>

## 6.3. Arduino

ArduinoでのADCの実装方法を説明します。
サンプルコードは以下のリポジトリにあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/adc-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/adc-arduino_core_ch32)

Arduinoでは、ADCチャンネル番号(0-7)を指定する必要はありません。
PA1などGPIOのポート名を指定すれば、対応するADCチャンネルが自動的に選択されます。
もちろん、前述の通りADC機能を持つポートでのみ利用可能です。

まず、ピンの初期化を`pinMode()`を使って行います。

```c
pinMode(PA1, INPUT_ANALOG);
pinMode(PA2, INPUT_ANALOG);
```

次に`analogRead()`を使って、アナログ値を取得します。
引数にはチャンネル名ではなく、ポート名を指定します。

```c
// 0～1023の範囲で値を取得
uint32_t x = analogRead(PA1);
uint32_t y = analogRead(PA2);
```
