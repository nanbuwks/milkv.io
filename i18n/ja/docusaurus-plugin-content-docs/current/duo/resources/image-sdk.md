---
sidebar_label: '公式イメージとSDK'
sidebar_position: 10
---

# 公式イメージとSDK

## イメージ
- **(最新版)** [サイトのmilkv-duo-xxx.img.zipを使ってください](https://github.com/milkv-duo/duo-buildroot-sdk/releases/)
    新しいイメージとSDKをアップロード:

    sshを有効化  
    RNDISを有効化  
    LEDが点滅  
    rootのパスワード: milkv  
    RNDIS経由でsshでログイン:  
    ```
    ssh root@192.168.42.1  
    ```
    LEDの点滅機能をオフにしたい場合:
    ```
    mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
    ```
    その後、rebootコマンドを実行するか、電源を再投入します。

    これは、LEDの点滅スクリプトの名前を変更し、Duoを再起動すると、LEDは点滅しなくなります。

    LEDの点滅を復元したい場合、その名前を元に戻し、デバイスを再起動します。

    ```
    mv /mnt/system/blink.sh_backup /mnt/system/blink.sh && sync
    ```
    その後、rebootコマンドを実行するか、電源を再投入します。

    IO-Boardの使用方法
    (https://milkv.io/docs/duo/io-board/usb-ethernet-iob)

## SDK

### Milk-V Duo公式のbuildroot SDK
duo-buildroot-sdk [https://github.com/milkv-duo/duo-buildroot-sdk](https://github.com/milkv-duo/duo-buildroot-sdk)


### Milk-V Duo公式のC/C++アプリケーション開発参考例
duo-examples [https://github.com/milkv-duo/duo-examples](https://github.com/milkv-duo/duo-examples
)