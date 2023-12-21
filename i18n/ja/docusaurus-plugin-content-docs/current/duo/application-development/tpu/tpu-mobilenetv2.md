---
sidebar_label: 'MobileNetV2 画像分類'
sidebar_position: 35
---

# MobileNetV2に基づく画像分類

## 1. Docker開発環境の設定

[こちら](https://milkv.io/docs/duo/application-development/tpu/tpu-docker)を参照してください。Docker 開発環境を設定した後、ここに戻って次のステップを続けてください。

## 2. Docker 内で作業ディレクトリを準備する

`mobilenet_v2`作業ディレクトリを作成し、それに入ります。これは `tpu-mlir` と同じレベルのディレクトリであることに注意してください。
```
# mkdir mobilenet_v2 && cd mobilenet_v2
```

公式ウェブサイトから MobileNet モデルをダウンロードします：
```
git clone https://github.com/shicai/MobileNet-Caffe.git
```

clone した `MobileNet-Caffe` ディレクトリ内のモデルファイルと `tpu-mlir` ツールチェーンディレクトリ内の画像ファイルを現在のディレクトリに配置します。
```
# cp MobileNet-Caffe/mobilenet_v2_deploy.prototxt .
# cp MobileNet-Caffe/mobilenet_v2.caffemodel .
# cp -rf ${TPUC_ROOT}/regression/dataset/ILSVRC2012/ .
# cp -rf ${TPUC_ROOT}/regression/image/ .
```
ここでの`${TPUC_ROOT}`は環境変数で、`tpu-mlir`ディレクトリに対応しており、Docker 開発環境の前の設定で `source ./tpu-mlir/envsetup.sh` ステップでロードされます。

`MLIR`や`cvimodel` などのコンパイル済みファイルを保存するための `work` 作業ディレクトリを作成し、その中に入ります
```
# mkdir work && cd work
```

## 3. MobileNet-Caffe モデルの変換

:::tip
Duo 開発ボードは CV1800B チップを搭載しており、**ONNX series** と **Caffe models** をサポートしています。現在、TFLite モデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::

モデル変換の手順は以下の通りです：

- Caffe モデルを MLIR に変換
- 量子化に必要なキャリブレーションテーブルを生成
- MLIR を非対称 INT8 cvimodelに 量子化

### Caffe モデルを MLIR に変換

モデルの入力は画像です。モデルを変換する前に、モデルの前処理を理解する必要があります。モデルが前処理済みの npz ファイルを入力として使用している場合、前処理を考慮する必要はありません。前処理の過程は次のように表現されます（ x は入力を表します）:
$$ y = (x-mean)\times scale $$

この例のモデルは BGR 入力で、`mean`と`scale`はそれぞれ`103.94, 116.78, 123.68`と`0.017, 0.017, 0.017`です。モデル変換コマンドは以下の通りです：
```
model_transform.py \
 --model_name mobilenet_v2 \
 --model_def ../mobilenet_v2_deploy.prototxt \
 --model_data ../mobilenet_v2.caffemodel \
 --input_shapes [[1,3,224,224]] \
 --resize_dims=256,256 \
 --mean 103.94,116.78,123.68 \
 --scale 0.017,0.017,0.017 \
 --pixel_format bgr \
 --test_input ../image/cat.jpg \
 --test_result mobilenet_v2_top_outputs.npz \
 --mlir mobilenet_v2.mlir
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_05.png)

MLIR モデルに変換した後、`mobilenet_v2.mlir`ファイルが生成されます。これは MLIR モデルファイルです。また、`mobilenet_v2_in_f32.npz` ファイルと `mobilenet_v2_top_outputs.npz` ファイルも生成されます。これらは、後続のモデル変換の入力ファイルです。

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_06.png)

### MLIR を INT8 モデルに変換

#### 量子化に必要なキャリブレーションテーブルを生成

`run_calibration.py` を実行してキャリブレーションテーブルを取得します。入力データの数は状況によりますが、おおよそ 100〜1000 枚程度であるべきです。ここでは、既存の ILSVRC2012 からの 100 枚の画像を例に、キャリブレーションコマンドを実行します:

```
run_calibration.py mobilenet_v2.mlir \
 --dataset ../ILSVRC2012 \
 --input_num 100 \
 -o mobilenet_v2_cali_table
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_07.png)

操作が完了すると、`mobilenet_v2_cali_table`ファイルが生成されます。これは、後続の INT8 モデルのコンパイルに使用されます。

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_08.png)

MLIR を非対称 INT8 cvimodel に量子化

`model_deploy.py`スクリプトパラメータを使用して非対称を非対称量子化に使用し、MLIR ファイルを INT8 非対称量子化モデルに変換します：

```
model_deploy.py \
 --mlir mobilenet_v2.mlir \
 --asymmetric \
 --calibration_table mobilenet_v2_cali_table \
 --fuse_preprocess \
 --customization_format BGR_PLANAR \
 --chip cv180x \
 --quantize INT8 \
 --test_input ../image/cat.jpg \
 --model mobilenet_v2_cv1800_int8_asym.cvimodel
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_09.png)

コンパイルが完了すると、`mobilenet_v2_cv1800_int8_asym.cvimodel` ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_10.png)

## 4. Duo 開発ボードでの検証

### Duo 開発ボードの接続

前のチュートリアルに従って Duo 開発ボードとコンピューターの接続を完了し、`mobaxterm` や `Xshell` などのツールを使用して Duo 開発ボードを操作するためのターミナルを開きます。

### tpu-sdk の取得

Docker  ターミナルで `/workspace` ディレクトリに切り替えます
```
cd /workspace
```

tpu-sdk をダウンロードします
```
git clone https://github.com/milkv-duo/tpu-sdk.git
```

### tpu-sdk とモデルファイルを Duo にコピーします

Duoボードのターミナルで、新しいディレクトリ`/mnt/tpu/`を作成します
```
# mkdir -p /mnt/tpu && cd /mnt/tpu
```

Dockerターミナルで、`tpu-sdk`とモデルファイルを Duo にコピーします
```
# scp -r /workspace/tpu-sdk root@192.168.42.1:/mnt/tpu/
# scp /workspace/mobilenet_v2/work/mobilenet_v2_cv1800_int8_asym.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
```

### 環境変数を設定します

Duoボードのターミナルで、環境変数を設定します
```
# cd /mnt/tpu/tpu-sdk
# source ./envs_tpu_sdk.sh
```

### 画像分類を実行します


Duoボード上で、画像に対して画像分類を実行します

![duo](/docs/duo/tpu/duo-tpu-cat.jpg)

samples ディレクトリに入ります

```
# cd samples
```

cvimodel 情報を表示します
```
./bin/cvi_sample_model_info ../mobilenet_v2_cv1800_int8_asym.cvimodel
```

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_11.png)

画像分類テストを実行します
```
./bin/cvi_sample_classifier_fused_preprocess \
 ../mobilenet_v2_cv1800_int8_asym.cvimodel \
 ./data/cat.jpg \
 ./data/synset_words.txt
```

成功した分類結果の例

![duo](/docs/duo/tpu/duo-tpu-mobilenetv2_12.png)
