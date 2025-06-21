# 11. NeoPixel

NeoPixelはAdafruit社のRGB LEDテープの商品名ですが、本書ではWS2812Bチップを使用したRGB LEDの総称として扱います。
このデバイスの使用方法を解説します。

## 11.1. NeoPixel

<figure class="wide">
<img src="./img/ws2812_photo.drawio.svg" width="70%"/>
<figcaption>NeoPixel (WS2812B) RGB LEDテープ</figcaption>
</figure>

NeoPixelは、RGB LEDが数珠つなぎに接続されており、1本のデータピンで制御します。

NeoPixelの制御には、0.3μ秒という非常に短いパルス幅の信号を出力する必要があります。
このような高速な信号制御は、Arduino標準の`digitalWrite()`関数では困難なようです。

そのため、レジスタを直接操作したり、DMAと組み合わせるなどのPeripheralを活用する必要があります。
SPIで信号を代用する方法も知られています。

## 11.2. Arduino

Arduino環境でも、レジスタへの直接アクセスは可能です。
レジスタ直接操作や特定のCPU命令を利用して、NeoPixelを制御する関数を作成しました。
この関数をプロジェクトにコピーして使用することで、NeoPixelを制御できます。

> [https://github.com/74th/ch32v003-book-code/tree/main/neopixel-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/neopixel-arduino_core_ch32)

```c
// NeoPixelの制御
// digital_pin: NeoPixelのデータピン
// data: NeoPixelのデータ
// led_num: NeoPixelのLEDの数
void write_neopixel(uint32_t digital_pin, uint8_t *data, uint32_t led_num)
{
  uint32_t pin_name = digitalPinToPinName(digital_pin);
  GPIO_TypeDef *gpio = get_GPIO_Port(CH_PORT(pin_name));
  uint32_t pin_mask = CH_GPIO_PIN(pin_name);
  uint32_t h = pin_mask;
  uint32_t l = pin_mask << 16;
  uint32_t data_size = led_num * 3;

  for (int i = 0; i < data_size; i++)
  {
    uint16_t c = data[i];
    for (int j = 0; j < 8; j++)
    {
      if (c & 0x1)
      {
        // 0.7us
        gpio->BSHR = h;
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        // 0.6us
        gpio->BSHR = l;
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
      }
      else
      {
        // 0.35us
        gpio->BSHR = h;
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        // 0.8us
        gpio->BSHR = l;
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
        asm("c.nop");
      }
      c = c >> 1;
    }
  }
}
```

この関数の使い方を説明します。
まず、`setup()`関数内でNeoPixelのデータピンを出力モードに設定します。

```c
void setup()
{
  pinMode(PC0, OUTPUT);
}
```

NeoPixelの色は、Green、Red、Blue（GRB）の順で各色8ビット、合計24ビットのデータで指定します。
この24ビットデータを、接続されているLEDの数だけ連続して送信します。
例えば、LEDを順番に緑、赤、青で点灯させるためのデータは、次のように準備できます。

```c
#define LED_NUM 6

// 各LEDの色データを格納する配列 (GRB)
uint8_t data[LED_NUM * 3];

for (int i = 0; i < LED_NUM; i++) {
  switch (i % 3) {
    case 0: // Green
      data[i * 3] = 0x20;
      data[i * 3 + 1] = 0x00;
      data[i * 3 + 2] = 0x00;
      break;
    case 1: // Red
      data[i * 3] = 0x00;
      data[i * 3 + 1] = 0x20;
      data[i * 3 + 2] = 0x00;
      break;
    case 2: // Blue
      data[i * 3] = 0x00;
      data[i * 3 + 1] = 0x00;
      data[i * 3 + 2] = 0x20;
      break;
  }
}
```

準備したデータ配列とピン番号、LEDの数を引数として、`write_neopixel`関数を呼び出します。

```c
write_neopixel(PC0, data, LED_NUM);
```
