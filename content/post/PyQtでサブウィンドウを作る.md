+++
author = "すかい"
title = "PyQtでサブウィンドウを作る"
date = "2016-09-15"
description = "PyQtでサブウィンドウを作る"
tags = [
    "PyQt",
    "Python",
]
+++

## はじめに

いつも検索しちゃうのでメモ
QtDesignerで作ったオリジナルなダイアログを出したい時の話
PyQt5で書いてるけど4でも大体同じ

## ウィンドウを作る

予め2つのウィンドウを作成します
メインの画面はMainWindowを使って作り
サブの画面はFormを使って作成します

## コンバート

いつも通りpyファイルを生成する

```
$ pyuic5 mainwindow.ui > ui_mainwindow.py
$ pyuic5 subwindow.ui > ui_subwindow.py
```

## コード

```py
# -*- coding: utf-8 -*-

import sys
from PyQt5 import QtWidgets


class SubWindow(QtWidgets.QDialog):
    def __init__(self, parent=None):
        QtWidgets.QDialog.__init__(self, parent)
        self.ui = Ui_SubWindow()
        self.ui.setupUi(self)

    def show(self):
        self.exec_()

class MainWindow(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        QtWidgets.QMainWindow.__init__(self, parent)
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

        self.sub = SubWindow(self)

    def openDialog(self):
        self.sub.show()

if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)
    myapp = MainWindow()
    myapp.show()
    sys.exit(app.exec_())
```

こんな感じでオリジナルなダイアログが作れる
ダイアログ側も普段通り作れば動くのでメインウィンドウな気分で作れば良い
もしダイアログ開いててもメインウィンドウが操作したいような挙動を求める場合はSubWindowクラスのshow関数を消せばダイアログを出しつつ下のメインウィンドウも操作出来るようになる
サブウィンドウからメインウィンドウへ値を渡したい時はreturnで渡せば良い
