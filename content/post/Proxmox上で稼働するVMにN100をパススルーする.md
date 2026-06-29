+++
author = "すかい"
title = "Proxmox上で稼働するVMにN100をパススルーする"
date = "2026/06/27"
description = "Proxmoxでパススルーするメモ"
tags = [
    "Ubuntu",
    "Proxmox",
]
+++

## TL;DR

- BIOSからVT-d関連の設定を有効にする
- `/etc/default/grub` で `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"` する
- 管理画面からRaw Deviceの `UHD Graphics` をVMに割り当て
- VMを再起動して `lspci` で認識されていることを確認

## はじめに

最近録画鯖を再構築したのですが、ffmpegでエンコードさせているとCPUが限界を迎えてしまうので、QSVEncCに変更します。
対象のシステムはproxmox上のVMに構築されているk8s上で稼働しているため、まずはVMまでパススルーすることを目指します。
Kernelバージョンが低い場合（6.1以下？）にドライバが利用出来ないことがあるため、古い場合は先にそちらを更新する必要があります。

## 環境

- Proxmox 8.1.3
  - 4 x Intel(R) N100 (1 Socket)
- Ubuntu 22.04.3 LTS
  - 6.8.0-124-generic

## BIOSの設定変更

BIOSの設定画面からVT-dを有効にします。
今回の中華ミニPCには設定可能な項目の中には見つからなかったのですが、サポートされているCPUだったのでひとまず先に進みました。

## Proxmoxの設定変更

後述するデバイスの割り当てを実施しようとした際に `No IOMMU detected, please activate it.See Documentation for further information.` と出てくるためgrubの設定を追加します。

`/etc/default/grub` の `GRUB_CMDLINE_LINUX_DEFAULT` に追記します

```diff
- GRUB_CMDLINE_LINUX_DEFAULT="quiet"
+ GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

grubの設定を更新します。

```
update-grub
```

一度再起動しておきます

## VMの設定変更

対象のVMをシャットダウンして、

Virtual Machine -> 対象のVM -> Hardware へ移動してAddから追加します。
Raw Deviceの中にある `UHD Graphics` を選択します。

VMを再起動しておきます。

もし、起動しない場合は以下でログを見ると原因があるかもしれません。
自分の場合は今回の変更でoom-killされるようになっていたのでVMの設定を修正しました。

```
journalctl -n 50 --no-pager | grep -i qemu
```

## VMで認識していることを確認

i915を利用していることを確認します。

```
sky@base:~$ lspci -v -s 00:10.0
00:10.0 VGA compatible controller: Intel Corporation Device 46d1 (prog-if 00 [VGA controller])
        Subsystem: Device 1e50:8023
        Physical Slot: 16
        Flags: bus master, fast devsel, latency 0, IRQ 39
        Memory at fd000000 (64-bit, non-prefetchable) [size=16M]
        Memory at e0000000 (64-bit, prefetchable) [size=256M]
        I/O ports at f040 [size=64]
        Expansion ROM at 000c0000 [disabled] [size=128K]
        Capabilities: <access denied>
        Kernel driver in use: i915
        Kernel modules: i915, xe
```

vainfoとintel-gpu-toolsをインストールします。
インストール後再起動しておきます。

```
sudo apt-get install vainfo intel-gpu-tools
```

再起動後にそれぞれコマンドが動作することを確認します。

```
vainfo | head -n 10
error: can't connect to X server!
libva info: VA-API version 1.14.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_14
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.14 (libva 2.12.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 22.3.1 ()
vainfo: Supported profile and entrypoints
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileNone                   : VAEntrypointStats
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
```

```
sudo intel_gpu_top
intel-gpu-top: 8086:46d1 @ /dev/dri/card1 -    0/   0 MHz;   0% RC6;        0 irqs/s
         ENGINES     BUSY                                                        MI_SEMA MI_WAIT
       Render/3D    0.00% |                                                    |      0%      0%
         Blitter    0.00% |                                                    |      0%      0%
           Video    0.00% |                                                    |      0%      0%
    VideoEnhance    0.00% |                                                    |      0%      0%
```

## その他

おそらく直接関係はないと思われるが、試したことのメモ書き

### i915のドライバが利用されない

Kernelが古いとドライバが利用できないため更新します。

```
sudo apt install --install-recommends linux-generic-hwe-22.04 -y
```

以下も一応実施したのですが、おそらくKernel更新して解決したと思われるのでメモ程度に

`etc/modprobe.d/i915.conf` に以下を追記

```
options i915 enable_guc=3
options i915 enable_fbc=1
```

initramfsを更新して再起動

```
sudo update-initramfs -u
sudo reboot
```

### ProxmoxでiGPUを掴まないようにする

`/etc/modprobe.d/pve-blacklist.conf` に以下を追記

```
blacklist i915
blacklist intel_agp
options kvm ignore_msrs=1
options vfio_pci disable_vga=1
```

更新して再起動

```
update-initramfs -u -k all
reboot
```

## 参考サイト

- [Intel Quick Sync Video with Kubernetes](https://blog.stonegarden.dev/articles/2024/05/intel-quick-sync-k8s/)
