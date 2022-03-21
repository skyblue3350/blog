+++
author = "すかい"
title = "GitlabをPrometheusで監視する"
date = "2018-09-14"
description = "GitlabをPrometheusで監視する"
tags = [
    "Gitlab",
    "Prometheus",
]
+++

9.3からサポートされているGitlabのPrometheusのメトリクス公開を試してみた．
環境はsameersbn/docker-gitlabの11.2.3です．
公式イメージでもできるらしいですが未検証なので参考サイトのとこに情報だけ置いておきます．

## Gitlab

### WebUI側での設定

Webから管理者アカウントでログインして管理画面へアクセスします．
Settings->Metricsの項目にPrometheusがあるのでそこを開きます．
prometheus_multiproc_dirの環境変数が未定義との警告がありますがひとまず有効にします．
有効にした後再起動しないと利用出来ないので再起動する際に設定します．

### コンソール側での設定

prometheus_multiproc_dirとモニタリング用のエンドポイントへアクセスできるようにホワイトリストへ追加します．
今回はGitlabのDockerイメージを使用しているので

```yaml
  gitlab:
    restart: always
    image: sameersbn/gitlab:11.2.3
    ...
    environment:
    ...
    # Prometheus Config
    - prometheus_multiproc_dir=/dev/shm
    - GITLAB_MONITORING_IP_WHITELIST=192.168.xxx.xxx
```

のように必要な環境変数を設定します．
あとはコンテナに設定を適用して終わりです．

```
$ docker-compose up -d
```

これで
http://gitlab-host/-/metrics
からPrometheus用のフォーマットでメトリクスを収集することができます．

## Prometheus

監視対象としていつも通り登録すれば終わりですがいつもと違い/metricsではなく/-/metricsとなるのでそこだけリラベルする必要があります．

```yaml
- targets:
  - gitlab.local:10080
  labels:
    __metrics_path__: /-/metrics
```

## 参考サイト

- [Gitlab doc GitLab Prometheus metrics](https://docs.gitlab.com/ee/administration/monitoring/prometheus/gitlab_metrics.html)
- [Gitlab doc IP whitelist](https://docs.gitlab.com/ee/administration/monitoring/ip_whitelist.html)
- [[SOLVED] Gitlab Docker prometheus_multiproc_dir and /-/metrics](https://forum.gitlab.com/t/solved-gitlab-docker-prometheus-multiproc-dir-and-metrics/9719/3)
