---
sidebar_label: 'Docker開発環境の設定'
sidebar_position: 20
---

# Docker 開発環境の設定

## Docker のインストール

Windows 環境では、`Docker Desktop for Windows`をインストールできます。[Dockerのダウンロード](https://docs.docker.com/desktop/install/windows-install/)

![duo](/docs/duo/tpu/duo-tpu-docker_01.png)

Windows 下で Docker を実行するには、関連する依存関係が必要です。図に示すように、WSL2 バックエンドまたは Hyper-V バックエンドを実行依存関係として使用する必要があります。

Hyper-Vバックエンドの有効化方法は以下の通りです：

1. コントロールパネル - プログラムと機能 - Windowsの機能の有効化または無効化
2. `Hyper-V`を見つけ、`Hyper-V Management Tools`と`Hyper-V Platform`にチェックを入れ、OKをクリックしてシステムファイルの設定が完了するのを待ち、その後コンピュータを再起動します。

   ![duo](/docs/duo/tpu/duo-tpu-docker_02-en.png)

次に、`Docker Desktop for Windows`をインストールし、ダウンロードします。インストールガイドに従って、選択したバックエンドに応じて適切なチェックを行います。

インストールが完了したら、コンピュータを再起動してから Docker を使用できます。

## 開発に必要な Docker イメージをプルする

Docker hubからイメージファイルを取得します
```
docker pull sophgo/tpuc_dev:v3.1
```

```
PS C:\Users\Carbon> docker pull sophgo/tpuc_dev:v3.1
v3.1: Pulling from sophgo/tpuc_dev
b237fe92c417: Pull complete
db3c30810eab: Downloading  411.1MB/629.4MB
2651dfd68288: Download complete
db3c5981ae16: Download complete
16098f82aa65: Download complete
85e8821c88fd: Download complete
d8a25a7307da: Download complete
91fd425676de: Download complete
b3ad6c6ed19d: Downloading  346.1MB/480.6MB
ecaa420e1520: Download complete
a570c4642598: Download complete
2d76e68a7946: Download complete
1df3b38113a9: Download complete
4f4fb700ef54: Download complete
f835d42d7adc: Download complete
2b009425c205: Downloading  252.9MB/1.098GB
```

## Docker コンテナの起動

```
docker run --privileged --name <container_name> -v /workspace -it sophgo/tpuc_dev:v3.1
```
`<container_name>`は自分で定義したコンテナ名です。例えば DuoTPU です。
```
docker run --privileged --name DuoTPU -v /workspace -it sophgo/tpuc_dev:v3.1
```

## Dockerコンテナへのログイン

コンテナを起動した後、自動的にDockerウィンドウのターミナルインターフェースにログインします。新しいウィンドウを開く必要がある場合は、以下のように行います。

`docker ps`コマンドを使用して、現在のDockerコンテナリストを表示します。
```
PS C:\Users\Carbon\Duo-TPU> docker ps
CONTAINER ID   IMAGE                  COMMAND
f3a060efb1d3   sophgo/tpuc_dev:v3.1   "/bin/bash"
```

`CONTAINER ID`を使用してDockerコンテナにログインします。
```
docker exec -it f3a060efb1d3 /bin/bash
```

Dockerターミナルで、現在のディレクトリが `/workspace` であるか確認します。そうでない場合は、 `cd` コマンドを使用してディレクトリに入ります。
```
# cd /workspace/
```

![duo](/docs/duo/tpu/duo-tpu-docker_03.png)


## 開発キットの取得と環境変数の追加

Docker ターミナルで TPU-MLIR モデル変換ツールキットをダウンロードします。
```
git clone https://github.com/milkv-duo/tpu-mlir.git
```

Docker ターミナルで、`source`コマンドを使用して環境変数を追加します。
```
# source ./tpu-mlir/envsetup.sh
```
