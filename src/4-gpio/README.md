# 4. GPIO

本章では、GPIOとして、デジタル信号の入力出力を行う方法を説明します。

ここでは、以下のようなスイッチとLEDの付いた回路を制御できるようにします。

<figure class="wide">
<img src="./img/blink_and_read.svg" style="background-color: white;" width="40%"/>
<figcaption>LEDとスイッチの回路</figcaption>
</figure>

## 4.1. Arduino

Arduinoでの利用方法を解説します。
サンプルコードは以下のリポジトリにあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-arduino_core_ch32)

まず、各GPIOの値を取得する前に、GPIOの各ピンの設定をする必要があります。
Arduinoでは`pinMode`関数を使って、GPIOの設定を行います。

例えば、入力と出力の設定は以下のように記述できます。

```c
void setup()
{
  // 出力
  pinMode(PC0, OUTPUT);

  // 入力
  // INPUT: フローティング
  // INPUT_PULLUP: プルアップ
  // INPUT_PULLDOWN: プルダウン
  pinMode(PA1, INPUT_PULLUP);
}
```

pinModeの第2引数に、種別を指定します。

- `OUTPUT`: 出力
- `INPUT`: 入力、フローティング
- `INPUT_PULLUP`: プルアップを有効にした入力
- `INPUT_PULLDOWN`: プルダウンを有効にした入力

次に値を取得したり、出力するには`digitalWrite`と`digitalRead`を使います。
たとえば、LEDの点滅をしたり、ボタンの入力を取得する実装は以下となります。

```c
void loop() {
  // ボタンが押されるとLOWになる
  bool btn = digitalRead(PA1);
  if (!btn)
  {
    delay(1000);
    return;
  }

  digitalWrite(PC0, HIGH);
  delay(500);
  digitalWrite(PC0, LOW);
  delay(500);
}
```

`digitalWrite()`の第2引数には、`HIGH`または`LOW`を指定、もしくはbool値の値を入力します。
`digitalRead()`の戻り値は、`HIGH`または`LOW`のbool値です。
