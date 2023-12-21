---
sidebar_label: '紹介'
sidebar_position: 10
---

# 紹介

DuoのCPU CV1800Bは、CVITEK TPU を統合して、インテリジェントな検出を実現します。

TPU は、深層学習ニューラルネットワークの AI 加速エンジンで、画像分類、オブジェクト検出、顔検出と認識、セグメンテーション、 LSTM などを加速することができます。TPU の主な機能は、CPU の作業をオフロードし、コンピュータビジョンと音声関連の操作を加速することです。

CV1800B TPUがサポートするモデル：

![duo](/docs/duo/tpu/duo-cv1800b-tpu-model_202307.png)

:::tip
 Duo 開発ボードは CV1800B チップを搭載しており、**ONNX series** と **Caffe models** をサポートしています。現在、TFLite モデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::
