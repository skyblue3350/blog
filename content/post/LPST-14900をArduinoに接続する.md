+++
author = "すかい"
title = "LPST-14900をArduinoに接続する"
date = "2017-01-20"
description = "LPST-14900をArduinoに接続する"
tags = [
    "Arduino",
]
+++

## 追記

上手く行ったので新しく記事を書き直しました

- [LPST-14900をArduinoに接続する その2](../lpst-14900%E3%82%92arduino%E3%81%AB%E6%8E%A5%E7%B6%9A%E3%81%99%E3%82%8B-%E3%81%9D%E3%81%AE2/)

## はじめに

試行の記事なのでうまくいったら別記事でまとめます
先に結論だけ書いとくとNと1～6速まで問題なく取れます
~~Rだけ試行錯誤中~~
いけました

## 経緯

だいぶ前にDriving Force GTを購入してETSとかして遊んでるんですがシーケンシャルじゃなくてシフターが欲しくなった
かといってG25等々を新規に買うのは高いのでシフター単独でUSB接続出来るものを探した

追記　2017年9月17日
fanatec系列ならUSB アダプター買えば使えるっぽいのでそっちの方が良いかもしれない
シフター結構するけど…

シフター探し
TH8Aとかが単独で接続可能っぽかったけどいかんせん高い

> 価格： ￥ 19,380

のでもう少し安いシフターを探した
すると尼でLPST-14900がオススメに出てきてこちらは価格が5000円ちょっとで安い
ただG29にしか対応してない様子
どうにかこうにか単独で接続できないか調べると
G25やG27・G29のシフターをUSBに変換するアダプタを見つけた

- [Shifter Interface USB adapter for Logitech® G25, G27 and G29](http://www.leobodnar.com/shop/index.php?main_page=product_info&products_id=188)

> 1-9 $25.72

でありかなと思える価格 G29も対応してるっぽいのでこれでいけそうではある
けど送料込み込みだともう少し高そうなので自作できないか調べた

追記
このアダプタ購入して試しましたが無事使えました

コネクタ形状はD-subっぽいのでこれを良い感じにArduinoで読み出してLeonardoかMicroでUSBキーボードとして認識させれば実質単独でシフターとして運用出来るかもしれないと思って調べるとG25で既に動作するものを作ってる人がいる

- [Arduino based G25/27 Shifter Standalone Youtube](https://www.youtube.com/watch?v=62ETwZq3P4Q)

動画の説明文に詳細な解説付きのフォーラムのページがある

- [DIY G25 shifter interface with H-pattern, sequential and handbrake modes](http://www.isrtv.com/forums/topic/13189-diy-g25-shifter-interface-with-h-pattern-sequential-and-handbrake-modes/)

似たような感じでいけるんじゃないかと思って早速LPST-14900を購入してみた
とりあえず手持ちのArduino UNO R2でテストしてみる

## 試行錯誤の記録

### 使ったもの

- UNO R2
- D-sub 9ピン オス
- ジャンパワイヤ オス-オス 5本（中央でカットして10本にした）
- Micro 互換機（海外の安いのを買ったので発送待ち 2017年1月17日購入）
  2017年1月29日現在まだ来てないです　1ヶ月くらい掛かるらしいので作業中断

### 接続

あとで図書く
カットしたジャンパワイヤの被覆を一部剥いてD-subにはんだ付けする
ジャンパワイヤのオス側は上記のフォーラムの解説の通りに配置した
ただUNOなのでその辺だけ変えた
VCCとGNDはそのまま繋ぐ
あとは適当にA0～A5に繋いでおく

### 読み込み

#### テスト

PULL_UPしてanalogReadで値の変化を見て察する
正直グラフでも描画させれば良かったなって後悔した
温かみのある手作業で適当な値で切った
以下スケッチ

```c
void setup() {

  Serial.begin(9600);
  pinMode(A0, INPUT_PULLUP);
  pinMode(A1, INPUT_PULLUP);
  pinMode(A3, INPUT_PULLUP);
  pinMode(A2, INPUT_PULLUP);
  pinMode(A4, INPUT_PULLUP);
}

void loop() {
  Serial.print("A0:");
  Serial.println(analogRead(A0));
  Serial.print("A1:");
  Serial.println(analogRead(A1));
  Serial.print("A2:");
  Serial.println(analogRead(A2));
  Serial.print("A3:");
  Serial.println(analogRead(A3));
  Serial.print("A4:");
  Serial.println(analogRead(A4));
  Serial.println("-------");

  delay(1000);
}
```

#### シフト分け

察した結果を元に適当にif文で切る
あんま効率良くないコードなのでMicro互換機に載せる時に直す

```c
int X, Y;

void setup() {
  Serial.begin(9600);
  pinMode(A2, INPUT_PULLUP);
  pinMode(A4, INPUT_PULLUP);
}

void loop() {
  X = analogRead(A4);
  Y = analogRead(A2);
  if (143 < X){
    // 1 or 2
    if (Y < 100){
      Serial.println("1");
    }else if (Y < 138){
      Serial.println("change");
    }else{
      Serial.println("2");
    }
  }else if(123 < X){
    // 3 or 4 or N
    if (Y < 100){
      Serial.println("3");
    }else if (Y < 138){
      Serial.println("N");
    }else{
      Serial.println("4");
    }
  }else{
    // 5 or 6 or R
    if (Y < 100){
      Serial.println("5");
    }else if (Y < 138){
      Serial.println("change");
    }else{
      Serial.println("6");
    }
  }
}
```

Rの取り方が分からず　押し込みしても特に値に変化が見られない
もしかしたら使ってないピンで何か送ってる？
使わないと思ってはんだ付けしなかったので今度テストする

あと境界値が個体差ある気がするので適宜調整した方が良い
自分のシフターは4速時に2速側に押し込むと2速判定になる
場合分けで弾けば良いけど強く押し込まないと起きないので全体ができたら調整することにする

あとは互換機が届いたら実際にゲームで動作させてみる
