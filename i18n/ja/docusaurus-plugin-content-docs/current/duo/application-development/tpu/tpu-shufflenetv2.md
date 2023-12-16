---
sidebar_label: 'ShuffleNetV2 画像分類'
sidebar_position: 36
---

# ShuffleNetV2に基づく画像分類

## 1. Docker開発環境の設定

[こちら](https://milkv.io/docs/duo/application-development/tpu/tpu-docker)を参照してください。Docker開発環境を設定した後、ここに戻って次のステップを続けてください。

## 2. Docker内で作業ディレクトリを準備する

`shufflenet_v2`作業ディレクトリを作成し、それに入ります。これは`tpu-mlir`と同じレベルのディレクトリであることに注意してください。
```
# mkdir shufflenet_v2 && cd shufflenet_v2
```

Copy test image:
```
# cp -rf ${TPUC_ROOT}/regression/dataset/ILSVRC2012/ .
# cp -rf ${TPUC_ROOT}/regression/image/ .
```
ここでの`${TPUC_ROOT}`は環境変数で、`tpu-mlirデ`ィレクトリに対応しており、Docker開発環境の前の設定で`source ./tpu-mlir/envsetup.sh`ステップでロードされます。

新しい`export.py`ファイルを作成し、そのファイルに以下のコードを書き込みます：

```
import torch
from torchvision.models.shufflenetv2 import shufflenet_v2_x1_0
model = shufflenet_v2_x1_0(pretrained=True)
model.eval()
torch.jit.trace(model, torch.randn(1, 3, 640, 640)).save("./shufflenetv2_jit.pt")
```

`export.py`ファイルを実行します：
```
python export.py
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_05.png)

現在のディレクトリに`shufflenetv2_jit.pt`ファイルが生成されます。これは必要な元のモデルファイルです。

`MLIR`や`cvimodel`などのコンパイル済みファイルを保存するための`work`作業ディレクトリを作成し、その中に入ります
```
# mkdir work && cd work
```

## 3. ShuffleNetV2-PyTorch モデルの変換

:::tip
Duo開発ボードはCV1800Bチップを搭載しており、**ONNX series** と **Caffe models**をサポートしています。現在、TFLiteモデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::

モデル変換の手順は以下の通りです：

- CaffeモデルをMLIRに変換
- 量子化に必要なキャリブレーションテーブルを生成
- MLIRを非対称INT8 cvimodelに量子化


### PyTorchモデルをMLIRに変換

モデルの入力は画像です。モデルを変換する前に、モデルの前処理を理解する必要があります。モデルが前処理済みのnpzファイルを入力として使用する場合、前処理を考慮する必要はありません。前処理のプロセスは次のように表されます（xは入力を表します）:
 $$ y = (x-mean)\times scale $$

この例のモデルはBGR入力で、`mean`と`scale`はそれぞれ`103.94,` `116.78`, `123.68`と`0.017`,`0.017`,`0.017`です。モデル変換コマンドは以下の通りです：

```
model_transform.py \
 --model_name shufflenet_v2 \
 --model_def ../shufflenetv2_jit.pt \
 --input_shapes [[1,3,224,224]] \
 --resize_dims=256,256 \
 --mean 103.94,116.78,123.68 \
 --scale 0.017,0.017,0.017 \
 --pixel_format bgr \
 --test_input ../image/cat.jpg \
 --test_result shufflenet_v2_top_outputs.npz \
 --mlir shufflenet_v2.mlir
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_06.png)

MLIRモデルに変換した後、`shufflenet_v2.mlir`ファイルが生成されます。これはMLIRモデルファイルです。また、`shufflenet_v2_in_f32.npz`ファイルと`shufflenet_v2_top_outputs.npz`ファイルも生成されます。これらは、後続のモデル変換の入力ファイルです。

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_07.png)

### MLIRをINT8モデルに変換

#### 量子化に必要なキャリブレーションテーブルを生成

`run_calibration.py`を実行してキャリブレーションテーブルを取得します。入力データの数は状況によりますが、おおよそ100〜1000枚程度であるべきです。ここでは、既存のILSVRC2012からの100枚の画像を例に、キャリブレーションコマンドを実行します：
```
run_calibration.py shufflenet_v2.mlir \
 --dataset ../ILSVRC2012 \
 --input_num 100 \
 -o shufflenet_v2_cali_table
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_08.png)

操作が完了すると、`shufflenet_v2_cali_table`ファイルが生成されます。これは、後続のINT8モデルのコンパイルに使用されます。

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_09.png)

#### MLIRを非対称INT8 cvimodelに量子化

`model_deploy.py`スクリプトパラメータを使用して非対称を非対称量子化に使用し、MLIRファイルをINT8非対称量子化モデルに変換します：
```
model_deploy.py \
 --mlir shufflenet_v2.mlir \
 --asymmetric \
 --calibration_table shufflenet_v2_cali_table \
 --fuse_preprocess \
 --customization_format BGR_PLANAR \
 --chip cv180x \
 --quantize INT8 \
 --test_input ../image/cat.jpg \
 --tolerance 0.96,0.72 \
 --model shufflenet_v2_cv1800_int8_asym.cvimodel
```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_10.png)

コンパイルが完了すると、`shufflenet_v2_cv180x_int8_fuse.cvimodel`ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_11.png)

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
# scp /workspace/shufflenet_v2/work/shufflenet_v2_cv1800_int8_asym.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
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

samplesディレクトリに入ります

```
# cd samples
```

cvimodelの情報を表示します
```
./bin/cvi_sample_model_info ../shufflenet_v2_cv1800_int8_asym.cvimodel
```

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_12.png)

画像分類テストを実行します
```
./bin/cvi_sample_classifier_fused_preprocess \
 ../shufflenet_v2_cv1800_int8_asym.cvimodel \
 ./data/cat.jpg \
 ./data/synset_words.txt
```

成功した分類結果の例

![duo](/docs/duo/tpu/duo-tpu-shufflenetv2_13.png)
