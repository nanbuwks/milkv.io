---
sidebar_label: 'セットアップ'
sidebar_position: 20
---

# 作業環境をセットアップする

## USBnet セットアップ

USB ネットワークを使用するために、システムではデフォルトで RNDIS と DHCP が有効になっています。
### Windows

1. Type-C ケーブルを介して Duo をコンピュータに接続します。

2. 「RNDIS」デバイスがデバイスマネージャーに表示されます。

![rndis-step1](/docs/duo/rndis-step1.png)

3. 「RNDIS」を選択し、右クリックしてドライバーを更新します。

![rndis-step2](/docs/duo/rndis-step2.png)

4. 「コンピューターを参照してドライバーを探す」を選択します。

![rndis-step3](/docs/duo/rndis-step3.png)

5.「コンピューター上で利用可能なドライバーのリストから選択させてください」を選択します。

![rndis-step4](/docs/duo/rndis-step4.png)

6. 「ネットワークアダプター」を選択します

![rndis-step5](/docs/duo/rndis-step5.png)

7. Microsoft/USB RNDIS Adapterを選択します

![rndis-step6](/docs/duo/rndis-step6.png)

8. 警告メッセージを無視して「はい」をクリックします

![rndis-step7](/docs/duo/rndis-step7.png)

9. ドライバーのアップデートが成功しました

![rndis-step8](/docs/duo/rndis-step8.png)

10. "USB RNDIS Adapter"があるか確認してください。

![rndis-step9](/docs/duo/rndis-step9.png)

11. IP を見つけて、ping を使用してネットワークをテストします。

![rndis-step10](/docs/duo/rndis-step10.png)

rndis ドライバーのインストールに問題がある場合は、次のことを試してください。
[別方法](https://milkv.io/docs/duo/getting-started/windows-rndis-dirver)

### Linux

一般に、Linux では設定なしで RNDIS を使用できます。

コマンド ip を使用して、usb0 ネットワークを確認できます。

```
neko@milk-v:~ sudo dmesg | grep usb0
[1055270.386719] rndis_host 1-2.1:1.0 usb0: register 'rndis_host' at usb-0000:00:14.0-2.1, RNDIS device, aa:53:5d:bb:7f:28
[1055270.423753] rndis_host 1-2.1:1.0 enxaa535dbb7f28: renamed from usb0
neko@milk-v:~ ip addr show enxaa535dbb7f28
15: enxaa535dbb7f28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 42:a2:79:19:7f:e3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.42.69/24 brd 192.168.42.255 scope global dynamic noprefixroute enp0s20f0u1
       valid_lft 3569sec preferred_lft 3569sec
    inet6 fe80::3c92:ed74:3475:cb9c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
neko@milk-v:~ ping 192.168.42.1 -c 5
PING 192.168.42.1 (192.168.42.1) 56(84) bytes of data.
64 bytes from 192.168.42.1: icmp_seq=1 ttl=64 time=0.334 ms
64 bytes from 192.168.42.1: icmp_seq=2 ttl=64 time=0.287 ms
64 bytes from 192.168.42.1: icmp_seq=3 ttl=64 time=0.275 ms
64 bytes from 192.168.42.1: icmp_seq=4 ttl=64 time=0.287 ms
64 bytes from 192.168.42.1: icmp_seq=5 ttl=64 time=0.266 ms

--- 192.168.42.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4096ms
rtt min/avg/max/mdev = 0.266/0.289/0.334/0.031 ms
neko@milk-v:~ 
```

### macOS

RNDIS の公式ドライバーはありません。[HoRNDIS](https://joshuawise.com/horndis)をインストールする必要があります.

1. HoRNDISドライバーをダウンロードします
  - Intel https://github.com/jwise/HoRNDIS/releases
  - Apple silicon https://github.com/jwise/HoRNDIS/files/7323710/HoRNDIS-M1.zip

2. システム整合性保護を無効にします

    a. macOSのリカバリーシステムに入ります

    参考 [macOS User Guide -> Recovery](https://support.apple.com/en-hk/guide/mac-help/mchl338cf9a8/mac) リカバリを参照してください。.

    b. ターミナルを開き、以下のコマンドを入力します

   ```
    csrutil disable

    csrutil enable --without kext
   ```

    c. Macを再起動します

3. ZIP パック内の Kext 拡張機能をインストールします

4. ネットワーク設定を確認してください

## SSH

1. ターミナルを開き **ssh root@192.168.42.1**、と入力しyesと答えます

![rndis-ssh1](/docs/duo/rndis-ssh1.png)

2. パスワード  **milkv** を入力します (パスワードは画面に表示されません)

![rndis-ssh2](/docs/duo/rndis-ssh2.png)

3. ログインに成功します

![rndis-ssh3](/docs/duo/rndis-ssh3.png)


## シリアルコンソール

### USB to TTL シリアルケーブル

USB-to-TTL ケーブルの各ピンは次のように定義されます。

![usb2ttl](/docs/duo/usb2ttl.jpg)

### コネクション

以下に示すように、USB to TTL シリアル ケーブルを接続します。赤い線は接続しないでください。


| Milk-V Dou   | <---> | USB to TTL |
| ------------ | ----- | ---------- |
| TX (pin 16)  | <---> | 白線 |
| RX (pin 17)  | <---> | 緑線 |
| GND (pin 18) | <---> | 黒線 |


![duo-serial](/docs/duo/duo-serial.jpg)

Duo u-boot とカーネル コンソールのデフォルトのシリアル設定は次のとおりです。

```
   baudrate: 115200
   data bit: 8
   stop bit: 1
   parity  : none
   flow control: none
```

## sysroot
