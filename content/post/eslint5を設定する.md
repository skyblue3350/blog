+++
author = "すかい"
title = "eslint5を設定する"
date = "2018-07-06"
description = "eslint5を設定する"
tags = [
    "JavaScript",
]
+++

## はじめに

良い加減ESLint導入したいなと思って試したのでメモ
ES6なreact/jsx環境で試しています
当初airbnbを導入しましたが5系で非推奨な設定項目があって警告が出て気になったので一旦外しました

## 環境構築

必要なパッケージを入れます

```
$ npm install --save-dev eslint eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react 
```

## 設定

### npm scripts

npm scriptsに追加してsrcディレクトリ下にあるjsxを対象にチェックを行うようにします

```json
"scripts": {
   "test": "echo \"Error: no test specified\" && exit 1",
   "lint": "eslint --ext .jsx src"
}
```

jsに対しても行う場合は --ext jsと追加すれば良いです

### eslintrc

必要な設定・ルール等を記述します
eslint:recommendedをベースに気になるルールの変更と追加をしました

```
$ cat .eslintrc.yml
env:
  es6: true
  browser: true
parser: babel-eslint
parserOptions:
  sourceType: module
extends:
  - eslint:recommended
plugins:
  - eslint-plugin-react
rules:
  react/jsx-uses-vars: 1
  no-unused-vars:
    - error
    - args: none
  no-console:
    - warn
  quotes:
    - error
    - double
  semi:
    - error
  indent:
    - error
    - 4
    - SwitchCase: 1
  comma-spacing:
    - error
  comma-style:
    - error
  computed-property-spacing:
    - error
```

## 実行

先程登録しておいたのを呼び出しせば実行されます
設定したルールに基づいて検証が行われ結果が表示されるので適宜修正します

```
$ npm run lint
> eslint --ext .jsx src
/path/to/hoge.jsx
   1:51  error    Missing semicolon                              semi
   2:40  error    Missing semicolon                              semi
...

✖ 16 problems (8 errors, 8 warnings)
  4 errors, 0 warnings potentially fixable with the `--fix` option.
```
