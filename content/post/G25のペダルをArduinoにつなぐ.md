+++
author = "すかい"
title = "G25のペダルをArduinoにつなぐ"
date = "2018-09-30"
description = "G25のペダルをArduinoにつなぐ"
tags = [
    "Arduino",
]
+++

G25のペダルだけ入手したので独立させてPCにつなぎます．
G25のペダルにはD-sub9pinのオスがコネクタとしてついているのでD-sub9pinのメスコネクタを購入して線をArduinoまで引っ張って良しなにします．

https://github.com/functionreturnfunction/G27_Pedals_and_Shifter/

こちらにG27での利用例が一式上がっているのでこちらを参考にしました．
ペダルが5500円 ProMicroが400円しないくらい，
ジャンパワイヤとコネクタも安いので6500円くらいになりました．

## 使ったもの

- ブレッドボード x1
- Arduino Pro Micro互換機 x1
- Dsub 9pin メスコネクタ x1
- ジャンパワイヤ オス-オス x10
- 熱収縮チューブ

## ハード側

7セグは外すのが面倒でつけっぱなしなだけで特に意味はありません．

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/7wAlizGKp0">pic.twitter.com/7wAlizGKp0</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1045960498921955328?ref_src=twsrc%5Etfw">September 29, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### 事前準備

ジャンパワイヤを半分に切ってケーブルの被膜を向いておきます．
Dsub側に予備ハンダをつけてからケーブルをつけます．
使うピンは6本しかないので最低限のピンだけでも大丈夫です．下記参照
熱収縮チューブを収縮させて終わり．

### 配線

以下の表に従って配線します．

| Dsub側 pin番号 | Arduino側 pin番号 | 用途 |
| - | - | - |
| 1 | +5V(VCC) | +5V |
| 2 | A0 | アクセル |
| 3 | A1 | ブレーキ |
| 4 | A2 | クラッチ |
| 6 | GND | GND |
| 9 | GND | GND |

## ソフト側

### 事前準備

機器によって多少差があるので予めそれぞれのペダルの値の範囲を確認します．

```c
void setup(){
    Serial.begin(9600);
}

void loop(){
    Serial.println(analogRead(A0)); //アクセル
}
```

アクセルから順に値の範囲を確認します．
自分の場合は
アクセル 51-900
ブレーキ 40-890
クラッチ 40-910
くらいでした．

### コーディング

[前回の記事](../arduinojoysticklibrary%E3%82%92%E4%BD%BF%E3%81%86/)を参考にArduinoJoystickLibraryを使います．
事前に調べた値をRangeとして設定して常時値を流し続けるだけです．

```c
#include <Joystick.h>

Joystick_ Joystick = Joystick_(
  0x03,                    // reportid
  JOYSTICK_TYPE_GAMEPAD,   // type
  0,                       // button count
  0,                       // hat switch count
  false,                    // x axis enable
  false,                    // y axis enable
  false,                   // z axis enable
  false,                   // right x axis enable
  false,                   // right y axis enable
  false,                   // right z axis enable
  false,                   // rudder enable
  true,                    // throttle enable
  true,                    // accelerator enable
  true,                    // brake enable
  false                    // steering enable
);

void setup(){
  Joystick.begin();
  Joystick.setAcceleratorRange(50, 900);
  Joystick.setBrakeRange(40, 890);
  Joystick.setThrottleRange(40, 910);
}

void loop(){
  Joystick.setAccelerator(analogRead(A0))
  Joystick.setBrake(analogRead(A1))
  Joystick.setThrottle(analogRead(A2))
}
```

こんな感じで値が取れるので確認しておかしなところがないか確認します．
ゲーム側で設定値の反転があることが多いので回避できること多いですが，
いくつかゲームで設定してみて使った感じアクセルとブレーキはRange逆の方が良さそうですね．

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">頭空っぽでも一瞬でできるし最高過ぎた <a href="https://t.co/WvHkNyMn5S">pic.twitter.com/WvHkNyMn5S</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1045959329789046784?ref_src=twsrc%5Etfw">September 29, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## おまけ

雰囲気で書いたFritzingです

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">雰囲気過ぎる <a href="https://t.co/Orx52OYScC">pic.twitter.com/Orx52OYScC</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1047474317611610113?ref_src=twsrc%5Etfw">October 3, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
