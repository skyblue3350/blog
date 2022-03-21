+++
author = "すかい"
title = "Pythonで他プロセスのメモリを読む"
date = "2017-02-24"
description = "Pythonで他プロセスのメモリを読む"
tags = [
    "Python",
    "Windows",
]
+++

## はじめに

Windows上のPythonで他のプロセスのメモリの内容を読んで見る
使うのはWin32apiで他の言語同様調べれば良い
今回はなんとなく斑鳩のスコアを読み出してみる
プレイ時間と同時に取れたらグラフ化したりしてモチベに繋がらないかなって思ったので
予めメモリの必要な番地は適当に調べておきます
今回はうさみみで調べておきました

## 読み方

プロセスを取ってハンドルを取得したらReadProcessMemoryを呼ぶだけ
読み出すだけならこんな感じにシンプルに書ける

```py
from ctypes import windll, create_string_buffer
import time
import sys

pid = 11111

OpenProcess = windll.kernel32.OpenProcess
ReadProcessMemory = windll.kernel32.ReadProcessMemory
CloseHandle = windll.kernel32.CloseHandle
PROCESS_ALL_ACCESS = 0x1F0FFF
# ツールで見た時の番地
address = 0x09412838
size = 64
buf = create_string_buffer(size)

processHandle = OpenProcess(PROCESS_ALL_ACCESS, False, pid)
if ReadProcessMemory(processHandle, address, buf, size, None):
    print buf.raw
```

## 斑鳩のスコアを読む

今回の目的の斑鳩はうさみみで調べると0x09412838からリトルエンディアンで入ってるみたいなのでひたすら読み出して変換して表示するだけ
こんな感じでスコアが取れる

```py
# coding:utf-8
from ctypes import windll, create_string_buffer
import time
import sys
import struct
import win32gui

def getProcessId(classname):
	result = []

	def window(hwnd, a):
		# if win32gui.IsWindowVisible(hwnd):
		# 	print win32gui.GetClassName(hwnd)
		# 	print win32gui.GetWindowText(hwnd)
		if win32gui.GetClassName(hwnd) == classname:
			result.append(win32process.GetWindowThreadProcessId(hwnd)[1])

	win32gui.EnumWindows(window, None)
	return result

pid = getProcessId("Ikaruga")[0]

OpenProcess = windll.kernel32.OpenProcess
ReadProcessMemory = windll.kernel32.ReadProcessMemory
CloseHandle = windll.kernel32.CloseHandle

PROCESS_ALL_ACCESS = 0x1F0FFF
# ツールで見た時の番地
address = 0x09412838
size = 64
buf = create_string_buffer(size)

processHandle = OpenProcess(PROCESS_ALL_ACCESS, False, pid)

while True:
	if ReadProcessMemory(processHandle, address, buf, size, None):

	    sys.stdout.write("\rScore: %d" % struct.unpack("<i", buf.raw[:4]))
	    sys.stdout.flush()
	else:
	    sys.stdout.write("Failed")
	    sys.stdout.flush()
	time.sleep(0.1)

CloseHandle(processHandle)
```
