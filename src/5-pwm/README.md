# 5. PWM

PWM（Pulse Width Modulation）は、デジタル信号を用いてアナログ信号のように振る舞わせる技術です。
主にモーター制御やLEDの明るさ調整に使用されます。

TimerにはPWMのための機能が含まれています。
例えば256カウントでループするカウンタを作ったとして、PWM機能を使うとTimerのカウントの度に設定した値と現在の値を比べ、大きければGPIOをHighにできます。
これで、比較する値をデューティー比として使い、PWMのパルス幅を調整できます。
前章のTimerを使ってPWMを実装します。

なお、1つのTimerで複数のチャンネルを利用することで、異なるパルス幅のPWM信号を同時に出力できます。

## 7.1. PWMが使えるピン

PWMはTimerを使うため、Timerのアウトプットチャンネルに対応したGPIOピンで利用できます。

Timerの各チャンネルのマッピングが以下の表のようになっています。
デフォルトではDefault列の対応となりますが、AFIOを使うことで、リマップとして別の行の組み合わせにできます。
ただし、一部のチャンネルのみリマップするような使い方はできません。

**TIM1のアウトプットチャンネルのマッピング**

| TIM1マッピング (TIM1RM) | CH1 | CH2 | CH3 | CH4 |
| ----------------------- | --- | --- | --- | --- |
| Default (0b00)          | PD2 | PA1 | PC3 | PC4 |
| Partial 1 (0b01)        | PC6 | PC7 | PC0 | PD3 |
| Partial 2 (0b10)        | PD2 | PA1 | PC3 | PC4 |
| Full (0b11)             | PC4 | PC7 | PC5 | PD4 |

**TIM2のアウトプットチャンネルのマッピング**

| TIM2マッピング (TIM2RM) | CH1 | CH2 | CH3 | CH4 |
| ----------------------- | --- | --- | --- | --- |
| Default (0b00)          | PD4 | PD3 | PC0 | PD7 |
| Partial 1 (0b01)        | PC5 | PC2 | PD2 | PC1 |
| Partial 2 (0b10)        | PC1 | PD3 | PC0 | PD7 |
| Full (0b11)             | PC1 | PC7 | PD6 | PD5 |

## 7.2. Arduino

Arduinoでの利用は簡単です。
サンプルコードは以下のリポジトリにあります。

> [https://github.com/74th/ch32v003-book-code/tree/main/pwm-arduino_core_ch32](https://github.com/74th/ch32v003-book-code/tree/main/pwm-arduino_core_ch32)

まず、利用するピンは前節のマッピングのリストを参照してください。
"0b00 (Default)"にリストアップされたピンを利用するようにしてください。
TIM1、TIM2のどちらでも利用できます。
Remapでも動作できるように実装されてはいるようなのですが、動作を確認できませんでした。

ピンの向きを設定します。

```c
pinMode(PD2, OUTPUT);
```

次に`analogWrite`関数を使ってPWM信号を出力します。
値は12bit、0-4095の範囲で指定します。

32段階で明るさをフェードイン、フェードアウトする例を示します。

```c
void loop() {
  for(int i=1;i<=32;i++){
    analogWrite(PD2, 128*i-1);
    delay(31);
  }

  for(int i=1;i<=32;i++){
    analogWrite(PD2, 128*(33-i)-1);
    delay(31);
  }
}
```
