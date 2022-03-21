+++
author = "すかい"
title = "WordPress設置まとめ"
date = "2016-06-30"
description = "FC2のブログをやめて自鯖にWordPressを設置したのでメモ"
tags = [
    "WordPress",
]
+++

# はじめに

FC2のブログをやめて自鯖にWordPressを設置したので最初の記事はWordPress設置についてのまとめ
ApacheとMySQLだけ予めインストールしておく

## MySQLの設定
適当な名前で必要なテーブルやパスワードを作成する
今回は以下の設定での例

| 項目 | 設定値 |
| - | - |
| データベース名|	wp_hogehoge
| ユーザー名	|fuga
| パスワード	|aiueo
| ホスト名	l|ocalhost

```
$ mysql -u root -p
mysql> create database wp_hogehoge;
mysql> grant all on wp_hogehoge.* to 'fuga'@'localhost' identified by 'aiueo';
mysql> exit
Bye
```

## WordPressのダウンロード

次に公式サイトからWordPressをダウンロードしておく
ダウンロードしたものを解凍してブログを設置するディレクトリにアップロードしておく
もし実ディレクトリとURLを別にするならドキュメント読んで適当に設置する
最後にオーナーをApacheの実行ユーザーに変更する

```
$ wget URL
$ unzip hoge.zip
$ mv hoge /var/www/blog
$ sudo chown -R www-data /var/www/blog
```

## WordPressのインストール

アップロードしたURLにアクセスして初期設定を行う
先ほどMySQLに設定した内容を入力し、WordPressのユーザーを作成しておわり
もし以下のような画像が出たら自分でwp-config.phpに書き込む
権限がない　とか怒られたら自分でwp-config.phpを作って書き込む
とりあえずこれだけでインストールは終わり

## 追加でやること

### ログイン画面へのアクセス制限

記事を書くのは自宅でしかしないのでローカルアクセスでのみ管理画面へアクセス出来る用に設定する
予めApache側で.htaccessを有効になるように設定しておく
ログイン画面がwp-login.php、ログイン後の画面がwp-adminディレクトリなのでこの2つにアクセス制限をかける
アップロードしたディレクトリに.htaccessを作成し以下を入力してローカルからのみに制限

`.htaccess` に以下を記載する

```
<files wp-login.php>
order deny,allow
deny from all
allow from 192.168.1.
</files>
```

次にwp-adminのアクセス制限
/wp-admin/.htaccess

```
order deny,allow
deny from all
allow from 192.168.1.
```

configファイルへのアクセスを制限する
configファイルは外から見える必要性皆無なのでアクセスを制限する
先ほどの.htaccessに追加する

```
<files wp-config.php>
order allow,deny
deny from all
</files>
```

あと編集する必要がない時は

```
$ sudo chown 444 wp-config.php
```

にしておいた

### テーマ変更

デフォルトのテーマが寂しかったので変更した
管理画面に入って外観→テーマ→新しいテーマを追加から適当に気に入ったものを選択してインストールして適用

### パーマリンクが動かない

.htaccess に書き込み権限がなくてコケてた以下を追記

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /インストールパス/
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /インストールパス/index.php [L]
</IfModule>
```

### メタ情報にwordpressのURL出るのが気になる

/wp-includes/widgets/class-wp-widget-meta.php を編集する
良くわからんので56行目付近にある部分をコメントアウトして消しておく

```
<?php
/**
* Filter the "Powered by WordPress" text in the Meta widget.
*
* @since 3.6.0
*
* @param string $title_text Default title text for the WordPress.org link.
*/
/* //追加
echo apply_filters( 'widget_meta_poweredby', sprintf( '<li><a href="%s" title="%s">%s</a></li>',
esc_url( __( 'https://wordpress.org/' ) ),
esc_attr__( 'Powered by WordPress, state-of-the-art semantic personal publishing platform.' ),
_x( 'WordPress.org', 'meta widget link text' )
) );

wp_meta();
//追加*/
?>
```
