+++
author = "すかい"
title = "blackbox_exporterで複数のmoduleを実行する"
date = "2018-09-09"
description = "blackbox_exporterで複数のmoduleを実行する"
tags = [
    "Prometheus",
]
+++

blackbox_exporterでDNSの監視をしているのですが前回の記事のキャッシュサーバーのダウンを検知出来ませんでした．
使っていた設定とアラートはこんな感じです．

```yaml
  - job_name: blackbox_dns
    metrics_path: /probe
    params:
      module: [internal_dns, external_dns, cache_dns]
    static_configs:
      - targets:
        - ns.local
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__address__]
        target_label: instance
      - target_label: __address__
        replacement: blackbox_exporter:9115
```

アラート

```yaml
groups:
  - name: general
    rules:
      - alert: DNSServiceDown
        expr: probe_success{job="blackbox_dns"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "DNSService {{ $labels.instance }} down"
          description: "{{ $labels.instance }} が5分間ダウンしました"
```

paramsで指定できるmoduleは1つみたいですね．
設定時にきちんとblackbox_exporterのログを確認すれば良かったのですがきちんと見てませんでした．
blackbox_exporterは動作してるポートにアクセスするとRecent Probesから直近のログを確認することができます．
ここ見てようやく1つしかmoduleが実行されていないことに気付きました．

複数個jobを作れば解決しますが今回はラベルのリライトで同一job上でmoduleを切り替えます

```yaml
  - job_name: blackbox_dns
    metrics_path: /probe
    params:
      module: [internal_dns]
    static_configs:
      - targets:
        - ns.local|internal_dns
        - ns.local|external_dns
        - ns.local|cache_dns
    relabel_configs:
      - source_labels: [__address__]
        regex: (.*?)\|(.*)
        target_label: __param_target
        replacement: ${1}
      - source_labels: [__address__]
        regex: (.*?)\|(.*)
        target_label: __param_module
        replacement: ${2}
      - source_labels: [__address__]
        regex: (.*?)\|(.*)
        target_label: instance
        replacement: ${2}
      - target_label: __address__
        replacement: blackbox_exporter:9115
```

「|」で区切ってそれを正規表現で抽出し適宜replaceする感じですね．
ルールの方は問題ないのでそのまま前回記事の方法でわざと色々落としてみて動作確認して終了．

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">検出出来たので終わりです <a href="https://t.co/8XjCitGl0S">pic.twitter.com/8XjCitGl0S</a></p>&mdash; スカイ (@skyblue3350) <a href="https://twitter.com/skyblue3350/status/1038528349994274816?ref_src=twsrc%5Etfw">September 8, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 参考記事

- [blackbox exporter duplicated text in configs](https://groups.google.com/forum/#!topic/prometheus-developers/k7OA6-DoPzo)
