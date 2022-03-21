+++
author = "すかい"
title = "React RouterでBasenameを指定する"
date = "2016-12-19"
description = "React RouterでBasenameを指定する"
tags = [
    "JavaScript",
    "React",
]
+++

## はじめに

React Routerでルート以外の時に作成したアプリを置きたい時とかの話

```jsx
ReactDOM.render(
    <Router history={browserHistory}>
        <Route path="/" component={App} />
        <Route path="/pageA" component={pageA} />
        <Route path="/pageB" component={pageB} />
        <Route path="/pageC" component={pageC} />
    </Router>,
    document.getElementById("app")
);
```

みたいなルーティングをした状態でルート以外に作成したアプリを配置をしたい時の話
このままだとデプロイ先のルートにしか置けないので困る
かといってフルパスでルーティングを書くわけにもいかない

## 対処法

historyライブラリを使うことで解決出来る
/hogeに配置するコードはこんな感じ

```jsx
import { createHistory, useBasename } from "history";

let history = useBasename(createHistory)({
    basename: "/hoge/"
})

ReactDOM.render(
    <Router history={history}>
        <Route path="/" component={App} />
        <Route path="/pageA" component={pageA} />
        <Route path="/pageB" component={pageB} />
        <Route path="/pageC" component={pageC} />
    </Router>,
    document.getElementById("app")
);
```

これで/hoge/pageAの用なルーティングを実現することが出来る

## 参考記事

- [mjackson/history Basename Support](https://github.com/mjackson/history/blob/v2.x/docs/BasenameSupport.md)
