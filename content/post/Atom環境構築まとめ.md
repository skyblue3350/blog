+++
author = "すかい"
title = "Atom環境構築まとめ"
date = "2016-07-02"
description = "Atom環境構築まとめ"
tags = [
    "Atom",
]
+++

## はじめに

環境移行するときはユーザーディレクトリ以下の~/.atomを丸々運んだ方が楽
あとフォントは現在等幅フォントで落ち着いてるからいろいろ書き直すところある

## ダウンロードとインストール

最初はとりあえずAtomをダウンロードしてインストール。

## パッケージの導入

ファイル→環境設定（Ctrl+,）で設定を開いてインストールのタブをクリック。
以下のパッケージと必要に応じてパッケージを導入する。

- autocomplete-paths
  パスを入力する時に自動で補完してくれるプラグイン、結構便利　同階層の時は./とかで出る
- chary-tree-view
  デフォルトのツリービューはシングルクリックでファイルを展開してしまうのでダブルクリックに変更するプラグイン
- file-icons
  ツリービューのファイルアイコンをそれっぽく表示してくれる。気分的なとこで
- japanese-menu
  UIの日本語化
- minimap
  エディタに良くあるミニマップを表示するプラグイン
- pigments
  CSSとか書いてる時の色を実際に表示してくれるプラグイン
- terminal-plus
  エディタ内にコンソールを表示してくれるプラグイン

## 日本語の表示の調整
日本語の表示がちゃっちぃのでフォントを変更する
ファイル→スタイルシートから編集

```css
atom-workspace {
font-family: "Meiryo";
}

atom-text-editor {
font-family: "Meiryo";
}

.tooltip {
font-family: "Meiryo";
}

.markdown-preview {
font-family: "Meiryo";
}
```

## ショートカットキーの変更
現在の割当状況は環境設定→キーバインドから見れる。
割り当てられてないものや変更したいもののcommand部分をメモしておく。
今回はCtrl+nで新規ファイル作成をツリービューの新規ファイル作成の呼び出しに上書きする。
ツリービューでのファイルの追加はtree-view:add-fileというcommandなのでファイル→キーマップでkemap.csonを開き下記を追加する

```
'body':
'ctrl-n': 'tree-view:add-file'
```

これで呼び出せるようになってればOK
ダメな時はCtrl+.で現在どのショートカットが呼び出されているのか検証することが出来るので重複していないかどうか調べる。
unset!を割り当てると割当を取り消すことが出来るので間違えて呼び出しちゃうキーとかを封印出来る。
ダメな時はこれを利用して一度無効にしてから再度割り当てるといける。

```
'body':
'ctrl-n': 'unset!'
'ctrl-n': 'tree-view:add-file'
```

## その他設定
ファイル→環境構築→設定から設定を適宜自分好みに変更しておく。
自分の場合はTabキーを押した時にスペース4つで置換されるのが嫌なのでデフォルトでチェックの入っているソフトタブを外しておく。

これらの設定は全てWinだとユーザーディレクトリ下の.atomフォルダに入っているのでこれをコピペすれば他の環境でもすぐ同じ環境が構築出来る。
他のプラットフォームでも同様の位置にあるはず。

大したことsublimeでしてなかったのでこれからはAtom使いになろうと思った（こなみ
ただシンタックスカラーだけ気に食わない
テーマはAtom Dark Slimがsublimeからの移行組はオススメ
