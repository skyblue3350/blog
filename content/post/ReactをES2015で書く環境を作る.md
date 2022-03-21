+++
author = "すかい"
title = "ReactをES2015で書く環境を作る"
date = "2016-12-04"
description = "ReactをES2015で書く環境を作る"
tags = [
    "JavaScript",
    "React",
]
+++

## はじめに

フロント界隈の変遷が激しいですね良く分からないです
未だにjQueryでゴリゴリ書いてる人間なので少し勉強することにしたけど忘れそうなのでメモ
ちなみにRiotとFetchくらいしか触ったことない人間です
環境構築してチュートリアルまで流します

- [Webpack + React + ES6の最小構成を考えてみる。](http://uraway.hatenablog.com/entry/2015/12/25/Webpack_%2B_React_%2B_ES6%E3%81%AE%E6%9C%80%E5%B0%8F%E6%A7%8B%E6%88%90%E3%82%92%E8%80%83%E3%81%88%E3%81%A6%E3%81%BF%E3%82%8B%E3%80%82)

ほぼこちらの記事のまんまです

## 環境

- Windows7 64bit

## 環境構築

### Node.js

公式から落としてきてインストールする
パスを以下の2箇所に通す（インストーラーがやってくれてるかも）

- nodejsのインストール場所
  - Users/Username/AppData/Roaming/npm

### プロジェクト作成

```
$ npm init
```

適当に答えてpackage.jsonを作成する

### ライブラリを揃える

#### npm install --save

利用時に必要なライブラリを追加する

```
$ npm install --save react react-dom whatwg-fetch
```

reactとreact-domはreactを使うために入れる
whatwg-fetchはFetchAPIを対応してないブラウザでも使うためのpolyfill

#### npm install --save-dev

開発時に必要なライブラリを追加する

```
$ npm install --save-dev webpack webpack-dev-server
$ npm install --save-dev babel-loader babel-core babel-preset-react babel-preset-es2015
```

### タスク

コマンド打っても良いけど面倒なので何回もしそうなことは予め書いておく
package.jsonのscriptsの項目に以下を追記する

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "server": "webpack-dev-server --progress --colors --hot --open"
},
```

これで

```
$ npm run server
```

とかするとテスト用のサーバーが呼び出せる（こうするのが良いのかは知らないけど

## プロジェクト構成

チュートリアルをやる上でこのようなディレクトリ構成にした
componentsに下にコンポーネントを置いていく感じ

```
 ProjectDir/
  ├─ dist/
  │    └ index.html
  ├─ src/
  │    ├─ components/
  │    └─ main.jsx
  ├─ package.jspn
  └─ webpack.config.js
```

### index.html

読み込むだけなのでシンプル

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>はろー りあくと</title>
</head>
<body>
    <div id="app"></div>
    <script src="/bundle.js"></script>
</body>
</html>
```

### webpack.config.js

エントリポイント（後述）と出力とかを書いていく

```js
var path = require("path");
var webpack = require("webpack");

module.exports = {
    entry: "./src/main.jsx",
    output: { path: path.join(__dirname, "dist"), filename: "bundle.js" },
    module: {
        loaders: [{
            test: /.jsx?$/,
            loader: "babel-loader",
            exclude: /node_modules/,
            query: {
                presets: ["es2015", "react"]
            }
        }]
    }
}
```

### main.jsx

チュートリアルなぞり終わったあとの状態だけどこんな感じ

```js
import React from "react";
import ReactDOM from 'react-dom';

import CommentBox from './components/CommentBox.jsx';

ReactDOM.render(
    <CommentBox url="/dist/comments.json" pollInterval={2000} />,
    document.getElementById("app")
);
```

## チュートリアル

- [ES6版React.jsチュートリアル](http://qiita.com/nownabe/items/2d8b92d95186c3941de0)

こちらの記事をFetchAPIで書いた感じです
CommentBox.jsxのloadCommentsFromServerメソッドを以下のように変更

```js
import "whatwg-fetch";

    ...中略...

    loadCommentsFromServer() {
        fetch(this.props.url).then((response)=>{
            return response.json();
        }).then((json)=>{
            this.setState({data: json});
        }).catch((ex)=>{
            console.log(ex);
        });
    }
```

その他の変更点はCommentList.jsxでkeyを渡せって怒られたので以下の部分を変更しました

```js
    render(){
        var commentNodes = this.props.data.map((comment)=>{
            return(<Comment key={comment.author} author={comment.author}>{comment.text}</Comment>);
        });
    ...
    }
```

## 参考記事

- [Webpack + React + ES6の最小構成を考えてみる。](http://uraway.hatenablog.com/entry/2015/12/25/Webpack_%2B_React_%2B_ES6%E3%81%AE%E6%9C%80%E5%B0%8F%E6%A7%8B%E6%88%90%E3%82%92%E8%80%83%E3%81%88%E3%81%A6%E3%81%BF%E3%82%8B%E3%80%82)
- [ES6版React.jsチュートリアル](http://qiita.com/nownabe/items/2d8b92d95186c3941de0)
