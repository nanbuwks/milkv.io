---
sidebar_label: 'Googlenetに基づく画像分類'
sidebar_position: 37
---

# Docker開発環境の設定

## 1. Configure Docker development environment

[こちら](https://milkv.io/docs/duo/application-development/tpu/tpu-docker)を参照してください。Docker開発環境を設定した後、ここに戻って次のステップを続けてください。

## 2. Docker内で作業ディレクトリを準備する

`googlenet` 作業ディレクトリを作成し、それに入ります。これは `tpu-mlir` と同じ階層のディレクトリであることに注意してください。
```
# mkdir googlenet && cd googlenet
```

元のモデルを取得します
```
wget https://github.com/onnx/models/raw/main/vision/classification/inception_and_googlenet/googlenet/model/googlenet-12.onnx
```

テスト画像をコピーします：
```
# cp -rf ${TPUC_ROOT}/regression/dataset/ILSVRC2012/ .
# cp -rf ${TPUC_ROOT}/regression/image/ .
```
ここでの`${TPUC_ROOT}` は環境変数で、`tpu-mlir`ディレクトリに対応しています。Docker 開発環境を設定したときに、 `source ./tpu-mlir/envsetup.sh` ステップで既にロードされています。

`MLIR`や`cvimodel`などのコンパイル済みファイルを保存するための`work`作業ディレクトリを作成し、その中に入ります
```
# mkdir work && cd work
```

## 3. ONNXモデルの変換

:::tip

Duo開発ボードはCV1800Bチップを搭載しており、**ONNX series**と**Caffe models**をサポートしています。現在、TFLiteモデルはサポートしていません。量子化データ型については、 **quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::

### ONNXモデルをMLIRに変換

この例のモデルはRGB入力で、`mean` と` scale` はそれぞれ `123.675,116.28,103.53` と `0.0171,0.0175,0.0174` です。モデル変換コマンドは以下の通りです：
```
model_transform.py \
 --model_name googlenet \
 --model_def ../googlenet-12.onnx \
 --test_input ../image/cat.jpg \
 --input_shapes [[1,3,224,224]] \
 --resize_dims 256,256 \
 --mean 123.675,116.28,103.53 \
 --scale 0.0171,0.0175,0.0174 \
 --pixel_format rgb \
 --test_result googlenet_top_outputs.npz \
 --mlir googlenet.mlir
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_05.png)

MLIR モデルに変換した後、`googlenet.mlir`ファイルが生成されます。これは MLIR モデルファイルです。また、`googlenet_in_f32.npz`ファイルと`googlenet_top_outputs.npz`ファイルも生成されます。これらは、後続のモデル変換の入力ファイルです。

![duo](/docs/duo/tpu/duo-tpu-googlenet_06.png)

### MLIRをBF16モデルに変換

MLIRモデルをBF16モデルに変換するコマンドは以下の通りです：
```
model_deploy.py \
 --mlir googlenet.mlir \
 --quantize BF16 \
 --chip cv180x \
 --test_input googlenet_in_f32.npz \
 --test_reference googlenet_top_outputs.npz \
 --model googlenet_cv180x_bf16.cvimodel
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_07.png)

操作が完了すると、`googlenet_cali_table`ファイルが生成されます。これは、後続の INT8 モデルのコンパイルに使用されます。

![duo](/docs/duo/tpu/duo-tpu-googlenet_08.png)

### MLIR～INT8モデル

#### 定量化に必要なキャリブレーションテーブルを生成します

`run_calibration.py`を実行してキャリブレーション テーブルを取得します。入力データ数は状況に応じて100～1000件程度となります。ここでは例として ILSVRC2012 の既存の 100 個の画像を使用し、キャリブレーション コマンドを実行します。
```
run_calibration.py googlenet.mlir \
 --dataset ../ILSVRC2012 \
 --input_num 100 \
 -o googlenet_cali_table
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_09.png)

操作が完了すると、`googlenet_cali_table`ファイルが生成されます。このファイルは、以降の INT8 モデルのコンパイルに使用されます。

![duo](/docs/duo/tpu/duo-tpu-googlenet_10.png)

#### MLIRを非対称INT8 cvimodelに量子化

MLIR モデルを INT8 モデルに変換するコマンドは以下の通りです：
```
model_deploy.py \
 --mlir googlenet.mlir \
 --quantize INT8 \
 --calibration_table googlenet_cali_table \
 --chip cv180x \
 --test_input ../image/cat.jpg \
 --test_reference googlenet_top_outputs.npz \
 --compare_all \
 --fuse_preprocess \
 --model googlenet_cv180x_int8_fuse.cvimodel
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_11.png)

コンパイルが完了すると、`googlenet_cv180x_int8_fuse.cvimodel`ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-googlenet_12.png)

## 4. Duo開発ボードでの検証

### Duo開発ボードの接続

前のチュートリアルに従って Duo 開発ボードとコンピューターの接続を完了し、`mobaxterm`や`Xshell`などのツールを使用して Duo 開発ボードを操作するためのターミナルを開きます。

### tpu-sdkの取得

Docker ターミナルで `/workspace` ディレクトリに切り替えます
```
cd /workspace
```

tpu-sdk をダウンロードします
```
git clone https://github.com/milkv-duo/tpu-sdk.git
```

### tpu-sdk とモデルファイルを Duo にコピーします

Duoボードのターミナルで、新しいディレクトリ `/mnt/tpu/` を作成します
```
# mkdir -p /mnt/tpu && cd /mnt/tpu
```

Docker ターミナルで、 `tpu-sdk` とモデルファイルを Duo にコピーします
```
# scp -r /workspace/tpu-sdk root@192.168.42.1:/mnt/tpu/
# scp /workspace/googlenet/work/googlenet_cv180x_bf16.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
# scp /workspace/googlenet/work/googlenet_cv180x_int8_fuse.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
```

### 環境変数を設定します

Duo ボードのターミナルで、環境変数を設定します
```
# cd /mnt/tpu/tpu-sdk
# source ./envs_tpu_sdk.sh
```

### 画像分類を実行します

Duo ボード上で、画像に対して画像分類を実行します

![duo](/docs/duo/tpu/duo-tpu-cat.jpg)

`googlenet_cv180x_bf16.cvimodel` モデルを使用した画像分類
```
./samples/bin/cvi_sample_classifier_bf16 \
 ./googlenet_cv180x_bf16.cvimodel \
 ./samples/data/cat.jpg \
 ./samples/data/synset_words.txt
```

成功した分類結果の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_13.png)

`googlenet_cv180x_int8_fuse.cvimodel` モデルを使用した画像分類
```
./samples/bin/cvi_sample_classifier_fused_preprocess \
 ./googlenet_cv180x_int8_fuse.cvimodel \
 ./samples/data/cat.jpg \
 ./samples/data/synset_words.txt
```

成功した分類結果の例

![duo](/docs/duo/tpu/duo-tpu-googlenet_14.png)
