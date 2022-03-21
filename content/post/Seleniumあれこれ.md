+++
author = "すかい"
title = "Seleniumあれこれ"
date = "2018-04-11"
description = "Seleniumあれこれ"
tags = [
    "Python",
    "Selenium"
]
+++

## はじめに

Seleniumでちょっとあれこれした時のメモ

## インストール

各種ドライバは環境に合わせて落とす

```
$ pip install selenium
$ wget https://chromedriver.storage.googleapis.com/2.37/chromedriver_win32.zip
$ unzip chromedriver_win32.zip
```

## 今回の要件

スクロールに合わせてコンテンツが読み込まれていくタイプのページで何が読み込まれていったのか知りたい

### 自動でスクロールさせる

SeleniumでJSを実行させることができるのでJSを流し込んでスクロールさせます

```py
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://example.com")
cmd = "document.querySelector(\".example_class\").scrollTo(0, 100);")
driver.execute_script(cmd)
```

### コンテンツの高さを取得する

同じくjsを実行して取得します
returnすると値をPython側でもらうことができます

```py
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://example.com")
cmd = "return document.querySelector(\".example_class\").scrollHeight;")
height = driver.execute_script(cmd)
```

### バックグラウンドでアクセスしたページを調べる

SeleniumはChromeのみPerformanceへのアクセスができるみたいなのでこれを使います
いろいろデータが取れるので各自確認していただきたいですがコンテンツの取得が完了した際に「Network.responseReceived」というmethodが呼ばれるみたいなのでこれを捕まえます

```py
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

caps = DesiredCapabilities.CHROME
caps["loggingPrefs"] = {"performance": "ALL"}
driver = webdriver.Chrome(desired_capabilities=caps)

driver.get("http://example.com")
# スクロール処理
for entry in driver.get_log("performance"):
    d = json.loads(entry["message"])
    if d["message"]["method"] == "Network.responseReceived":
        res = d["message"]["params"]["response"]
        if d["message"]["params"]["type"] == "Image":
            print("画像", end="")
        print(res["url"])
```

みたいな感じでレスポンスタイプが画像のやつだけ探したりできます
