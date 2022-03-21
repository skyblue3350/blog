+++
author = "すかい"
title = "gitlab-ci-multi-runnerの動作確認をする"
date = "2017-08-22"
description = "gitlab-ci-multi-runnerの動作確認をする"
tags = [
    "Gitlab",
]
+++

## はじめに

Sphinx用のGitlabタスクランナーを久しぶりに作ってたらどうもうまく動かなくて困った
原因としては他のプロジェクトのランナーが有効になってたのを見落としてただけだった
良く見ればわかるんだけどもうちょっと直感的にわかりやすく表示して欲しいなって…

## 環境

こんな感じのDockerコンテナが動いてる状況

```
FROM gitlab/gitlab-runner:v9.2.0

RUN apt-get update
RUN wget https://bootstrap.pypa.io/get-pip.py \
  && python3 get-pip.py \
  && apt-get install python3.4-venv -y \
  && apt-get clean \
  && apt-get autoremove

ENTRYPOINT ["/usr/bin/dumb-init", "/entrypoint"]
CMD ["run", "--user=gitlab-runner", "--working-directory=/home/gitlab-runner"]
```

## デバッグする

対象のコンテナで

```
# docker-compose exec gitlab-runner bash
# su - gitlab-runner
$ gitlab-ci-multi-runner --debug run
Checking for jobs... nothing                        runner=[ランナーID]
Feeding runners to channel                          builds=0
```

と監視してる状況を見れるのでGitlab上でjobを動かして見てみる
今回はこれでそもそも違うコンテナで動いてることに気付いた
蓋をあけると一瞬で片がついたけど時間かかったなぁ…

## おまけ

この後無事venv環境に切り替えて作業出来たのですが

```
$ pip install -r requirements.txt
中略
  File "/usr/lib/python3.4/encodings/ascii.py", line 26, in decode
    return codecs.ascii_decode(input, self.errors)[0]
UnicodeDecodeError: 'ascii' codec can't decode byte 0xef in position 0: ordinal not in range(128)
```

と出て死にました
当該バージョンは

```
$ python3 -m pip -V
pip 1.5.4
```

デフォだとこんなバージョンなのか…

```
$ python3 -m pip install --upgrade pip
$ python3 -m pip -V
pip 9.0.1
```

して解決
