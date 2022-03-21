+++
author = "すかい"
title = "ArduinoJoystickLibraryを使う"
date = "2018-06-29"
description = "ArduinoJoystickLibraryを使う"
tags = [
    "Arduino",
]
+++

## はじめに

簡単な工作したいなと思ったのでアケコンを作ってみた時に使ったArduinoJoystickLibraryの使い方のメモ
作ったの自体は機会があればまた別に記事書きます

## 必要なもの

- ATmega32U4を搭載したArduino
  今回はPro Microの互換品を使います

## 環境構築

- [MHeironimus/ArduinoJoystickLibrary](https://github.com/MHeironimus/ArduinoJoystickLibrary)

からcloneしてJoystickディレクトリをArduino IDEのインストールディレクトリにあるlibrariesに配置します

## スケッチ

あとはコード書くだけです
サンプルは公式にあるので実際に使ったスケッチを載せておきます

デフォルトだといろいろ有効になっているので不要なものは適宜コンストラクタでfalseにして無効にしておきます
これだけのコードでゲームパッドとして認識されるので便利な世の中ですね

```c
#include <Joystick.h>

Joystick_ Joystick = Joystick_(
  0x03,                    // reportid
  JOYSTICK_TYPE_GAMEPAD,   // type
  3,                       // button count
  0,                       // hat switch count
  true,                    // x axis enable
  true,                    // y axis enable
  false,                   // z axis enable
  false,                   // right x axis enable
  false,                   // right y axis enable
  false,                   // right z axis enable
  false,                   // rudder enable
  false,                   // throttle enable
  false,                   // accelerator enable
  false,                   // brake enable
  false                    // steering enable
  );

void setup(){
  // Joystick
  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);
  pinMode(4, INPUT_PULLUP);
  pinMode(5, INPUT_PULLUP);
  
  // Switch
  pinMode(6, INPUT_PULLUP);
  pinMode(7, INPUT_PULLUP);
  pinMode(8, INPUT_PULLUP);

  // Joystick lib init
  Joystick.begin();
  Joystick.setXAxisRange(0, 2);
  Joystick.setYAxisRange(0, 2);
  Joystick.setXAxis(1);
  Joystick.setYAxis(1);
}

void loop(){
  int up = !digitalRead(2); // up
  int down = !digitalRead(3); // down
  int right = !digitalRead(4); // right
  int left = !digitalRead(5); // left

  if (up){
    Joystick.setYAxis(0);
  }else if(down){
    Joystick.setYAxis(2);
  }else{
    Joystick.setYAxis(1);
  }

  if (left){
    Joystick.setXAxis(0);
  }else if(right){
    Joystick.setXAxis(2);
  }else{
    Joystick.setXAxis(1);
  }

  Joystick.setButton(0, !digitalRead(6));
  Joystick.setButton(1, !digitalRead(7));
  Joystick.setButton(2, !digitalRead(8));
}
```

こんな感じになる

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ざっくりコード書いて動作確認した <a href="https://t.co/vEkEW4KuOy">pic.twitter.com/vEkEW4KuOy</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1012600876609908737?ref_src=twsrc%5Etfw">June 29, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
