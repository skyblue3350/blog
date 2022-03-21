+++
author = "すかい"
title = "LPST-14900をArduinoに接続する その2"
date = "2018-10-03"
description = "LPST-14900をArduinoに接続する その2"
tags = [
    "Arduino",
]
+++

## はじめに

前回失敗したLPST-14900の値をArduinoで取得するのに再チャレンジした．
結果としては成功しました．
前回記事でも参考したレポジトリのピン配置と実装をほぼそのまま借りた形になっています．

検証はArduino Unoでやっているのでゲームパッドとして認識させられませんが前回の記事とか見て雰囲気で書けば使えます．
手持ちのProMicroが使用中なので新しいのが来たら…

## ピン配置

ピンの位置はシフターのコネクタ側から見た時のピン番号です．
ただの自爆ですが，何も考えずにD-subのコネクタ側から見てピンを挿して混乱しました．

この記事のスクリプトではアナログピンをデジタルピンとして使えるので全部アナログピンを利用していますが実際に必要となるのはX/Y軸のピンだけです．
この記事では以下のように配置しています．

- D-sub 1 -> A2 （デジタル入出力ピンとして使用）
- D-sub 2 -> A3 （デジタル入出力ピンとして使用）
- D-sub 3 -> A4 （デジタル入出力ピンとして使用）
- D-sub 4 -> A1 （アナログ入力ピンとして使用）
- D-sub 6 -> GND
- D-sub 8 -> A0 （アナログ入力ピンとして使用）
- D-sub 9 -> VCC

## プログラム

以下のような感じになりました．
必要な情報を設定すると出力されます．

<script src="https://gist.github.com/skyblue3350/bec89897160777722c5a208ffd0c86f5.js"></script>

手持ちのLPST-14900で問題なく全部取れました．
次はG25のペダルと組み合わせて1個のProMicroで完結するようにします．

## 参考記事

- [DIY G25 shifter interface with H-pattern, sequential and handbrake modes](https://www.isrtv.com/forums/topic/13189-diy-g25-shifter-interface-with-h-pattern-sequential-and-handbrake-modes/)
- [functionreturnfunction/G27_Pedals_and_Shifter](https://github.com/functionreturnfunction/G27_Pedals_and_Shifter)
