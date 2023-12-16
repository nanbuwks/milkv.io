---
sidebar_label: '🌍 概要'
sidebar_position: 1
---

# Duo
| Version | 日付          | 変更内容                                              |
|---------|----------------|----------------------------------------------------------------------------|
|0.1      |  2023/11/24    | 初期版                                                              |
|0.2      |  2023/12/15    | 256M版の発表                              |
--------------------------------------------------------------------------------------------------------
-------------------------
# 紹介
Milk-V Duoは、CV1800Bチップを基にしたコンパクトな組み込み開発プラットフォームです。LinuxとRTOSを実行でき、専門家、工業ODM、AIoT、DIY、クリエイターに信頼性の高い、低コスト、高性能のプラットフォームを提供します。  
  
![duo](/docs/duo/duo-v1.2.png)

# 仕様
|            | Duo                                                     | Duo 256M                                         |
| ---------- | ------------------------------------------------------- | ------------------------------------------------ |
| SoC        | CVITEK CV1800B                                          | SG2002                                           |
| RISC-V CPU | C906@1Ghz + C906@700MHz                                 | C906@1Ghz + C906@700MHz                          |
| Arm CPU    | N/A                                                     | 1 x Cortex-A53@1GHz                                |
| MCU        | N/A                                                     | 8051@6KB SRAM                                    |
| NPU        | 0.5Top@INT8                                             | 1Top@INT8                                        |
| Storage    | 1 x microSD connector or 1x SD NAND on board             | 1 x microSD connector or 1x SD NAND on board      |
| Memory     | DDR2 64MB                                               | DDR3 256MB                                       |
| Storage    | 1 x Mirco SD slot,1x SD NAND solder pad                  | 1 x microSD connector or 1x SD NAND on board      |
| USB        | 1 x Type-C for data and Power,1x USB2 solder pad         | 1 x Type-C for power and data, USB Pads available |
| CSI        | 1 x 16P FPC connector (MIPI CSI 2-lane)                  | 1 x 16P FPC connector (MIPI CSI 2-lane)           |
| Sensor Support| 4M @ 25fps                                            | 5M @ 30fps                                        |
| Ethernet   | 100Mbps ethernet with PHY                                | 100Mbps ethernet with PHY                         |
| Audio      | N/A                                                      | Via GPIO Pads                                     |
| GPIO       | up to 26 Pins available for general purpose I/O（GPIO）  | up to 26 Pins available for general purpose I/O（GPIO）  |
| Power      | 5V/1A                                                   | 5V/1A                                                   |
| OS Support | Buildroot, RTOS                                         | Buildroot, RTOS, Debian/Ubuntu                       |
| Size       | 21mm*51mm                                               | 21mm*51mm                                            |


# 特徴 

## プロセッサ
- 1GHzと700MHzのRISC-V C906プロセッサ
- CVITEK TPUを統合し、画像認識が可能。
- H.264/H.265ビデオエンコーディングをサポートし、最大で2880x1620@20fpsまで対応。
- 高解像度のCMOSセンサーと互換性があります。
- センサークロック用のプログラム可能な周波数出力。
- 画像最適化のための包括的なISP機能。
- OpenCVライブラリの一部がCVハードウェアアクセラレーションをサポート。
- 16ビットオーディオコーデック、内蔵マイク入力と出力機能。
- フレキシブルなネットワーク設定、1つのイーサネットPHYがあります。

## CSI-2 (MIPIシリアルカメラ)
- 2レーンのMIPIカメラ入力用の16ピンFPCインターフェースがあります。
- I2C、CLK、RST信号は1.8V電圧レベルで動作します。

## イーサネット
- Milk-V Duoには、100MbpsのPHYを備えたCV1800Bチップが含まれています。
- PHYは5ピンのはんだパッドに接続されています。
- イーサネットを使用する場合、外部トランスとRJ45ソケットが必要です。

## USB
- USB 2.0規格に準拠し、USB 1.1と下位互換性があります。
- 各種速度モード、ホスト/デバイス機能、転送プロトコルをサポート。
- USBハブを使用してインターフェースを拡張可能（最大127デバイスまで）。
- 省電力モード、HIDデバイスをサポート。
- 設定可能なソフトウェアを使用してUSBスレーブデバイスの機能を提供。
- ストレージメディアへのアクセス用のUSB Type-C。

## マイクロSD
- SDIO0はセキュアデジタルメモリ（SD 3.0）プロトコルと互換性があります。

## ピン 
- MilkV-Duoの40ピンソケットには、最大26個のGPIOピンがあり、内部デバイス（SDIO、I2C、PWM、SPI、J-TAG、UARTなど）にアクセスできます。
- 最大3x I2C
- 最大5x UART
- 最大1x SDIO1
- 最大1x SPI
- 最大2x ADC
- 最大7x PWM
- 最大1x RUN
- 最大1x JTAG  
![pinoutv1.2](/docs/duo/pinout.webp)


## 基板仕様
- 片面PCB
- 寸法: 51×21mm
- 厚さ: 1mm
- Type-Cポートは上端に配置
- 残りの端には両面のはんだパッド/スルーホールピン
- 表面実装モジュールまたはDual Inline Package（DIP）形式として使用可能
- 2.54mm（0.1インチ）ピッチグリッドに1mmの穴を持つ40本の主要なユーザーピン
- ユニバーサル基板とブレッドボードと互換性があります。

## クイックスタート
- 慣れている方は、下から直にダウンロードしてください。
初めての方はタブから始め方(Getting Started)を参照ください。
- また日本語訳に関して、大元である英語版を翻訳しております。上手く動かない場合は翻訳時のミスも疑い英語版をみてください。更新頻度は英語が早いです。
- [イメージのダウンロード](https://github.com/milkv-duo/duo-buildroot-sdk/releases)

- [SDKに関して](https://milkv.io/docs/duo/getting-started/buildroot-sdk)

## サポート
疑問がある場合こちらの世界中のメンバーが集まるコミュニティで相談してみてください。 [Milk-V Community Duo Category](https://community.milkv.io/c/duo/5).