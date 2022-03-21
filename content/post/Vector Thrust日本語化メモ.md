+++
author = "すかい"
title = "Vector Thrust日本語化メモ"
date = "2016-11-12"
description = "Vector Thrust日本語化メモ"
tags = [
    "Vector Thrust",
]
+++

## Vector Thrust

Steamで売ってるエースコンバットもどき
日本語化してる人がいないみたいなのでチャレンジした

## 手順

~~そのうち公開するレポジトリから落として上書きする~~
[公開した](https://github.com/skyblue3350/VT_translation)

ただフォントの定義がまだ適当だからこれだと一部の文字が小さくなる
core_font_JP.xmlのFont要素を必要な個数増やして必要な属性を修正して種類を増やした上で
Font_Lang_JP.iniに対応するFont要素名を書けば直せる
ただそれだと文字が欠けた時に修正が面倒なのでこういう構成になってる

ここまでやったは良いけどしばらく時間が取れないから続きの作業が出来ないし困った

## メモ

### ファイル構成

ゲームのインストールディレクトリ（Steam/steamapp/common/VectorThrust/）をルートとして

### Language

翻訳ファイルとゲーム側の定義とフォントの定義の紐付けをするファイルが入ってる
Lang_JP.iniが翻訳ファイル
Font_Lang_JP.iniがゲーム側の定義とフォントの定義の紐付け
```ini
[Fonts]
Default = "JP.19"
```
と書くとゲーム側のDefaultフォントの参照がcore_font_JP.xmlのFont要素のname属性の対応するものが参照される
ゲーム内で使われてるフォントは「フォント定義名」のとこで調べられる

### media/MyGUI_Media

GUI周りのファイルが入ってる
core_font_JP.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<MyGUI type="Font">
  <Font name="JP.19" source="jp.ttf" size="16" resolution="50" antialias_colour="false" space_width="4" tab_width="8" cursor_width="2" distance="5" offset_height="0">
    <Code range="33 126"/>
    <Code range="1025 1105"/>
    <Code range="8470"/>
    <!-- ぁ - ん -->
    <Code range="12353 12435"/>
    <!-- ア - ン -->
    <Code range="12449 12534"/>
    <!-- 亜 - 熙 -->
    <Code range="19968 39640"/>
    <!-- ー -->
    <Code range="12540"/>

    <Code hide="128"/>
    <Code range="161 255" />
    <Code hide="1026 1039"/>
    <Code hide="1104"/>
    <!-- Latin-1 Supplement -->
    <Code range="128"/>
    <Code range="129 160" />
    <Code range="1026 1039"/>
    <Code range="1104"/>
    <Code range="8364"/>
    <Code range="260 380" />
  </Font>
</MyGUI>
```

Font要素で1個のフォント名を定義出来るっぽい
name属性に先程のファイルで参照する際に使う名前を適宜する
source属性は参照される実際のファイル
size属性は実際の文字サイズ
Code要素はrange属性で参照する文字コードを書く
例えばぁ～んまでを参照したい場合は

```
<Code range="12353 12435"/>
```

と書く

```
$ python
>>> ord(u"ぁ"), ord(u"ん")
(12353, 12435)
```

現在は適当に宣言してある

参照したくない場合はhide属性を使う？
予めCode要素で宣言しておかないと表示しようとしても空白になってしまう
またxmlファイルはシンタックスをミスるとアプリが起動しないので編集する時は注意する

### フォント定義名

ゲーム内で使用してるフォントはcore_font.xmlに宣言されてる
ので全部に対応するFont要素を作成してあげれば良いと思うけど後回し

こんな感じのコードで探せる

```py
from xml.etree.ElementTree import parse

tree = parse("core_font.xml")
elm = tree.getroot()

for e in elm.findall(".//Font"):
    print e.get("name")
```

### 定義名メモ

全体一括

```
Default = "フォント名"
```

トップのメニューのフォント

```
Monkirta.25 = "フォント名"
```

オプションのボタンフォント

```
Monkirta.33 = "フォント名"
```

オプションのラベルフォント

```
ArialRounded.19 = "フォント名"
```

キャンペーンのラベル

```
ArialRounded.25 = "フォント名"
```

### 省略語メモ

- MM
  メインメニュー
- CS
  キャンペーン
- HM
  キャンペーンのミッション開始前？
- PM
  プレイヤープロフィール
- オプション
  - OM
    オプションのボタン
  - OD
    全般設定
  - OS
    音設定
