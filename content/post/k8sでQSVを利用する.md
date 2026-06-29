+++
author = "すかい"
title = "k8sでQSVを利用する"
date = "2026-06-28"
description = "k8s上で稼働するKonomi-TVでQSVEncCするメモ"
tags = [
    "Ubuntu",
    "Kubernetes",
]
+++

## TL;DR

- intel-device-plugins-operatorを導入する
- .spec.containers[].resourcesの設定を行う
- PodからQSVが使えることを確認する

## はじめに

[前回](./Proxmox上で稼働するVMにN100をパススルーする.md) の続きです。
ドキュメント通り入れるだけですが作業ログとして残します。

## 環境

- k8s v1.36.2
- intel-device-plugins-operator v0.36.0

## intel-device-plugins-operatorを導入する

https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/operator/README.html#installation

を参考に入れていきます。

### NFDのインストール

NFDを導入して、動作確認をします。

```
# deploy NFD
kubectl apply -k 'https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/nfd?ref=v0.36.0'
# deploy NodeFeatureRules
kubectl apply -k 'https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/nfd/overlays/node-feature-rules?ref=v0.36.0'
```

```
kubectl get pod -n node-feature-discovery
NAME                          READY   STATUS    RESTARTS   AGE
nfd-gc-df4cc89c-285g4         1/1     Running   1          30h
nfd-master-5dc96497fb-bfxv7   1/1     Running   1          30h
nfd-worker-kk4kj              1/1     Running   1          30h
```

### cert-managerのインストール

microk8sを使っている関係でプラグインを有効化して完了

```
microk8s enable cert-manager
```

### Device Plugin Operatorのインストール

ドキュメント通りapplyするのみ

```
kubectl apply -k 'https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/operator/default?ref=v0.36.0'
```

### GPU Pluginのインストール

こちらもドキュメント通り

```
kubectl apply -f https://raw.githubusercontent.com/intel/intel-device-plugins-for-kubernetes/main/deployments/operator/samples/deviceplugin_v1_gpudeviceplugin.yaml
```

## Podからの利用

nodeからi915が利用可能なことを確認します

```
kubectl get node -o 'jsonpath={.items[*].status.allocatable}'
{"cpu":"4","ephemeral-storage":"236209612Ki","gpu.intel.com/i915":"10","gpu.intel.com/monitoring":"1","hugepages-2Mi":"0","memory":"10081868Ki","pods":"110"}
```

利用するように適宜設定します

```yaml
    resources:
      limits:
        gpu.intel.com/i915: "1"
      requests:
        gpu.intel.com/i915: "1"
```

konomi TVのdockerイメージではQSVが利用出来るようになっているので、設定を変更して適用しておきます。

```diff
general:
-   encoder: 'FFmpeg'
+   encoder: 'QSVEncC'
```

この状態で適当なチャンネルを開いて再生すると、QSVが利用されるようになっています。
前回のホストに入れた `intel_gpu_top` で実際に利用されていることを確認します。

```
sudo intel_gpu_top
intel-gpu-top: 8086:46d1 @ /dev/dri/card1 -  101/ 632 MHz;   0% RC6;      238 irqs/s

         ENGINES     BUSY                                                 MI_SEMA MI_WAIT
       Render/3D    9.12% |████▏                                        |      0%      0%
         Blitter    0.00% |                                             |      0%      0%
           Video    9.13% |████▏                                        |      0%      0%
    VideoEnhance    7.97% |███▋                                         |      0%      0%
```
