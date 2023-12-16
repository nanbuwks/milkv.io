---
sidebar_label: 'Squeezenet 画像分類'
sidebar_position: 38
---

# Squeezenetに基づく画像分類

## 1. Docker開発環境の設定

[こちら](https://milkv.io/docs/duo/application-development/tpu/tpu-docker)を参照してください。Docker開発環境を設定した後、ここに戻って次のステップを続けてください。

## 2. Docker内で作業ディレクトリを準備する

`squeezenet1.1`作業ディレクトリを作成し、それに入ります。これは`tpu-mlir`と同じレベルのディレクトリであることに注意してください。
```
# mkdir squeezenet1.1 && cd squeezenet1.1
```

元のモデルを取得します
```
# wget https://github.com/onnx/models/raw/main/vision/classification/squeezenet/model/squeezenet1.1-7.tar.gz
```
`squeezenet1.1-7.tar.gz` を解凍します
```
# tar -zxvf squeezenet1.1-7.tar.gz
```
解凍が完了すると、現在のディレクトリに`squeezenet1.1`フォルダが生成されます。これには`squeezenet1.1.onnx`モデルファイルが含まれています。

テスト画像をコピーします：
```
# cp -rf ${TPUC_ROOT}/regression/dataset/ILSVRC2012/ .
# cp -rf ${TPUC_ROOT}/regression/image/ .
```
ここでの`${TPUC_ROOT}`は環境変数で、`tpu-mlir`ディレクトリに対応しており、Docker開発環境の前の設定で`source ./tpu-mlir/envsetup.sh`ステップでロードされます。

`MLIR`や`cvimodel`などのコンパイル済みファイルを保存するための`work`作業ディレクトリを作成し、その中に入ります
```
# mkdir work && cd work
```

## 3. ONNX Modelの変換

:::tip
Duo開発ボードはCV1800Bチップを搭載しており、**ONNX series** と **Caffe models**をサポートしています。現在、TFLiteモデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::

モデル変換の手順は以下の通りです：

- CaffeモデルをMLIRに変換
- 量子化に必要なキャリブレーションテーブルを生成
- MLIRを非対称INT8 cvimodelに量子化


### ONNXモデルをMLIRに変換

この例のモデルはRGB入力で、`mean`と`scale`はそれぞれ`123.675`,`116.28`,`103.53`と`0.0171`,`0.0175`,`0.0174`です。

ONNXモデルをMLIRモデルに変換するコマンドは以下の通りです：
```
model_transform.py \
 --model_name squeezenet1.1 \
 --model_def ../squeezenet1.1/squeezenet1.1.onnx \
 --test_input ../image/cat.jpg \
 --input_shapes [[1,3,224,224]] \
 --resize_dims 256,256 \
 --mean 123.675,116.28,103.53 \
 --scale 0.0171,0.0175,0.0174 \
 --pixel_format rgb \
 --test_result squeezenet1.1_top_outputs.npz \
 --mlir squeezenet1.1.mlir
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-squeezenet_05.png)

MLIRモデルに変換した後、`squeezenet1.1.mlir`ファイルが生成されます。これはMLIRモデルファイルです。また、`squeezenet1.1_in_f32.npz`ファイルと`squeezenet1.1_top_outputs.npz`ファイルも生成されます。これらは、後続のモデル変換の入力ファイルです。

![duo](/docs/duo/tpu/duo-tpu-squeezenet_06.png)

### MLIRをINT8モデルに変換

#### 量子化に必要なキャリブレーションテーブルを生成

INT8モデルに変換する前に、キャリブレーションテーブルを生成する必要があります。ここでは、既存のILSVRC2012からの100枚の画像を例に、キャリブレーションコマンドを実行します：
```
run_calibration.py squeezenet1.1.mlir \
 --dataset ../ILSVRC2012 \
 --input_num 100 \
 -o squeezenet1.1_cali_table
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-squeezenet_07.png)

操作が完了すると、`squeezenet1.1_cali_table`ファイルが生成されます。これは、後続のINT8モデルのコンパイルに使用されます。

![duo](/docs/duo/tpu/duo-tpu-squeezenet_08.png)

#### MLIRを非対称INT8 cvimodelに量子化

MLIRモデルをINT8モデルに変換するコマンドは以下の通りです：
```
model_deploy.py \
 --mlir squeezenet1.1.mlir \
 --quantize INT8 \
 --calibration_table squeezenet1.1_cali_table \
 --chip cv180x \
 --test_input ../image/cat.jpg \
 --test_reference squeezenet1.1_top_outputs.npz \
 --compare_all \
 --fuse_preprocess \
 --model squeezenet1.1_cv180x_int8_fuse.cvimodel
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-squeezenet_09.png)

コンパイルが完了すると、`squeezenet1.1_cv1800_int8_asym.cvimodel`ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-squeezenet_10.png)

## 4. Duo開発ボードでの検証

### Duo開発ボードの接続

前のチュートリアルに従ってDuo開発ボードとコンピューターの接続を完了し、`mobaxterm`や`Xshell`などのツールを使用してDuo開発ボードを操作するためのターミナルを開きます。

### tpu-sdkの取得

Dockerターミナルで`/workspace`ディレクトリに切り替えます
```
cd /workspace
```

tpu-sdkをダウンロードします
```
git clone https://github.com/milkv-duo/tpu-sdk.git
```

### tpu-sdkとモデルファイルをDuoにコピー


Duoボードのターミナルで、新しいディレクトリ`/mnt/tpu/`を作成します
```
# mkdir -p /mnt/tpu && cd /mnt/tpu
```

Dockerターミナルで、`tpu-sdk`とモデルファイルをDuoにコピーします
```
# scp -r /workspace/tpu-sdk root@192.168.42.1:/mnt/tpu/
# scp /workspace/squeezenet1.1/work/squeezenet1.1_cv180x_int8_fuse.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
```

### 環境変数を設定

Duoボードのターミナルで、環境変数を設定します
```
# cd /mnt/tpu/tpu-sdk
# source ./envs_tpu_sdk.sh
```

### 画像分類を実行

Duoボード上で、画像に対して画像分類を実行します

![duo](/docs/duo/tpu/duo-tpu-cat.jpg)

`squeezenet1.1_cv180x_int8_fuse.cvimodel`モデルを使用した画像分類：
```
./samples/bin/cvi_sample_classifier_fused_preprocess \
 ./squeezenet1.1_cv180x_int8_fuse.cvimodel \
 ./samples/data/cat.jpg \
 ./samples/data/synset_words.txt
```

成功した操作の例


![duo](/docs/duo/tpu/duo-tpu-squeezenet_11.png)
