---
sidebar_label: 'wiringX'
sidebar_position: 10
---

# 紹介
WireringX は、開発者が汎用的で統一された関数を使用してさまざまなプラットフォームの GPIO を制御できるようにするライブラリです。 WireringX を使用すると、WiringX がサポートするすべてのプラットフォームで同じコードがネイティブに実行されます。

この記事では、wiringX を使用して Duo 上でアプリケーションを開発する方法を次の 4 つのパートに分けて紹介します。

1. wiringX APIs
2. 基本的な使用方法のコードのデモ
3. WireringXに基づくアプリケーションコンパイル環境の構成
4. WireringX を使用して実装されたいくつかのデモとプロジェクトの紹介

WireringX の使用法にすでに慣れている場合は、サンプル コードを直接参照できます。 [duo-examples](https://github.com/milkv-duo/duo-examples)

Milk-V Duo のワイヤリングX ピン番号は、Duo のピン名の番号と一致しています。ただし、LED 制御ピンは 40 ピンの物理ピン配置では使用できず、その配線 X ピン番号は `25`です。

<div className='gpio_style'>

| wiringX | PIN NAME |              Pin#               |              Pin#                | PIN NAME | wiringX |
| ------- | -------- | :-----------------------------: | :------------------------------: | -------- | ------- |
| 0       | GP0      | <div className='green'>1</div>  |    <div className='red'>40</div> | VBUS     |         |
| 1       | GP1      | <div className='green'>2</div>  |    <div className='red'>39</div> | VSYS     |         |
|         | GND      | <div className='black'>3</div>  |  <div className='black'>38</div> | GND      |         |
| 2       | GP2      | <div className='green'>4</div>  | <div className='orange'>37</div> | 3V3_EN   |         |
| 3       | GP3      | <div className='green'>5</div>  |    <div className='red'>36</div> | 3V3(OUT) |         |
| 4       | GP4      | <div className='green'>6</div>  |   <div className='gray'>35</div> |          |         |
| 5       | GP5      | <div className='green'>7</div>  |   <div className='gray'>34</div> |          |         |
|         | GND      | <div className='black'>8</div>  |  <div className='black'>33</div> | GND      |         |
| 6       | GP6      | <div className='green'>9</div>  |  <div className='green'>32</div> | GP27     | 27      |
| 7       | GP7      | <div className='green'>10</div> |  <div className='green'>31</div> | GP26     | 26      |
| 8       | GP8      | <div className='green'>21</div> | <div className='orange'>30</div> | RUN      |         |
| 9       | GP9      | <div className='green'>12</div> |  <div className='green'>29</div> | GP22     | 22      |
|         | GND      | <div className='black'>13</div> |  <div className='black'>28</div> | GND      |         |
| 10      | GP10     | <div className='green'>14</div> |  <div className='green'>27</div> | GP21     | 21      |
| 11      | GP11     | <div className='green'>15</div> |  <div className='green'>26</div> | GP20     | 20      |
| 12      | GP12     | <div className='green'>16</div> |  <div className='green'>25</div> | GP19     | 19      |
| 13      | GP13     | <div className='green'>17</div> |  <div className='green'>24</div> | GP18     | 18      |
|         | GND      | <div className='black'>18</div> |  <div className='black'>23</div> | GND      |         |
| 14      | GP14     | <div className='green'>19</div> |  <div className='green'>22</div> | GP17     | 17      |
| 15      | GP15     | <div className='green'>20</div> |  <div className='green'>21</div> | GP16     | 16      |
|         |          | &nbsp;                          |                                  |          |         |
| 25      | GP25     | <div className='blue'>LED</div> |                                  |          |         |

</div>

Duo のピンの多くは多目的機能を備えていることに注意してください。 `WireringX`を使用して各ピンの機能を制御する場合、ピンの現在の状態を確認して、目的の機能と一致していることを確認することが重要です。そうでない場合は、`duo-pinmux`コマンドを使用して目的の機能に切り替えることができます。

詳細については、詳細な使用手順を参照してください。
[pinmux](https://milkv.io/docs/duo/application-development/pinmux)


## 1. wiringX APIs

### General

<details><summary>int wiringXSetup(char *name, ...)</summary>

  GPIO ピンを構成および管理するために WiringX ライブラリを初期化するための Duo の固定構文は次のとおりです。
  ```
  wiringXSetup("duo", NULL)
  ```

</details>


<details><summary>int wiringXValidGPIO(int pin)</summary>

  GPIO ピンが利用可能かどうかを確認する

</details>


<details><summary>void delayMicroseconds(unsigned int ms)</summary>

  何ミリ秒かの遅延

</details>


<details><summary>int wiringXGC(void)</summary>

  リソースを解放する
</details>


<details><summary>char *wiringXPlatform(void)</summary>

  プラットフォーム情報を返す

</details>


### GPIO

<details><summary>int pinMode(int pin, pinmode_t mode)</summary>

  指定したピンの動作モードを設定します。pin はピン番号で、モードは次のとおりです。

  - PINMODE_INPUT 入力モード
  - PINMODE_OUTPUT 出力モード
  - PINMODE_INTERRUPT 割り込みモード

</details>


<details><summary>int digitalRead(int pin)</summary>

  指定したピンの入力値を読み取り、戻り値は HIGH または LOW です

</details>


<details><summary>int digitalWrite(int pin, enum digital_value_t value)</summary>

  指定したピンの出力値を設定します。値は
  - HIGH high level
  - LOW low level

</details>


<details><summary>int waitForInterrupt(int pin, int ms)</summary>

  ミリ秒ミリ秒のタイムアウトで、ピンで割り込みが発生するのを待ちます。
  *この関数は廃止されました。配線XISRの使用をお勧めします*

</details>


<details><summary>int wiringXISR(int pin, enum isr_mode_t mode)</summary>

ピンを割り込みモードとして設定します。モードには複数のモードがあります。
- ISR_MODE_RISING
- ISR_MODE_FALLING
- ISR_MODE_BOTH

</details>


### I2C

<details><summary>int wiringXI2CSetup(const char *dev, int addr)</summary>

  I2C ノードと I2C アドレスを構成する

</details>


<details><summary>int wiringXI2CRead(int fd)</summary>

  1バイトのデータを読み取ります

</details>


<details><summary>int wiringXI2CReadReg8(int fd, int reg)</summary>

  reg レジスタから 1 バイトのデータを読み取ります

</details>


<details><summary>int wiringXI2CReadReg16(int fd, int reg)</summary>

  reg レジスタから 2 バイトのデータを読み取ります

</details>


<details><summary>int wiringXI2CWrite(int fd, int reg)</summary>

  レジスタ reg のアドレスを書き込みます

</details>


<details><summary>int wiringXI2CWriteReg8(int fd, int reg, int value8)</summary>

  8 ビットのデータ値 8 をレジスタ reg に書き込みます。

</details>


<details><summary>int wiringXI2CWriteReg16(int fd, int reg, int value16)</summary>

  16 ビットのデータ値 16 をレジスタ reg に書き込みます。

</details>

### SPI

<details><summary>int wiringXSPISetup(int channel, int speed)</summary>

  SPI デバイスのチャネル (Duo では 0) と速度 (デフォルトは 500000) を構成します。

</details>

<details><summary>int wiringXSPIDataRW(int channel, unsigned char *data, int len)</summary>

  SPI バスは立ち上がりエッジでデータを書き込み、立ち下がりエッジでデータを読み取ります。したがって、この関数は読み取りと書き込みの両方の操作を同時に実行し、その結果、読み取りデータが書き込みデータを上書きします。ご使用の際はご注意ください

</details>

<details><summary>int wiringXSPIGetFd(int channel)</summary>

  Duo ではチャネルのデフォルトが 0 である SPI デバイスのファイル記述子を取得します。

</details>

### UART

<details><summary>int wiringXSerialOpen(const char *dev, struct wiringXSerial_t serial)</summary>

  シリアル ポート デバイスを開きます。`dev`はデバイス記述子、`serial`はシリアル ポート関連のパラメータを入力する必要がある構造体です。詳細については、[UART の使用例](#UART-Usage-Example)を参照してください。

  ```c
  typedef struct wiringXSerial_t {
      unsigned int baud;           // baudrate
      unsigned int databits;       // 7/8
      unsigned int parity;         // o/e/n
      unsigned int stopbits;       // 1/2
      unsigned int flowcontrol;    // x/n
  } wiringXSerial_t;
  ```

</details>

<details><summary>void wiringXSerialClose(int fd)</summary>

  シリアルポートを閉じます

</details>

<details><summary>void wiringXSerialFlush(int fd)</summary>

  バッファをフラッシュします

</details>

<details><summary>void wiringXSerialPutChar(int fd, unsigned char c)</summary>

  文字を出力します

</details>

<details><summary>void wiringXSerialPuts(int fd, const char *s)</summary>

  文字列を出力する

</details>

<details><summary>void wiringXSerialPrintf(int fd, const char *message, ...)</summary>

  Format output

</details>

<details><summary>int wiringXSerialDataAvail(int fd)</summary>

  バッファ内で受信したデータの数を返します

</details>

<details><summary>int wiringXSerialGetChar(int fd)</summary>

  シリアルポートデバイスから文字を読み取ります

</details>

## 2. Code example

### GPIO の使用例

ここでは、GPIO を使用する例を示します。 Duo のピン 20 を 1 秒ごとに切り替えて、ハイにした後、ローにします。ピン 20 の物理ピン番号は、WiringX 番号付けシステムでは 15 です。

```c
#include <stdio.h>
#include <unistd.h>

#include <wiringx.h>

int main() {
    int DUO_GPIO = 15;

    if(wiringXSetup("duo", NULL) == -1) {
        wiringXGC();
        return -1;
    }

    if(wiringXValidGPIO(DUO_GPIO) != 0) {
        printf("Invalid GPIO %d\n", DUO_GPIO);
    }

    pinMode(DUO_GPIO, PINMODE_OUTPUT);

    while(1) {
        printf("Duo GPIO (wiringX) %d: High\n", DUO_GPIO);
        digitalWrite(DUO_GPIO, HIGH);
        sleep(1);
        printf("Duo GPIO (wiringX) %d: Low\n", DUO_GPIO);
        digitalWrite(DUO_GPIO, LOW);
        sleep(1);
    }

    return 0;
}
```
Duo でコンパイルして実行した後、マルチメータまたはオシロスコープを使用してピン 20 の状態を測定し、それが予想される動作と一致するかどうかを確認できます。


Duo でコンパイルして実行した後、マルチメータまたはオシロスコープを使用してピン 20 の状態を測定し、それが予想される動作と一致するかどうかを確認できます。

オンボード LED ピンを使用して動作を確認することもできます。 LEDのオン/オフ状態を観察することで、プログラムが正しく実行されているかどうかを直感的に判断できます。 LED の WiringX ピン番号は `25`です。ピン`15`をピン `25`に置き換えて、上記のコードを変更するだけです。ただし、デフォルトのファームウェアには起動時に LED の点滅を制御するスクリプトが含まれており、これを無効にする必要があることに注意してください。無効にする方法については、以下の[blink](#blink)の説明例を参照してください。


### I2Cの使用例

ここではI2Cとの通信の仕方を説明します。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdint.h>

#include <wiringx.h>

#define I2C_DEV "/dev/i2c-1"

#define I2C_ADDR 0x04

int main(void)
{
    int fd_i2c;
    int data = 0;

    if(wiringXSetup("duo", NULL) == -1) {
        wiringXGC();
        return -1;
    }

    if ((fd_i2c = wiringXI2CSetup(I2C_DEV, I2C_ADDR)) <0) {
        printf("I2C Setup failed: %d\n", fd_i2c);
        wiringXGC();
        return -1;
    }

    // TODO
}
```

### SPIの使用例

ここではSPIとの通信の仕方を説明します。

```c
#include <stdio.h>
#include <unistd.h>
#include <stdint.h>

#include <wiringx.h>

int main(void)
{
    int fd_spi;

    if(wiringXSetup("duo", NULL) == -1) {
        wiringXGC();
        return -1;
    }

    if ((fd_spi = wiringXSPISetup(0, 500000)) <0) {
        printf("SPI Setup failed: %d\n", fd_spi);
        wiringXGC();
        return -1;
    }

    // TODO
}
```

### UARTの使用例

以下は、ピン 4/5 で UART4 を使用した UART 通信の例です。

```c
#include <stdio.h>
#include <unistd.h>

#include <wiringx.h>

int main() {
    struct wiringXSerial_t wiringXSerial = {115200, 8, 'n', 1, 'n'};
    char buf[1024];
    int str_len = 0;
    int i;
    int fd;

    if(wiringXSetup("duo", NULL) == -1) {
        wiringXGC();
        return -1;
    }

    if ((fd = wiringXSerialOpen("/dev/ttyS4", wiringXSerial)) < 0) {
        printf("Open serial device failed: %d\n", fd);
        wiringXGC();
        return -1;
    }

    wiringXSerialPuts(fd, "Duo Serial Test\n");

    while(1)
    {
        str_len = wiringXSerialDataAvail(fd);
        if (str_len > 0) {
            i = 0;
            while (str_len--)
            {
                buf[i++] = wiringXSerialGetChar(fd);
            }
            printf("Duo UART receive: %s\n", buf);
        }
    }

    wiringXSerialClose(fd);

    return 0;
}
```
テスト方法:

USB - シリアル変換ケーブルの RX は Duo のピン 4 (UART4_TX) に接続され、シリアル ケーブルの TX は Duo のピン 5 (UART4_RX) に接続され、シリアル ケーブルの GND は Duo の GND に接続されます。デュオ。コンピュータ上で、シリアル デバッグ ツールを使用して COM ポートとパラメータを設定します。

上記プログラムのコンパイルされた実行可能ファイルの名前は `uart_test`です。 SSH 経由で Duo にアップロードして実行すると、コンピューター上のシリアル ツールで受信した文字列`Duo Serial Test`が表示されるはずです。シリアル ツールから文字列`Hello World`を送信すると、Duo の端末で受信された対応する文字列も表示されます。これにより、シリアル通信が正しく機能していることが確認されます。


![duo](/docs/duo/duo-wiringx-uart-test.png)


## 3. 開発環境のセットアップ

### Ubuntu20.04で環境を構築する

仮想マシンにインストールされた Ubuntu、Windows 上の WSL 経由でインストールされた Ubuntu、または Docker を使用した Ubuntu ベースのシステムを使用することもできます。 

- 依存関係をコンパイルするツールをインストールします。
  ```
  sudo apt-get install wget git make
  ```
- サンプルソースを取得します。
  ```
  git clone https://github.com/milkv-duo/duo-examples.git
  ```

- コンパイル環境を準備する
  ```
  cd duo-examples
  source envsetup.sh
  ```
  初めてソースするときは、必要な SDK パッケージが自動的にダウンロードされます。サイズは約 180MB です。ダウンロードが完了すると、`duo-sdk`という名前で、`duo-examples`ディレクトリに自動的に抽出されます。次回ソースするときに、ディレクトリがすでに存在する場合、再度ダウンロードされることはありません。 

- コンパイルテスト  

  hello-world を例として、hello-world ディレクトリに入り、make を実行します。
  ```
  cd hello-world
  make
  ```
  コンパイルが成功したら、生成された `helloworld`実行可能プログラムをネットワーク ポートまたは RNDIS ネットワーク経由で Duo デバイスに送信します。たとえば、 [default firmware](https://github.com/milkv-duo/duo-buildroot-sdk/releases)でサポートされている RNDIS 方式、Duo の IP は 192.168.42.1、ユーザー名は`root`、パスワードは`milkv`です。
  ```
  scp helloworld root@192.168.42.1:/root/
  ```
  送信が成功したら、ssh またはシリアル ポート経由でログインしたターミナルで `./helloworld`を実行すると、`Hello, World!`が表示されます。
  ```
  [root@milkv]~# ./helloworld
  Hello, World!
  ```
  **この時点で、コンパイルおよび開発環境を使用する準備が整いました。 **

### 独自のプロジェクトを作成する方法

既存の例をコピーして、必要な変更を加えることができます。 たとえば、GPIO を操作する必要がある場合は、`blink`の例を参照できます。 LED の点滅は、GPIO の電圧レベルを制御することによって実現されます。 現在の SDK は、GPIO 操作に WiringX ライブラリを利用しており、これは Duo に特化して調整されています。 プラットフォームの初期化と GPIO 制御メソッドは、参照用の`blink.c`コードにあります。

- `my-project` という名前の独自のプロジェクト ディレクトリを作成します。
- `blink.c` ファイルと `Makefile` ファイルを `blink` サンプルから `my-project` ディレクトリにコピーします。
- 「blink.c」の名前を、「gpio_test.c」などの希望の名前に変更します。
- 「TARGET=blink」を「TARGET=gpio_test」に変更して、「Makefile」を変更します。
- `gpio_test.c` を変更して、独自のコード ロジックを実装します。
- makeコマンドを実行してコンパイルします。
- 生成された「gpio_test」実行可能プログラムを Duo に送信して実行します。
  
Note:
- 新しいプロジェクト ディレクトリの作成は、`duo-examples` ディレクトリ内に配置する必要はありません。 好みに応じて任意の場所を選択できます。 `make` コンパイル コマンドを実行する前に、`duo-examples` ディレクトリ (source /PATH/TO/duo-examples/envsetup.sh) からコンパイル環境をロードするだけで十分です。
- コンパイル環境 (envsetup.sh) がロードされるターミナル内では、ARM や X86 などの他のプラットフォーム用の Makefile プロジェクトをコンパイルしないでください。 他のプラットフォーム用にプロジェクトをコンパイルする必要がある場合は、新しいターミナルを開きます。

## 4. 各例の説明 hello-world

### [hello-world](https://github.com/milkv-duo/duo-examples/tree/main/hello-world)

Duoの周辺機器とは対話せず、出力"Hello, World!"を印刷するだけのシンプルな例です。これは開発環境を確認するためのものです。

### [blink](https://github.com/milkv-duo/duo-examples/tree/main/blink)

この例では、GPIOピンに接続されたLEDを制御する方法を示しています。WiringXライブラリを使用してGPIOピンの電圧レベルを切り替え、LEDが点滅するようにします。 

`blink.c`コードには、WiringX ライブラリからのプラットフォームの初期化および GPIO 操作メソッドが含まれています。

LED の点滅を含む`blink`の例をテストするには、Duo のデフォルトのファームウェアで LED の自動点滅を担当するスクリプトを無効にする必要があります。 Duo ターミナルで次のコマンドを実行します。
```
mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
```
このコマンドは、LED 点滅スクリプトの名前を変更します。 Duo を再起動すると、LED は点滅しなくなります。

C で実装された`blink`プログラムのテストが完了したら、LED 点滅スクリプトを復元したい場合は、次のコマンドを使用して名前を元に戻し、Duo を再起動します。
```
mv /mnt/system/blink.sh_backup /mnt/system/blink.sh && sync
```

### [I2C](https://github.com/milkv-duo/duo-examples/tree/main/i2c)

#### [bmp280_i2c](https://github.com/milkv-duo/duo-examples/tree/main/i2c/bmp280_i2c)

このコード例は、Milk-V Duo を Bosch 製の人気のある BMP280 温度および気圧センサーとインターフェースする方法を示しています。

