---
sidebar_label: 'RTOSコアについて'
sidebar_position: 50
---

# 紹介

DuoのCPUはデュアルコア設計で、大コアではLinuxシステムが動作し、小コアではリアルタイムシステムが動作しています。現在はFreeRTOSが動作しています。

# 小コアの使い方

## コア間通信の例
Duoの大コアと小コアの間の通信は、メールボックスのモジュールを通じて実現されています。最新のイメージでは、大コアのLinuxカーネルにメールボックスドライバが追加され、小コアのFreeRTOSコードにも関連機能が実装されています。テストには[V1.0.6](https://github.com/milkv-duo/duo-buildroot-sdk/releases/tag/Duo-V1.0.6)または[更新されたイメージ ](https://github.com/milkv-duo/duo-buildroot-sdk/releases) を使用してください。


### 大コアで小コアを操作しLEDを光らせる

この例は、大きなコア上で動作するLinuxアプリケーションで、大きなコアのLinuxカーネル内のメールボックスドライバを通じて、小さなコアのFreeRTOSがDuo上の青色LEDを先に点灯させ、3秒後に消灯させるように通知します。

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <unistd.h>

enum SYSTEM_CMD_TYPE {
	CMDQU_SEND = 1,
	CMDQU_SEND_WAIT,
	CMDQU_SEND_WAKEUP,
};

#define RTOS_CMDQU_DEV_NAME "/dev/cvi-rtos-cmdqu"
#define RTOS_CMDQU_SEND                         _IOW('r', CMDQU_SEND, unsigned long)
#define RTOS_CMDQU_SEND_WAIT                    _IOW('r', CMDQU_SEND_WAIT, unsigned long)
#define RTOS_CMDQU_SEND_WAKEUP                  _IOW('r', CMDQU_SEND_WAKEUP, unsigned long)

enum SYS_CMD_ID {
    CMD_TEST_A  = 0x10,
    CMD_TEST_B,
    CMD_TEST_C,
    CMD_DUO_LED,
    SYS_CMD_INFO_LIMIT,
};

enum DUO_LED_STATUS {
	DUO_LED_ON	= 0x02,
	DUO_LED_OFF,
    DUO_LED_DONE,
};

struct valid_t {
	unsigned char linux_valid;
	unsigned char rtos_valid;
} __attribute__((packed));

typedef union resv_t {
	struct valid_t valid;
	unsigned short mstime; // 0 : noblock, -1 : block infinite
} resv_t;

typedef struct cmdqu_t cmdqu_t;
/* cmdqu size should be 8 bytes because of mailbox buffer size */
struct cmdqu_t {
	unsigned char ip_id;
	unsigned char cmd_id : 7;
	unsigned char block : 1;
	union resv_t resv;
	unsigned int  param_ptr;
} __attribute__((packed)) __attribute__((aligned(0x8)));

int main()
{
    int ret = 0;
    int fd = open(RTOS_CMDQU_DEV_NAME, O_RDWR);
    if(fd <= 0)
    {
        printf("open failed! fd = %d\n", fd);
        return 0;
    }

    struct cmdqu_t cmd = {0};
    cmd.ip_id = 0;
    cmd.cmd_id = CMD_DUO_LED;
    cmd.resv.mstime = 100;
    cmd.param_ptr = DUO_LED_ON;

    ret = ioctl(fd , RTOS_CMDQU_SEND_WAIT, &cmd);
    if(ret < 0)
    {
        printf("ioctl error!\n");
        close(fd);
    }
    sleep(1);
    printf("C906B: cmd.param_ptr = 0x%x\n", cmd.param_ptr);

    sleep(3);

    cmd.cmd_id = CMD_DUO_LED;
    cmd.param_ptr = DUO_LED_OFF;
    ret = ioctl(fd , RTOS_CMDQU_SEND, &cmd);
    if(ret < 0)
    {
        printf("ioctl error!\n");
        close(fd);
    }
    sleep(1);
    printf("C906B: cmd.param_ptr = 0x%x\n", cmd.param_ptr);

    close(fd);
    return 0;
}
```

このテストプログラムはすでにduo-examplesリポジトリにあります。 [duo-examples](https://github.com/milkv-duo/duo-examples/tree/main/mailbox-test) このリポジトリを初めて使用する場合は、READMEを参考に環境を設定してコンパイルを完了できます。 [README](https://github.com/milkv-duo/duo-examples/blob/main/README-zh.md) :

1. 推奨されるコンパイル環境`Ubuntu 22.04 LTS`
2. 必要なツールをインストールします:
   ```
   sudo apt-get install wget git make
   ``` 
3. 必要なツールをインストールします[duo-examples](https://github.com/milkv-duo/duo-examples)：
   ```
   git clone https://github.com/milkv-duo/duo-examples.git --depth=1
   ```
4. コンパイル環境をロードします：
   ```
   cd duo-examples
   source envsetup.sh
   ```
5.  `mailbox-test` ディレクトリに移動してコンパイルします
   ```
   cd mailbox-test
   make
   ```


コンパイルが成功すると、生成された`mailbox_test`  テストプログラムを、EthernetやUSBネットワーク(RNDIS)などの方法でDuoデバイスに転送します。例えば、USBネットワークの場合、DuoのIPは`192.168.42.1`、ユーザ名は  `root`、パスワードは `milkv`です。
```
$ scp mailbox_test root@192.168.42.1:/root/
```

Duo上で `mailbox_test` プログラムに実行権限を追加します:
```
chmod +x mailbox_test
```

Duoのデフォルトのファームウェアでは、大きなコアのLinuxシステムがLEDを制御して点滅させます。これは、起動スクリプトを通じて実現されています。このプログラムをテストする際には、LEDの点滅スクリプトを無効にする必要があります。Duoの端末で以下のコマンドを実行します:

```
mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
```


このコマンドは、LED 点滅スクリプトの名前を変更します。 Duo を再起動すると、LED は点滅しなくなります。

Duoのシリアル端末で `./mailbox_test`  を実行してテストします。出力は以下の通りです:
```
[root@milkv-duo]~# ./mailbox_test 
RT: [507.950049]prvQueueISR
RT: [507.952485]recv cmd(19) from C906B, param_ptr [0x00000002]
RT: [507.958306]recv cmd(19) from C906B...send [0x00000004] to C906B
C906B: cmd.param_ptr = 0x4
RT: [511.965433]prvQueueISR
RT: [511.967867]recv cmd(19) from C906B, param_ptr [0x00000003]
RT: [511.973689]recv cmd(19) from C906B...send [0x00000004] to C906B
C906B: cmd.param_ptr = 0x3
```

Duoの上の青色LEDが先に点灯し、その後消灯する現象が観察できます。

`RT`で始まるログは小さなコアの `FreeRTOS` が出力したもので、`C906B`で始まるログは大きなコア上の`mailbox_test`  アプリケーションが出力したものです。

ログを見ると、大きなコアが小コアに点灯指示を送信した後、小コアから大コアに0x4(0x00000004)が返送され、大コアも0x4を受け取ったことがわかります。また、消灯指示を送信した後にも、小コアから大コアに0x4(0x00000004)が返送されますが、大コア端で印刷される値は0x3です。これは、消灯指示を送信する前のパラメータで、`RTOS_CMDQU_SEND_WAIT` と`RTOS_CMDQU_SEND`の二つのパラメータの違いを示しています:
```
RTOS_CMDQU_SEND_WAIT  戻り値を待つ
RTOS_CMDQU_SEND       戻り値はない
```

また、ログにはテストプログラムと小さなコアのログしか表示されていませんが、大きなコアのLinuxカーネル内のメールボックスドライバの関連ログを表示するには、以下の二つの方法があります:

1. シリアル端末でカーネルの印刷レベルを変更します:
   ```
   echo 8 > /proc/sys/kernel/printk
   ```
   その後、テストプログラムを実行します:
   ```
   ./mailbox_test 
   ```
2. `dmesg` コマンドを使用して表示します

### 付録
関連コードのディレクトリは以下の通りです:

1. 大コア mailbox Linux driver:
   ```
   linux_5.10/drivers/soc/cvitek/rtos_cmdqu/
   ```
2. 小コア FreeRTOS:
   ```
   freertos/cvitek/driver/rtos_cmdqu/include/rtos_cmdqu.h
   freertos/cvitek/task/comm/src/riscv64/
   ```
