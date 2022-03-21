+++
author = "すかい"
title = "rails-erdで多対多のリレーションの描画"
date = "2017-02-01"
description = "rails-erdで多対多のリレーションの描画"
tags = [
    "Ruby on Rails",
]
+++

## tl;dr

rails-erdでE-R図描いたら多対多のリレーションが表示されない
→rails-erdはhas_manyやthroughを使ったリレーションは一方向にしか表示されない
らしいので結局railroadyを使った
使った環境がだいぶ前に構築したものだからかもしれないので備忘録の側面が強い

初めてrails触ってるのでそもそも何か間違ってるのかもしれない

以下備忘録代わりのメモ

## 作業内容

例えば記事に複数のカテゴリをつけられるようなリレーションモデルを作成する

### プロジェクト生成と設定

生成と必要なライブラリの追加とか

```
$ rails new blog
$ vi Gemfile
gem "railroady"
gem "rails-erd"
を末尾に追加
$ bundle install
$ rake db:create:all
```

### DBの設定

DBの設定をする

```
$ vi config/database.yml
```

### モデルの生成

記事としてArticle　カテゴリとしてCategoryモデルを作る
そして これらをつなぐ中間モデルArticleCategoryを作成する

```
$ rails g scaffold Article title:text date:datetime body:text
$ rails g scaffold Category name:text description:text
$ rails g scaffold ArticleCategory article:references category:references
```

各モデルのリレーションを記述する

```
$ vi app/models/article.rb
class Article < ActiveRecord::Base
    has_many :ArticleCategorys
    has_many :Categorys, through: :ArticleCategorys
end

$ vi app/models/category.rb
class Category < ActiveRecord::Base
    has_many :ArticleCategorys
    has_many :Articles, through: :ArticleCategorys
end

$ vi app/models/article_category.rb
class ArticleCategory < ActiveRecord::Base
  belongs_to :Article
  belongs_to :Category
end
```

### モデルの反映

作成したモデルを反映する

```
$ rake db:migrate
```

### ER図の作成

#### rails-erd

rails-erdでのER図の生成

```
$ rake erd filetype=png
```

プロジェクトのルートディレクトにerd.pngという名前でファイルが生成される
見やすいが多対多のモデルをhas_manyやthroughを使って記述すると一方向にしか表示されないらしい

- [Indirect associations - the association should be symmetric #25](https://github.com/gemhome/rails-erd/issues/25)

![](/images/2017-02-01-001.png)

#### railroady

とりあえず一通り出してみる
docディレクトリの下に画像ファイルが生成される

```
$ rake diagram:all
```

svgで吐き出されるので適宜変換する

![](/images/2017-02-01-002.png)

## 参考記事

- [もた日記 Railsメモ(15) : rails-erdでER図を出力する](http://wonderwall.hatenablog.com/entry/2015/08/11/161010)
- [Qiita zaru/RailsアプリでER図とかクラス図を作る](http://qiita.com/zaru/items/8227686ebee9f519985b)
