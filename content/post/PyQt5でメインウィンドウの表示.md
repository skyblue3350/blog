+++
author = "すかい"
title = "PyQt5でメインウィンドウの表示"
date = "2016-09-08"
description = "PyQt5でメインウィンドウの表示"
tags = [
    "PyQt5",
    "Python",
]
+++

## はじめに

今更ながらにPyQt5に移行したのでメモ
PyQt4から少し変わってるのでそのままだと動かない

## インストール

pipからできようになった

```
$ pip3 install pyqt5
または
$ python3 -m pip install pyqt5
```

とかでインストール出来る

～2016年9月8日現在の話～
QtDesignerが入らないっぽいので現状Qtの公式のインストーラーからツールだけ選択して落としたQtCreatorを使うしかなさ気？
Toolだけ選択しても2GB近くあるので心証が宜しくない
QtDesignerと同じ画面を呼ぶにはファイル→ファイル/プロジェクトの新規作成→ファイルとクラスからQt→Qt Designer フォームを選択でいつもの慣れ親しんだ画面が出てくる

waybackとかで見る限り5.6まではexeで配布されてたので入ってるっぽいですが5.7（7月末）から変わった？
どこかからQtDesignerだけ落とすかと思ったけど見当たらず
詳細分かる方教えて下さい
～終わり～

## ウィンドウ作成

いつも通りQtDesignerを使って作成する（今回はPyQt4の時に使ってたQtDesignerで作業した）
作るのはMainWindowの方
uiファイル読み込む派の人はサンプル探せばあるのでそちらで
pyファイルに変換するにはpyuic5を使う

```
$ where pyuic5
インストールパス/Scripts/pyuic5
$ pyuic5 hoge.ui > ui.py
```

## スクリプトを書く

パッケージ名がPyQt4→PyQt5に変わったのと
QtGuiから呼んでたものがQtWidgetsから呼ぶようになった感じ

```py
# -*- coding: utf-8 -*-

import sys
from PyQt5 import QtWidgets
from ui import Ui_MainWindow


class Application(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        QtWidgets.QMainWindow.__init__(self, parent)
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)
    myapp = Application()
    myapp.show()
    sys.exit(app.exec_())
```

とりあえずUIの表示はこんな感じで問題なく出来る
QtDesignerの件だけ微妙に困る
腑に落ちないけどQtCreator使うことにする
