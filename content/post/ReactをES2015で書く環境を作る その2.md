+++
author = "すかい"
title = "ReactをES2015で書く環境を作る その2"
date = "2017-06-21"
description = "ReactをES2015で書く環境を作る その2"
tags = [
    "JavaScript",
    "React",
]
+++

## はじめに

大体[その1](../react%E3%82%92es2015%E3%81%A7%E6%9B%B8%E3%81%8F%E7%92%B0%E5%A2%83%E3%82%92%E4%BD%9C%E3%82%8B/)と同じ内容
ReactRouter入れたりくらいの変更
webpackも変遷の中で今微妙とかなんとか聞くけどとりあえずこれで…

## 目標

ReactとReact RouterとMaterial UIを入れてES2015で書ける環境を作る

## 環境構築

### プロジェクト作成

```
$ npm init -yes
```

### パッケージを揃える

react-routerがv3からv4に変わる過程でreact-router-domになったので注意する

```
$ npm install --save react react-dom material-ui react-router-dom
$ npm install --save-dev webpack webpack-dev-server babel-loader babel-core babel-preset-react babel-preset-es2015
```

### webpack.config.js

とりあえずこんな感じで
minifyする際にオプションを追加する必要があるけど今のところは置いておくのでこんな感じ

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
