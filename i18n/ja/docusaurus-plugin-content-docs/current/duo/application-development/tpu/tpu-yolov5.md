---
sidebar_label: 'YOLOv5 物体検出'
sidebar_position: 30
---

# YOLOv5に基づく物体検出

## 1. Windowsで元のモデルファイルを準備する

### YOLOv5 開発キットと yolov5n.pt ファイルを準備する


[YOLOv5 development toolkit](https://codeload.github.com/ultralytics/yolov5/zip/refs/heads/master)と [yolov5n.pt](https://github.com/ultralytics/yolov5/releases/download/v6.2/yolov5n.pt)をダウンロードします。ダウンロードが完了したら、ツールキットを解凍し、`yolov5n.pt` ファイルを `yolov5-master` ディレクトリに配置します。

### conda 環境を設定する

事前に[Anaconda](https://www.anaconda.com/) をインストールする必要があります。

新しい`Anaconda Prompt` ターミナルを開き、`conda env list` を実行して現在の環境を表示します
```
(base) C:\Users\Carbon> conda env list
# conda environments:
#
base                  *  C:\Users\Carbon\anaconda3
```

新しい conda 仮想環境を作成し、python のバージョン 3.9.0 をインストールします。`duotpu` の名前はあなたが自由に選んでください。
```
(base) C:\Users\Carbon> conda create --name duotpu python=3.9.0
```

成功した後、再度現在の環境を確認します。
```
(base) C:\Users\Carbon> conda env list
# conda environments:
#
base                  *  C:\Users\Carbon\anaconda3
duotpu                   C:\Users\Carbon\anaconda3\envs\duotpu
```

新しくインストールした 3.9.0 の環境をアクティベートします。
```
(base) C:\Users\Carbon> activate duotpu
```

アクティベーションが成功したことを確認します。
```
(duotpu) C:\Users\Carbon> conda env list
# conda environments:
#
base                     C:\Users\Carbon\anaconda3
duotpu                *  C:\Users\Carbon\anaconda3\envs\duotpu
```

その後、以下のコマンドを使用して PyTorch バージョン 1.12.1 をインストールできます。具体的なインストールコマンドは、あなたの要件に基づいて選択してください。後続のプロセスでは CPU のみが必要です。
```
# CUDA 10.2
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=10.2 -c pytorch

# CUDA 11.3
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.3 -c pytorch

# CUDA 11.6
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.6 -c pytorch

# CPU Only
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cpuonly -c pytorch
```

次に、ターミナルのパスを開発キットの `yolov5-master` パスに `cd` し、`pip install -r requirements.txt` を入力して他の依存関係をインストールします。

```
(duotpu) C:\Users\Carbon> cd Duo-TPU\yolov5-master

(duotpu) C:\Users\Carbon\Duo-TPU\yolov5-master> pip install -r requirements.txt
```

### 元のモデルファイルを生成する

`yolov5-master` ディレクトリに新しい `main.py` ファイルを作成し、そのファイルに以下のコードを書き込みます：
```
import torch
from models.experimental import attempt_download
model = torch.load(attempt_download("./yolov5n.pt"),
map_location=torch.device('cpu'))['model'].float()
model.eval()
model.model[-1].export = True
torch.jit.trace(model, torch.rand(1, 3, 640, 640), strict=False).save('./yolov5n_jit.pt')
```

次に、`yolov5-master/models/yolo.py` ファイルを見つけ、63 行目から 79 行目までのコードをコメントアウトし、80 行目に `return x` のコードを追加します。以下のようになります：

![duo](/docs/duo/tpu/duo-tpu-yolo5_01.png)

さらに、このファイルも修正する必要があります：
```
C:\Users\Carbon\anaconda3\envs\duotpu\Lib\site-packages\torch\nn\modules\upsampling.py
```
153 行目付近で以下の変更を行います

![duo](/docs/duo/tpu/duo-tpu-yolo5_02.png)

修正が完了したら、`python main.py` ファイルを実行し、`yolov5-master` ディレクトリに `yolov5n_jit.pt` ファイルが生成されます。このファイルは、必要な元のモデルファイルです。
```
(duotpu) C:\Users\Carbon\Duo-TPU\yolov5-master> python main.py
```

### conda環境を終了する（オプション）

上記で必要なモデルファイルが生成されました。`conda deactivate`コマンドを使用して conda 環境を終了できます：
```
(duotpu) C:\Users\Carbon\Duo-TPU\yolov5-master> conda deactivate
```

この conda 仮想環境（duotpu）がもう必要ない場合は、以下のコマンドで削除できます：
```
conda env remove --name <envname>
```

## 2. Docker 開発環境の設定

[こちら](https://milkv.io/docs/duo/application-development/tpu/tpu-docker)を参照してください。Docker 開発環境を設定した後、ここに戻って次のステップを続けてください

## 3. Docker内で作業ディレクトリを準備する

`yolov5n_torch` 作業ディレクトリを作成し、それに入ります。これは `tpu-mlir` と同じ階層のディレクトリであることに注意してください。そして、モデルファイルと画像ファイルをこのディレクトリに入れます。
```
# mkdir yolov5n_torch && cd yolov5n_torch
```

新しい Windows ターミナルを作成し、`yolov5n_jit.pt`を Windows から Docker にコピーします。
```
docker cp <path>/yolov5-master/yolov5n_jit.pt <container_name>:/workspace/yolov5n_torch/yolov5n_jit.pt
```

`<path>` は、Windows システム内の yolov5 開発キットが位置しているファイルディレクトリで、`<container_name>`はコンテナ名です。例えば、
```
docker cp C:\Users\Carbon\Duo-TPU\yolov5-master\yolov5n_jit.pt DuoTPU:/workspace/yolov5n_torch/yolov5n_jit.pt
```

Dockerターミナルに戻り、画像ファイルを現在のディレクトリ（`yolov5n_torch`）に入れます。
```
# cp -rf ${TPUC_ROOT}/regression/dataset/COCO2017 .
# cp -rf ${TPUC_ROOT}/regression/image .
```
ここでの`${TPUC_ROOT}`は環境変数で、`tpu-mlir`ディレクトリに対応しており、Docker 開発環境の前の設定で `source ./tpu-mlir/envsetup.sh` ステップでロードされます。

`MLIR` や `cvimodel` などのコンパイル済みファイルを保存するための `work` 作業ディレクトリを作成し、その中に入ります。
```
# mkdir work && cd work
```

## 4. YOLOv5n-TORCH モデルの変換

:::tip
Duo開発ボードはCV1800Bチップを搭載しており、**ONNX series** と **Caffe models**をサポートしています。現在、TFLite モデルはサポートしていません。量子化データ型については、**quantization in BF16 format** と **asymmetric quantization in INT8 format**をサポートしています。
:::

モデル変換の手順は以下の通りです：

- Caffe モデルをMLIR に変換
- 量子化に必要なキャリブレーションテーブルを生成
- MLIR を非対称 INT8 cvimodel に量子化

### TORCH モデルを MLIR に変換

この例では、モデルはRGB入力で、`mean`と`scale`はそれぞれ`0,0,0`と`0.0039216`, `0.0039216`, `0.0039216`です。 TORCHモデルをMLIRモデルに変換するコマンドは以下の通りです。
 ```
# model_transform.py \
 --model_name yolov5n \
 --model_def ../yolov5n_jit.pt \
 --input_shapes [[1,3,640,640]] \
 --pixel_format "rgb" \
 --keep_aspect_ratio \
 --mean 0,0,0 \
 --scale 0.0039216,0.0039216,0.0039216 \
 --test_input ../image/dog.jpg \
 --test_result yolov5n_top_outputs.npz \
 --output_names 1219,1234,1249 \
 --mlir yolov5n.mlir
 ```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-yolo5_06.png)

MLIRモデルに変換した後、`yolov5n.mlir`ファイルが生成されます。これはMLIRモデルファイルです。また、`yolov5n_in_f32.npz`ファイルと`yolov5n_top_outputs.npz`ファイルも生成されます。これらは、後続のモデル変換の入力ファイルです。

![duo](/docs/duo/tpu/duo-tpu-yolo5_07.png)

### MLIR を INT8 モデルに変換

#### 量子化に必要なキャリブレーションテーブルを生成

INT8モデルに変換する前に、キャリブレーションテーブルを生成する必要があります。ここでは、既存のCOCO2017からの100枚の画像を例に、キャリブレーションを実行します。
```
# run_calibration.py yolov5n.mlir \
 --dataset ../COCO2017 \
 --input_num 100 \
 -o ./yolov5n_cali_table
 ```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-yolo5_08.png)

コンパイルが完了すると、`yolov5n_int8_fuse.cvimodel`ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-yolo5_09.png)

#### MLIRを非対称INT8 cvimodelに量子化

MLIRモデルをINT8モデルに変換するコマンドは以下の通りです。
```
# model_deploy.py \
 --mlir yolov5n.mlir \
 --quantize INT8 \
 --calibration_table ./yolov5n_cali_table \
 --chip cv180x \
 --test_input ../image/dog.jpg \
 --test_reference yolov5n_top_outputs.npz \
 --compare_all \
 --tolerance 0.96,0.72 \
 --fuse_preprocess \
 --debug \
 --model yolov5n_int8_fuse.cvimodel
 ```

成功した操作の例

![duo](/docs/duo/tpu/duo-tpu-yolo5_10.png)

コンパイルが完了すると、`yolov5n_int8_fuse.cvimodel`ファイルが生成されます。

![duo](/docs/duo/tpu/duo-tpu-yolo5_11.png)

## 5. Duo 開発ボードでの検証

### Duo 開発ボードの接続

前のチュートリアルに従ってDuo開発ボードとコンピューターの接続を完了し、`mobaxterm`や`Xshell`などのツールを使用して Duo 開発ボードを操作するためのターミナルを開きます。

### tpu-sdk の取得

Docker ターミナルで`/workspace`ディレクトリに切り替えます
```
cd /workspace
```

tpu-sdk をダウンロードします
```
git clone https://github.com/milkv-duo/tpu-sdk.git
```

### tpu-sdk とモデルファイルを Duo にコピー

Duo ボードのターミナルで、新しいディレクトリ `/mnt/tpu/` を作成します
```
# mkdir -p /mnt/tpu && cd /mnt/tpu
```

Dockerターミナルで、`tpu-sdk`とモデルファイルを Duo にコピーします
```
# scp -r /workspace/tpu-sdk root@192.168.42.1:/mnt/tpu/
# scp /workspace/yolov5n_torch/work/yolov5n_int8_fuse.cvimodel root@192.168.42.1:/mnt/tpu/tpu-sdk/
```

### 環境変数を設定

Duo ボードのターミナルで、環境変数を設定します
```
# cd /mnt/tpu/tpu-sdk
# source ./envs_tpu_sdk.sh
```

### 物体検出を実行

Duo ボード上で、画像に対して物体検出を実行します

![duo](/docs/duo/tpu/duo-tpu-dog.jpg)

`yolov5n_int8_fuse.cvimodel`モデルを使用した物体検出：
```
# ./samples/samples_extra/bin/cvi_sample_detector_yolo_v5_fused_preprocess \
 ./yolov5n_int8_fuse.cvimodel \
 ./samples/samples_extra/data/dog.jpg \
 yolov5n_out.jpg
 ```

成功した検出結果の例

![duo](/docs/duo/tpu/duo-tpu-yolo5_12.png)

操作が成功すると、検出結果ファイル `yolov5n_out.jpg` が生成されます。これは、Windows ターミナルの scp コマンドを通じて PC に引き出すことができます。
```
scp root@192.168.42.1:/mnt/tpu/tpu-sdk/yolov5n_out.jpg .
```

![duo](/docs/duo/tpu/duo-tpu-yolo5_13.png)

Windows PC 上でテスト結果ファイルを表示します。

![duo](/docs/duo/tpu/duo-tpu-yolo5_14.jpg)
