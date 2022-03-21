+++
author = "すかい"
title = "sphinxでうまいこと仮想環境を見てくれない時"
date = "2018-04-25"
description = "sphinxでうまいこと仮想環境を見てくれない時"
tags = [
    "Python",
    "Sphinx",
]
+++

Sphinxで上手いこと仮想環境を見てないと

```
The 'sphinx-build' command was not found. Make sure you have Sphinx
installed, then set the SPHINXBUILD environment variable to point
to the full path of the 'sphinx-build' executable. Alternatively you
may add the Sphinx directory to PATH.

If you don't have Sphinx installed, grab it from
http://sphinx-doc.org/
```

とか出てきたりして困る
エラーの通りSPHINXBUILDにsphinx-buildのパスを入れても良いがこれだと仮想環境使ってる時困るので

Winならmake.batに

```
if "%SPHINXBUILD%" == "" (
	set SPHINXBUILD=python -m sphinx
)
```

Linux系列ならMakefileに

```
SPHINXBUILD   = python -m sphinx
```

と書いておくと仮想環境に切り替わっていればPythonのパスは仮想環境へ向いてるはずなので良い感じに動く
