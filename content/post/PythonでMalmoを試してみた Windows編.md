+++
author = "すかい"
title = "PythonでMalmoを試してみた Windows編"
date = "2016-07-15"
description = "PythonでMalmoを試してみた Windows編"
tags = [
    "Malmo",
    "Python",
]
+++

## Malmoとは

マインクラフトで人工知能を育てる「Project Malmo」、MicrosoftがGitHubで一般公開
Microsoftが3月に「Project AIX」としてプライベートプレビューを開始した「Minecraft（マインクラフト）」で人工知能（AI）を訓練するプロジェクト「Project Malmo」を一般公開した。GitHubからmodとコードをダウンロードできる。
[ITMediaニュース](http://www.itmedia.co.jp/news/articles/1607/10/news022.html)

というわけでこれをチュートリアルのテストコードの動作確認まで試します

## 環境

- Win7 64bit
- Python2.7 32bit64bit環境が必要でした
  マインクラフトは普通に購入したのインストールしてあったけど不要っぽい？

## 環境構築

基本的には公式のドキュメント通り
https://github.com/Microsoft/malmo

### 各プラットフォーム向けの実行環境をダウンロードする

下記から各プラットフォーム向けのものをダウンロードする
https://github.com/Microsoft/malmo/releases

各プラットフォームで必要な環境の構築
Windowsの場合は下記を参考に構築する
https://github.com/Microsoft/malmo/blob/master/doc/install_windows.md

#### 7-zip

インストールするだけ

#### FFMPEG

公式から64bit版のexeをダウンロードしてからC:\FFMEPG\ffmpeg.exeとか適当なところに配置してパスを通す

```
> ffmpeg -version
'ffmpeg' は、内部コマンドまたは外部コマンド、
操作可能なプログラムまたはバッチ ファイルとして認識されていません。
```

パスが通ってないとこうなる
コマンド・プロンプト起動時に読み込まれるので環境変数の編集前に開いてたなら開き直すこと

```
> ffmpeg -version
ffmpeg version N-80129-ga1953d4 Copyright (c) 2000-2016 the FFmpeg developers
built with gcc 5.3.0 (GCC)
...
```

って感じなら問題なし

#### CodeSynthesis

http://www.codesynthesis.com/products/xsd/download.xhtml
ここからxsd-4.0.msiをダウンロード・インストール
インストール時のオプションは特に変更しなかった
パスを通すってオプションだけチェック入ってるか確認しておくこと

#### Python

公式から落としてパスを通すだけ
※64bit版をインストールして下さい

#### その他メモ

```
ImportError: DLL load failed: %1 は有効なWin32 アプリケーションではありません
```

32bit版のPythonから起動しているとimport MalmoPythonの時点で上記エラーが出ます

```
ImportError: DLL load failed: デバイスの準備ができていません。
```

64bit版のPythonから起動して上記エラーが出る場合はPCを再起動したら直りました

#### JDK

Javaの公式からJDKをダウンロード・インストール

```
C:\Program Files\Java\jdk1.8.0_91\bin\
```

辺りにあると思うのでパスを通す
ついでにJAVA_HOMEという環境変数を作って変数値を

```
C:\Program Files\Java\jdk1.8.0_91
```

にする

#### Microsoft Visual Studio 2013 redistributable

下記からインストール
https://www.microsoft.com/en-us/download/details.aspx?id=40784

環境構築終わり

## 実行

### Minecraftの起動

まずMalmo用のマインクラフトを起動する
一番最初にダウンロードしたzipを展開して

```
> cd Malmo-0.14.0-Windows-64bit/Minecraft
> launchClient.bat
ログがズラー
```

で最後に起動すればおっけー
初回は結構時間かかります

Microsoft Visual Studio 2013 redistributable入れ忘れててビルドエラーでコケた

### サンプルスクリプトの実行

もう1個コマンド・プロンプトを開いて

```
> cd Malmo-0.14.0-Windows-64bit/Python_Examples
> python run_mission.py
```

クライアント側で実行されてたら成功
あとはチュートリアルやれば良いと思うのである程度まとまったら記事にします

#### サンプル実行メモ

Minecraft内で視点移動が出来ない
チュートリアルのPDFのTipsに解決方法が書いてありました
Enterで視点移動の有効/無効を切り替えられます
