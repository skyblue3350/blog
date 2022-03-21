+++
author = "すかい"
title = "自前運用のGitlabでパスワード認証ができない"
date = "2018-04-14"
description = "自前運用のGitlabでパスワード認証ができない"
tags = [
    "Gitlab",
]
+++

表題の通りですがcloneとかする時にhttpsでユーザー認証しようとすると

```
remote: HTTP Basic: Access denied
```

と怒られてしまう

構成はNginx -> Docker Gitlabって感じでhttpsでリバースプロキシしてる環境

nginxのconfで

```
proxy_set_header X-Scheme $scheme;
```

してないといけない
何故か抜けてて認証通らなくて唸ってた

追記　2018年4月17日
余談ですがこのような構成下でアクセストークン等を利用した認証をしようとすると通らなくてかなり悩んでいたんですがドキュメントの「Using HTTPS with a load balancer」の最後に

> In case GitLab responds to any kind of POST request (login, OAUTH, changing settings etc.) with a 422 HTTP Error, consider adding this to your reverse proxy configuration:
> proxy_set_header X-Forwarded-Ssl on; (nginx format)

とあって追加でこのヘッダをつける必要がありました
以下を追記しておきます

```
proxy_set_header X-Forwarded-SSL on;
```
