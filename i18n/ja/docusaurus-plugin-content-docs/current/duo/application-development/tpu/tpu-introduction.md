---
sidebar_label: '紹介'
sidebar_position: 10
---

# 紹介

DuoのCPU CV1800Bは、CVITEK TPUを統合して、インテリジェントな検出を実現します。

TPUは、深層学習ニューラルネットワークのAI加速エンジンで、画像分類、オブジェクト検出、顔検出と認識、セグメンテーション、LSTMなどを加速することができます。TPUの主な機能は、CPUの作業をオフロードし、コンピュータビジョンと音声関連の操作を加速することです。

CV1800B TPUがサポートするモデル：

![duo](/docs/duo/tpu/duo-cv1800b-tpu-model_202307.png)

:::tip
 Duo開発ボードはCV1800Bチップを搭載しており、**ONNX series** と**Caffe models**をサポートしています。現在、TFLiteモデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::
