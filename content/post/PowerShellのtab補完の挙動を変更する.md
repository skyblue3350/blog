+++
author = "すかい"
title = "PowerShellのtab補完の挙動を変更する"
date = "2019-01-08"
description = "PowerShellのtab補完の挙動を変更する"
tags = [
    "PowerShell",
    "Windows",
]
+++

PowerShell使うと毎回気になるのがTabで補完すると

```
PS > .\hoge.txt
```

先頭に「.\」が自動で付与される挙動が微妙に気になっていたので，手を入れてみました．

## 環境

- Windows 10 Home Edition
- PowerShell

```
PS > $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.17134.407
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.17134.407
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

## Tabに割り当てられているFunctionを探す

tabという名称の付いたFunctionを探します．

```
PS > ls function:*tab*
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        TabExpansion2
```

## ソースコードの取得と書き出し

これのソースコードを取得します．

```
PS > (Get-Command 'TabExpansion2').ScriptBlock
もしくは
PS > $function:tabexpansion2
```

で標準出力で確認出来るのでこれを適当なところにリダイレクトします．

```
PS > $function:tabexpansion2 > /path/to/tab.ps1
```

あとはこれをISEなどで自分好みに加工します．
最後に任意のprofile.ps1に先程の関数名で作成したものを上書き定義して終わりです．

```
Function TabExpansion2 {
 ...
}
```

### 実行時の問題

PowerShellはデフォルトだと同じ階層にあるプログラムを実行する際は

```
PS > .\hoge.exe
```

のようにする必要がありますが，Pathに「.」を追加することで「.\」がなくとも実行可能になります．

```
PS > $env:path ="$($env:path);."
```

追記：検索し過ぎて混乱してこれも入れてましたが，よく考えたらbashもこの挙動なのでこれに関しては要らないですね．

## 成果物

こんな感じになったのでユーザーディレクトリ/Document/WindowsPowerShell/profile.ps1として置いておけば当該ユーザーのデフォルトプロファイルとして利用できます．

<script src="https://gist.github.com/skyblue3350/d1d70505df7552758eb96ea9fd03d621.js"></script>

## 参考記事

- [PowerShell Team Blog The new TabExpansion feature…](https://blogs.msdn.microsoft.com/powershell/2006/04/26/the-new-tabexpansion-feature/)
- [Windows PowerShell SDK CommandCompletion Class](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation?view=powershellsdk-1.1.0)
- [superuser Avoid dot backslash Windows 10 Powershell](https://superuser.com/questions/1373012/avoid-dot-backslash-windows-10-powershell)
