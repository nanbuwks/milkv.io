---
sidebar_label: 'pinmux'
sidebar_position: 05
---

### Duoのピンネーム

<div className='gpio_style'>

| PIN NAME |              Pin#               |              Pin#                | PIN NAME |
| -------- | :-----------------------------: | :------------------------------: | -------- |
| GP0      | <div className='green'>1</div>  |    <div className='red'>40</div> | VBUS     |
| GP1      | <div className='green'>2</div>  |    <div className='red'>39</div> | VSYS     |
| GND      | <div className='black'>3</div>  |  <div className='black'>38</div> | GND      |
| GP2      | <div className='green'>4</div>  | <div className='orange'>37</div> | 3V3_EN   |
| GP3      | <div className='green'>5</div>  |    <div className='red'>36</div> | 3V3(OUT) |
| GP4      | <div className='green'>6</div>  |   <div className='gray'>35</div> |          |
| GP5      | <div className='green'>7</div>  |   <div className='gray'>34</div> |          |
| GND      | <div className='black'>8</div>  |  <div className='black'>33</div> | GND      |
| GP6      | <div className='green'>9</div>  |  <div className='green'>32</div> | GP27     |
| GP7      | <div className='green'>10</div> |  <div className='green'>31</div> | GP26     |
| GP8      | <div className='green'>21</div> | <div className='orange'>30</div> | RUN      |
| GP9      | <div className='green'>12</div> |  <div className='green'>29</div> | GP22     |
| GND      | <div className='black'>13</div> |  <div className='black'>28</div> | GND      |
| GP10     | <div className='green'>14</div> |  <div className='green'>27</div> | GP21     |
| GP11     | <div className='green'>15</div> |  <div className='green'>26</div> | GP20     |
| GP12     | <div className='green'>16</div> |  <div className='green'>25</div> | GP19     |
| GP13     | <div className='green'>17</div> |  <div className='green'>24</div> | GP18     |
| GND      | <div className='black'>18</div> |  <div className='black'>23</div> | GND      |
| GP14     | <div className='green'>19</div> |  <div className='green'>22</div> | GP17     |
| GP15     | <div className='green'>20</div> |  <div className='green'>21</div> | GP16     |
|          | &nbsp;                          |                                  |          |
| GP25     | <div className='blue'>LED</div> |                                  |          |

</div>

### ピン多重構成

MDuoのピンの多くは多目的機能を持っています。アプリケーション（例えばwiringXやpinpongなど）を使用して各ピンの機能を制御するとき、ピンの現在の状態を確認して、それが望ましい機能に一致することを確認することが重要です。もし一致しない場合は、`duo-pinmux`コマンドを使用して望ましい機能に切り替えることができます。

`duo-pinmux`コマンドを直接実行すると、使用方法の指示を表示することができます。
```
[root@milkv-duo]~# duo-pinmux
pinmux for duo
duo-pinmux -p          <== List all pins
duo-pinmux -l          <== List all pins and its func
duo-pinmux -r pin      <== Get func from pin
duo-pinmux -w pin/func <== Set func to pin
```

特定のピン、例えばDuoのピン1の多重化状態を確認するには、そのピンの名前を知る必要があります。上記の図で示されているように、ピン1の名前は`GP0`です。その多重化状態を表示するには、`duo-pinmux -r`コマンドに続けてピン名を使用します。
```
[root@milkv-duo]~# duo-pinmux -r GP0
GP0 function:
[ ] JTAG_TDI
[ ] UART1_TX
[ ] UART2_TX
[ ] GP0
[v] IIC0_SCL
[ ] WG0_D0
[ ] DBG_10
```
現在の機能が`IIC0_SCL`であることがわかります。これは、それがI2C0のSCLピンとして設定されていることを示しています。ピン1をGPIOとして設定したい場合は、次のコマンドを使用できますd: `duo-pinmux -w pin_name/function`
```
duo-pinmux -w GP0/GP0
```
次に、再度ピンの多重化状態を確認します。
```
[root@milkv-duo]~# duo-pinmux -r GP0
GP0 function:
[ ] JTAG_TDI
[ ] UART1_TX
[ ] UART2_TX
[v] GP0
[ ] IIC0_SCL
[ ] WG0_D0
[ ] DBG_10
```
これで、`GP0`と`して設定されたことがわかります。

同様に、ピン1をUART1のTXピンとして設定するには、次を実行する必要があります。
```
duo-pinmux -w GP0/UART1_TX
```
多重化状態を確認します。
```
[root@milkv-duo]~# duo-pinmux -r GP0
GP0 function:
[ ] JTAG_TDI
[v] UART1_TX
[ ] UART2_TX
[ ] GP0
[ ] IIC0_SCL
[ ] WG0_D0
[ ] DBG_10
```
期待通りに設定されていることが確認できます。