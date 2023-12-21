---
sidebar_label: 'センサーデモ'
sidebar_position: 40
---
# センサーデモ
## DHT22

### ハード情報

#### Duo 開発ボードのピン

GitHub Link: https://github.com/milkv-duo/duo-files/blob/main/hardware/duo/duo-schematic-v1.2.pdf

![](/docs/duo/sensor-demo/1.jpg)

#### DHT22 温湿度センサー

dfrobotwiki Link: https://wiki.dfrobot.com.cn/\_SKU_SEN0137\_%E9%AB%98%E7%B2%BE%E5%BA%A6DHT22%E6%B8%A9%E6%B9%BF%E5%BA%A6%E4%BC%A0%E6%84%9F%E5%99%A8

DHT22 温湿度センサは、環境温度と湿度を測定するための多目的でコスト効率の良いセンサです。デジタル信号出力に基づいており、温度解像度は 0.1℃、湿度解像度は 0.1% という高精度な測定を提供します。センサは、湿度を測定するための容量性湿度感知要素と、温度を測定するためのサーミスタを使用しています。DHT22 センサは比較的低消費電力で、3.3V から 5V の電圧範囲で動作するため、バッテリー駆動のプロジェクトに適しています。さらに、センサは長期的な安定性と高い信頼性を提供し、HVAC システム、気象ステーション、室内空気品質監視システムなど、さまざまなアプリケーションに理想的な選択肢となります。

#### センサモジュール

DHT22 の裸のセンサだと4つのピンを持っていますが、DHT22 モジュールだと合計3つのピンを持ちます。3 つのピンの機能は、2 つは電源ピンで、1 つはデータピンです。4 ピンあるセンサでは、追加のピンは特定の機能を持たないNC（No Connection）ピンです。モジュールとセンサの両方のピン配置は次のとおりです：
![](/docs/duo/sensor-demo/DHT22_10.png)

\- DATA: 1-Wire通信のためのデータピン。

\- GND: モジュールのグラウンドピン。

\- VCC: モジュールの電源ピン。

\- Not Used: このピンはこのセンサでは使用されていません。

#### DHT22 センサモジュールコンポーネントマーキング

センサ以外に、PCB 上の DHT22 モジュールにはプルアップ抵抗とデカップリングコンデンサのみが含まれています。DHT22 モジュールのコンポーネントマーキングは次のとおりです。

![](/docs/duo/sensor-demo/DHT22_11.png)

#### DHT22 モジュール回路図

DHT22 温湿度センサモジュールの完全な回路図は、以下の図に示されています。

![](/docs/duo/sensor-demo/DHT22_12.png)

DHT22 モジュールの回路図は上記のようになっています。前述の通り、ボードにはほんの少数のコンポーネントしかありません。VCC ピンと GND ピンはDHT22 センサに直接接続されており、プルアップ抵抗は DATA ピンに接続されています。タンタルと多層コンデンサは十分なフィルタリングを提供します。一部の PCB では、電源インジケータとして LED インジケータを見つけることができますが、ほとんどの回路板ではLEDは存在しません。

#### DHT22 センサモジュールについてのよくある質問

Q: 簡単に言うと、DHT22 とは何ですか？

A: DHT22 は、より良い仕様を持つ DHT11 センサのより高価なバージョンです。その温度測定範囲は -40 から +125℃ で、精度は ±0.5 度です。一方、DHT11 の温度範囲は 0 から 50℃ で、精度は ±2 度です。

Q: DHT22 はアナログですか、デジタルですか？

A: DHT-22（別名AM2302）は、デジタル出力の相対湿度と温度センサです。

Q: DHT2 2は防水ですか？

A: いいえ、防水ではありません。

Q: DHT11 センサのサンプリングレートは？

A: DHT22 のサンプリングレートは 1Hz です。

Q: DHT22 はどのプロトコルを使用していますか？

A: DHT22 センサは、タイムドパルスを通じてデータを送受信する独自のシングルワイヤ通信プロトコルを使用しています。

#### DHT22 はどのように動作しますか？

オリジナルの DHT22 センサを使用している場合、それは NTC サーミスタとセンサモジュールを含んでいます。しかし、市場で入手できるほとんどのセンサは非オリジナルの部品で、下の画像に示すような小さなセンサを含んでいます。

![](/docs/duo/sensor-demo/DHT22_13.png)

湿度検知素子は、2 つの電極の間に挟まれた吸湿性基板で構成されています。基板が湿気を吸収すると、2 つの電極間の抵抗が減少します。電極間の抵抗変化は相対湿度に比例します。相対湿度が高くなると電極間の抵抗は減少し、相対湿度が低くなると抵抗は増加します。この抵抗変化はオンボード MCU の ADC によって測定され、相対湿度の計算に使用されます。

![](/docs/duo/sensor-demo/DHT22_14.png)

各 DHT22 コンポーネントは、非常に正確な湿度校正を使用して、実験室で厳密な校正を受けます。校正係数は、センサーの内部信号検出プロセスで使用するために OTP メモリにプログラムとして保存されます。

#### DHT22 単線通信プロトコル

DHT22 とマイクロコントローラーとの通信には単線通信プロトコルが使用されます。データのサンプリングが完了するまでに約 4 ミリ秒かかります。データは小数部分と整数部分の両方で構成され、MSB 形式で合計 40 ビットの長さになります。データ形式は次のとおりです: 8 ビット整数 RH データ + 8 ビット 10 進数 RH データ + 8 ビット整数 T データ + 8 ビット 10 進数 T データ + 8 ビット チェックサム。データ送信が正しい場合、チェックサムは「8 ビット整数 RH データ + 8 ビット 10 進数 RH データ + 8 ビット整数 T データ + 8 ビット 10 進数 T データ」の最後の 8 ビットであるはずです。

MCU が開始信号を送信すると、DHT は低電力モードから実行モードに変わり、40 ビットのデータすべてをマイクロコントローラーにダンプします。マイクロコントローラーはデータを読み取り、バイナリ データに基づいて温度と湿度を計算します。

![](/docs/duo/sensor-demo/DHT22_15.png)

上の画像は、マイクロコントローラーと DHT22 によるデータ通信がどのように機能するかを示しています。

#### 開発ボードへの接続

DHT22: 赤色のワイヤを 3V3 (OUT) に、黒色のワイヤをアースに、緑色のワイヤを GPIOA15 に接続します。

回路図は次のとおりです。 黒丸は DHT22 を表します。

![](/docs/duo/sensor-demo/5.jpg)

DHT22 は次のように接続する必要があります。

![](/docs/duo/sensor-demo/DHT22_16.jpg)

### Example Code:
GitHub link: https://github.com/milkv-duo/duo-examples

**dht22.c：**
```
// Ref: https://github.com/technion/lol_dht22/blob/master/dht22.c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <wiringx.h>
#define MAXTIMINGS 85
static int DHTPIN = 15;
static int dht22_dat[5] = {0, 0, 0, 0, 0};
static uint8_t sizecvt(const int read)
{
    /* digitalRead() and friends from wiringpi are defined as returning a value
    < 256. However, they are returned as int() types. This is a safety function */

    if (read > 255 || read < 0)
    {
        printf("Invalid data from wiringPi library\n");
        exit(EXIT_FAILURE);
    }
    return (uint8_t)read;
}

static int read_dht22_dat()
{
    uint8_t laststate = HIGH;
    uint8_t counter = 0;
    uint8_t j = 0, i;

    dht22_dat[0] = dht22_dat[1] = dht22_dat[2] = dht22_dat[3] = dht22_dat[4] = 0;

    // pull pin down for 18 milliseconds
    pinMode(DHTPIN, PINMODE_OUTPUT);
    digitalWrite(DHTPIN, HIGH);
    // delay(500);
    delayMicroseconds(500000);
    digitalWrite(DHTPIN, LOW);
    // delay(20);
    delayMicroseconds(20000);
    // prepare to read the pin
    pinMode(DHTPIN, PINMODE_INPUT);

    // detect change and read data
    for (i = 0; i < MAXTIMINGS; i++)
    {
        counter = 0;
        while (sizecvt(digitalRead(DHTPIN)) == laststate)
        {
            counter++;
            delayMicroseconds(2);
            if (counter == 255)
            {
                break;
            }
        }
        laststate = sizecvt(digitalRead(DHTPIN));

        if (counter == 255)
            break;

        // ignore first 3 transitions
        if ((i >= 4) && (i % 2 == 0))
        {
            // shove each bit into the storage bytes
            dht22_dat[j / 8] <<= 1;
            if (counter > 16)
                dht22_dat[j / 8] |= 1;
            j++;
        }
    }

    // check we read 40 bits (8bit x 5 ) + verify checksum in the last byte
    // print it out if data is good
    if ((j >= 40) &&
        (dht22_dat[4] == ((dht22_dat[0] + dht22_dat[1] + dht22_dat[2] + dht22_dat[3]) & 0xFF)))
    {
        float t, h;
        h = (float)dht22_dat[0] * 256 + (float)dht22_dat[1];
        h /= 10;
        t = (float)(dht22_dat[2] & 0x7F) * 256 + (float)dht22_dat[3];
        t /= 10.0;
        if ((dht22_dat[2] & 0x80) != 0)
            t *= -1;
        printf("Humidity = %.2f %% Temperature = %.2f *C \n", h, t);
        return 1;
    }
    else
    {
        printf("Data not good, skip\n");
        return 0;
    }
}

int main()
{
    if (wiringXSetup("duo", NULL) == -1)
    {
        wiringXGC();
        return -1;
    }

    if (wiringXValidGPIO(DHTPIN) != 0)
    {
        printf("Invalid GPIO %d\n", DHTPIN);
    }

    while (1)
    {
        read_dht22_dat();
        delayMicroseconds(1500000);
    }

    return 0;
}

```

**Makefile：**
```
TARGET=dht22

ifeq (,$(TOOLCHAIN_PREFIX))
$(error TOOLCHAIN_PREFIX is not set)
endif

ifeq (,$(CFLAGS))
$(error CFLAGS is not set)
endif

ifeq (,$(LDFLAGS))
$(error LDFLAGS is not set)
endif

CC = $(TOOLCHAIN_PREFIX)gcc
CFLAGS += -I$(SYSROOT)/usr/include
LDFLAGS += -L$(SYSROOT)/lib
LDFLAGS += -L$(SYSROOT)/usr/lib
LDFLAGS += -lwiringx

SOURCE = $(wildcard *.c)
OBJS = $(patsubst %.c,%.o,$(SOURCE))

$(TARGET): $(OBJS)
	$(CC) -o $@ $(OBJS) $(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -o $@ -c $<

.PHONY: clean
clean:
	@rm *.o -rf
	@rm $(OBJS) -rf
	@rm $(TARGET)

```
### Ubuntu20.04 で環境構築

仮想マシンにインストールされた Ubuntu、Windows 上の WSL 経由でインストールされた Ubuntu、または Docker を使用した Ubuntu ベースのシステムを使用することもできます。


- 依存関係をコンパイルするツールをインストールします。
  ```
  sudo apt-get install wget git make
  ```
- ソースコードの例を取得する
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
  コンパイルが成功したら、生成された `helloworld`実行可能プログラムをネットワーク ポートまたは RNDIS ネットワーク経由で Duo デバイスに送信します。たとえば、[default firmware](https://github.com/milkv-duo/duo-buildroot-sdk/releases)でサポートされている RNDIS 方式、Duo の IP は 192.168.42.1、ユーザー名は`root`、パスワードは `milkv`です。
  ```
  scp helloworld root@192.168.42.1:/root/
  ```
送信が成功したら、ssh またはシリアル ポート経由でログインしたターミナルで ./helloworld を実行すると、`Hello, World!`が表示されます。
  ```
  [root@milkv]~# ./helloworld
  Hello, World!
  ```
  **この時点で、コンパイルおよび開発環境を使用する準備が整いました。**
### 操作手順

![](/docs/duo/sensor-demo/DF9GMS_7.png)

次にコンパイルします。 dht22 を例として、例のディレクトリに入り、単に make を実行します。

```
cd dht22
make it
```
エラーレポートを作成し、それをソースします。コンパイルが成功すると、df9gms 実行可能プログラムが得られます。以下の図に示すように。

![](/docs/duo/sensor-demo/DHT22_17.png)


次に、df9gms を開発ボードのルートパスにアップロードし、```./dht22```と入力して実行します。正常に実行された場合のスクリーンショットを以下に示します。


![](/docs/duo/sensor-demo/DHT22_18.png)

## モーターDF9GMS 180°の操作のやり方

### ハード情報

#### Duo development board pin

GitHub：https://github.com/milkv-duo/duo-files/blob/main/hardware/duo/duo-schematic-v1.2.pdf

![](/docs/duo/sensor-demo/1.jpg)

#### DF9GMS 180°

![](/docs/duo/sensor-demo/DF9GMS_2.png)

DFRobot のマイクロサーボ DF9GMS 、このサーボは、内部高精度ナイロンギアセット、精密制御回路、ハイエンド軽量中空カップモーターを備えた高強度 ABS 透明ケースを備えており、このミニサーボの重量はわずか 9g です。出力トルクは驚異の 1.6kg/cm に達します。 

モーター技術仕様:  
動作電圧: 4.8V  
トルク: 1.6kg/cm (4.8V)  
スピード: 0.14 seconds/60 degrees (4.8V)  
動作温度: -30 to +60 degrees Celsius  
デッドバンド 幅: 0.5 ミリセカンド  
物理的なサイズ: 23x12.2x29mm  
重さ: 9g

#### DF9GMS マイクロサーボの構成と動作原理

![](/docs/duo/sensor-demo/DF9GMS_3.png)

#### 接続図:
• ハードウェア  
o 1 x Arduino UNO 制御ボード  
o 1 x DF9GMS マイクロサーボ  
o 幾つかの接続用ケーブル  
o グレー - GND, 赤 - VCC, 黄色 - signal line

![](/docs/duo/sensor-demo/DF9GMS_4.png)

#### 開発ボードへのつなぎ方

DF9GMS: 赤色のワイヤは VSYS に接続され、茶色のワイヤはグランドに接続され、オレンジ色のワイヤは GPIO19 に接続されます。
以下に回路図を示します。紫色の丸は DF9GMS を表します。
![](/docs/duo/sensor-demo/5.jpg)

DF9GMS は次のように接続する必要があります。

![](/docs/duo/sensor-demo/DF9GMS_6.jpg)

### サンプルコード: 
GitHub link: https://github.com/milkv-duo/duo-examples

**df9gms.c:**  
```
#include <stdio.h>
#include <unistd.h>
#include <wiringx.h>
/*
Duo
------------------------------------------
PWM operation at a fixed frequency clock of 100MHz, writing Period in units of nanoseconds.
	DF9GMS 360-degree PWM Duty Cycle 
	------------------------------------------ 
	0.4ms - 1.4ms CW deceleration 
	1.5ms Stop 
	1.6ms - 3ms CCW acceleration 
*/
static int PWM_PIN = 4; // PWM5@GP4

int main()
{
    long i;

    if(wiringXSetup("duo", NULL) == -1) {
        wiringXGC();
        return -1;
    }

    wiringXSetPWMPeriod(PWM_PIN, 20000000);  // 20ms
    wiringXSetPWMDuty(PWM_PIN, 1500000);     // 1.5ms stop
    wiringXSetPWMPolarity(PWM_PIN, 0);       // 0-normal, 1-inversed
    wiringXPWMEnable(PWM_PIN, 1);            // 1-enable, 0-disable

    delayMicroseconds(1000000); // 1s

    for (i = 10000; i< 3000000; i += 10000) // 10 us 
    {
        wiringXSetPWMDuty(PWM_PIN, i);
        printf("Duty: %ld\n", i);
        delayMicroseconds(50000); // 50ms
    }

    wiringXSetPWMDuty(PWM_PIN, 1500000);    // 1.5ms stop

    return 0;
}

```

**Makefile：**

```
TARGET=df9gms

ifeq (,$(TOOLCHAIN_PREFIX))
$(error TOOLCHAIN_PREFIX is not set)
endif

ifeq (,$(CFLAGS))
$(error CFLAGS is not set)
endif

ifeq (,$(LDFLAGS))
$(error LDFLAGS is not set)
endif

CC = $(TOOLCHAIN_PREFIX)gcc

CFLAGS += -I$(SYSROOT)/usr/include

LDFLAGS += -L$(SYSROOT)/lib
LDFLAGS += -L$(SYSROOT)/usr/lib
LDFLAGS += -lwiringx

SOURCE = $(wildcard *.c)
OBJS = $(patsubst %.c,%.o,$(SOURCE))

$(TARGET): $(OBJS)
	$(CC) -o $@ $(OBJS) $(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -o $@ -c $<

.PHONY: clean
clean:
	@rm *.o -rf
	@rm $(OBJS) -rf
	@rm $(TARGET)
```

### Ubuntu20.04での開発環境

仮想マシンにインストールしたUbuntu、WindowsのWSL経由でインストールしたUbuntu、またはDockerを使用したUbuntuベースのシステムも使用できます。

- コンパイル依存関係のツールをインストールします。
  ```
  sudo apt-get install wget git make
  ```
- サンプルソースコードを取得します
  ```
  git clone https://github.com/milkv-duo/duo-examples.git
  ```

- コンパイル環境を準備します
  ```
  cd duo-examples
  source envsetup.sh
  ```
  最初にソースを指定すると、必要な SDK パッケージが自動的にダウンロードされます。ダウンロードされたものは、約 180MB のサイズです。ダウンロードが完了すると、`duo-examples`ディレクトリに`duo-sdk`という名前で自動的に展開されます。次回以降ソースを指定すると、そのディレクトリがすでに存在する場合、再度ダウンロードは行われません。

- コンパイルテスト

  hello-worldを例に、hello-worldディレクトリに入り、makeを実行します
  ```
  cd hello-world
  make
  ```
 
  コンパイルが成功すると、生成された `helloworld` 実行可能プログラムをネットワークポートまたは RNDIS ネットワークを通じて Duo デバイスに送信します。例えば、[default firmware](https://github.com/milkv-duo/duo-buildroot-sdk/releases)でサポートされている RNDIS を使う場合、Duo の IP は 192.168.42.1、ユーザ名は`root`、パスワードは`milkv`です

  ```
  scp helloworld root@192.168.42.1:/root/
  ```
  送信が成功したら、ssh またはシリアル ポート経由でログインしたターミナルで ./helloworld を実行すると、`Hello, World!`が表示されます。
  ```
  [root@milkv]~# ./helloworld
  Hello, World!
  ```
  **この時点で、コンパイルおよび開発環境を使用する準備が整いました。**

### 操作手順

![](/docs/duo/sensor-demo/DF9GMS_7.png)

次にコンパイルします。 df9gms を例として、サンプルのディレクトリに入り、単に make を実行します。
```
cd df9gms
make it
```
エラーレポートを作成し、それをソースします。コンパイルが成功すると、df9gms 実行可能プログラムが得られます。以下の図に示すように。


![](/docs/duo/sensor-demo/DF9GMS_8.png)


次に、df9gms を開発ボードのルート パスにアップロードし、```./df9gms```と入力して実行します。正常に実行された場合のスクリーンショットを以下に示します。

![](/docs/duo/sensor-demo/DF9GMS_9.png)

