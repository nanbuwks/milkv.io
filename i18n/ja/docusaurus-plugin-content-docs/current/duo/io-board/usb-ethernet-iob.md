---
sidebar_label: 'Duo USB&Ethernet IOB '
sidebar_position: 10
---
# Duo USB&Ethernet IO-Board

## はじめに

Duo USB&Ethernet IO-Board は Milk-V Duo に USB x 4, RJ45 x 1, シリアルポートピン x 1 , Type-C 電源入力コネクターを追加します。

この拡張ボードは Duo の開発効率を向上させ、開発者が一般的に使用する USB 周辺機器や イーサネットにアクセスすることを促進します。

![duousbeth](/docs/duo/duousbethiob.webp)

## スペック

- 4x USB
- 1x 100Mbps RJ45
- 1x Serial port pinout
- 1x Type-C power input connector

## 使用方法

使用する前に、TFカードに [最新のOSイメージ](https://milkv.io/docs/duo/resources/image-sdk)を作成してください。



### 使用 Duo IO-Board

注：IOボードを使用する場合、Duo本体のRNDIS USBネットワークは使用できません。IOボードのイーサネットインターフェースをご利用ください。

もしあなたがIOボードのイーサネットポートに固定した MAC アドレスを割り当てたい場合、以下の作業を行ってください(** 操作説明中の MAC アドレスは利用者が希望するアドレスに変更してください。複数のデバイスを使用する場合、同じネットワークセグメント中で重複しないようにする必要があります址**)。
```
echo "pre-up ifconfig eth0 hw ether 78:01:B3:FC:E8:55" >> /etc/network/interfaces && sync
```
これを実行した後に再起動をしてください。

IOボード上の 4 USB を有効にするためには:
~~~
rm /mnt/system/usb.sh
ln -s /mnt/system/usb-host.sh /mnt/system/usb.sh
sync
~~~
これを実行した後に再起動をしてください。

例えば、USB フラッシュメモリを IO ボード上のUSBポートに接続した場合、`ls /dev/sd*` の操作でデバイスの認識状態を確認できます。

USB ドライブをマウントし、内容を確認するには(/dev/sda1 の場合の例):
```
mkdir /mnt/udisk
mount /dev/sda1 /mnt/udisk
```
`/mnt/udisk` の内容を確認するには:
```
ls /mnt/udisk
```

アンマウントするには以下の操作を行います。
```
umount /mnt/udisk
```
このあと reboot を行ってください。


IO-ボードをつかわなくなったとき、USBネットワーク(RNDIS)を以下のようにして復活させます。
~~~
rm /mnt/system/usb.sh
ln -s /mnt/system/usb-rndis.sh /mnt/system/usb.sh
sync
~~~
このあと reboot を行ってください。

## 回路図
- [Hardware schematics](https://github.com/milkv-duo/accessories/blob/master/Duo_USB%26Ethernet_IOB/duo_iob_v1.11.pdf)
