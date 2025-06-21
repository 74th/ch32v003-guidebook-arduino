# 13. その他元書で扱っていること

本書Arduino抜粋版では扱いませんでしたが、元の本ではCH32V003をさらに活用すべく以下のようなトピックを扱っています。

- 公式SDKでの開発方法
  - 公式IDE MounRiverStudioの使い方
  - PlatformIOでの開発方法
- ch32fun、レジスタを直接操作する開発方法
- 各Peripheralの使い方を、公式SDKやch32fun、レジスタ操作のそれぞれの方法で解説
- Arduinoからは扱えないPeripheralの使い方
  - Timer
  - DMA
  - WatchDogTimer
  - 省エネルギーモード

CH32V003は非常にプログラムフラッシュ領域が小さいため、ch32funのようにレジスタ操作を中心とすると積み込めるプログラムの幅が広がります。

これらに興味がある方はぜひ元の書籍を購入してみてください。

> CH32V003開発ガイドブック[74TH-B018] - 74th Books & Gadgets - BOOTH
> [https://74th.booth.pm/items/6934072](https://74th.booth.pm/items/6934072)
