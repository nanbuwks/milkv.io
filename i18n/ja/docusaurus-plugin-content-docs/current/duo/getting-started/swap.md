---
sidebar_label: 'スワップとは'
sidebar_position: 30
---

#　Swapとは

スワップ (仮想 RAM とも呼ばれます) は、物理メモリ (RAM) がいっぱいになった場合に、ハードディスクへのデータの保存をサポートするために使用されます。物理メモリがまだ使い尽くされていない場合でも、キャッシュ容量を増やすためにスワップが並行して使用される場合もあります。

# Duo でスワップを有効にする方法

:::注意
microSD カードの消耗が発生し、ドライブの寿命が短くなる可能性があります。使用可能なメモリが不足している場合にのみ、スワップ機能を有効にすることを強くお勧めします。
:::

最新のシステムイメージを使用してください。

[https://github.com/milkv-duo/duo-buildroot-sdk/releases](https://github.com/milkv-duo/duo-buildroot-sdk/releases)

次の 2 つのコマンドを実行してスワップを有効にします。
```
mkswap /dev/mmcblk0p3
swapon /dev/mmcblk0p3
```

次に、 `free -h`  コマンドを実行して、スワップがアクティブかどうかを確認します (256M)

```
[root@milkv-duo]~# free -h
              total        used        free      shared  buff/cache   available
Mem:          28.8M       13.6M        9.1M       76.0K        6.1M       12.4M
Swap:        256.0M           0      256.0M
```

# スワップパーティションのサイズを増やす方法
スワップ パーティションのデフォルトのファームウェア サイズは`256M` です。スワップ パーティションのサイズを増やすには 2 つの方法があります

## 1. SDKを変更して再コンパイルして新しいファームウェアを生成します

 [このサイズ](https://github.com/milkv-duo/duo-buildroot-sdk/blob/develop/milkv/genimage-milkv-duo.cfg#L36) の値を変更し、再コンパイルしてファームウェアを生成できます。

## 2. Duo で fdisk コマンドを使用して直接変更する

シリアル ケーブルまたは SSH 経由で Duo に接続すると、`fdisk` コマンドを使用して変更できます。原則として、既存のスワップ パーティションを削除し、指定されたサイズで新しいスワップ パーティションを作成することが含まれます。
:::注意
パーティションに変更を加えると、データ損失が発生する可能性があります。パーティション操作を実行する前に、必ず重要なデータをバックアップしてください。
:::

以下は、`fdisk` を使用したコマンドライン対話型の方法を段階的に示しています。この方法に慣れていない場合は、スクリプトベースのアプローチが後で提供されます。これを 1 回実行して変更を完了できます。

### fdisk のコマンドライン対話モード

`fdisk`  コマンドを入力すると、`fdisk`  のコマンドライン対話モードが開始されます。
```
fdisk /dev/mmcblk0
```
`p`コマンドを入力すると、MicroSD カード上の現在のパーティションの情報を表示できます。`swap`に使用されるパーティション /dev/mmcblk0p3 のサイズが `256M`であることがわかります。
```
Command (m for help): p
Disk /dev/mmcblk0: 15 GB, 15931539456 bytes, 31116288 sectors
486192 cylinders, 4 heads, 16 sectors/track
Units: sectors of 1 * 512 = 512 bytes

Device       Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/mmcblk0p1 *  0,0,2       16,81,2              1     262144     262144  128M  c Win95 FAT32 (LBA)
/dev/mmcblk0p2    16,81,3     114,57,8        262145    1835008    1572864  768M 83 Linux
/dev/mmcblk0p3    114,57,9    146,219,10     1835009    2359296     524288  256M  0 Empty
```

 `/dev/mmcblk0p3` の `StartLBA`(開始セクター) 値 `1835009` をメモしておいてください。後で必要になります。

パーティションを削除するには、`d` コマンドを実行し、削除するパーティション番号を入力する必要があります。この場合、`/dev/mmcblk0p3` を削除したいので、`3`を入力します。

```
Command (m for help): d
Partition number (1-4): 3
```


新しいパーティションを作成するには、`n` コマンドを使用できます。プロンプトが表示されたら、プライマリ パーティションを作成するか拡張パーティションを作成するかを選択します。プライマリ パーティションを作成したいので、`p`と入力します。次に、作成するパーティション番号を入力します。`/dev/mmcblk0p3` の場合は`3`です。

次のプロンプト`最初のセクター`では、新しいパーティションの開始セクター値 (上記で記録した値 `1835009`) を入力する必要があります。この値はプロンプトでデフォルトで認識されることがわかります。そのため、入力せずに Enter キーを押すだけで済みます。

最後に、新しいパーティションのサイズを入力するように求められます。終了セクター値を入力するか、パーティションのサイズを直接指定できます。たとえば、1GB を割り当てる場合は、プロンプトに従って `+1G`を入力します。
```
Command (m for help): n
Partition type
   p   primary partition (1-4)
   e   extended
p
Partition number (1-4): 3
First sector (1835009-31116287, default 1835009): 
Using default value 1835009
Last sector or +size{,K,M,G,T} (1835009-31116287, default 31116287): +1G
```

次に、`p` コマンドを再度実行して、現在のパーティションの状態を確認します。 `/dev/mmcblk0p3`が`1024M`になっていることがわかります。これは`1G`に相当します。

最後に、`w`コマンドを実行して変更を保存し、fdisk 対話モードを終了します。

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table
fdisk: WARNING: rereading partition table failed, kernel still uses old table: Resource busy
```

この時点で、パーティションの変更は、`reboot`コマンドを実行してシステムを再起動するか、デバイスの電源を入れ直すまで有効になりません。

スワップを有効にするには、Duo ターミナルで次の 2 つのコマンドを実行します。
```
mkswap /dev/mmcblk0p3
swapon /dev/mmcblk0p3
```

次に、コマンド `free -h`を実行して、スワップが有効になっているかどうかを確認します (1G、1024M に相当)

### fdiskのスクリプトベースのアプローチ

Duo で `swap_resize.sh`という名前のスクリプト ファイルを作成するか、PC 上で作成して Duo にアップロードします。

`swap_resize.sh`ファイルの内容を、`+1G`を希望のサイズに変更することに注意してください。
```bash
fdisk /dev/mmcblk0 <<EOF
p
d
3
n
p
3

+1G

w
EOF

fdisk -l /dev/mmcblk0

echo "----- resize /dev/mmcblk0p3 for swap completed -----"
```

Duo でこのスクリプトを実行すると、スワップ パーティションのサイズが自動的に変更されます
```bash
sh swap_resize.sh
```

同様に、変更を有効にするには、`reboot`コマンドを実行してシステムを再起動するか、デバイスの電源を入れ直す必要があります。その後、再びスワップをアクティブ化できます
```
mkswap /dev/mmcblk0p3
swapon /dev/mmcblk0p3
```

次に、コマンド `free -h`を実行して、スワップが有効になっているかどうかを確認します (1G、1024M に相当)

```
[root@milkv-duo]~# free -h
              total        used        free      shared  buff/cache   available
Mem:          28.8M       13.8M        8.2M       76.0K        6.8M       12.3M
Swap:       1024.0M           0     1024.0M
```