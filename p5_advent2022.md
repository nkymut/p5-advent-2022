---
layout: page
title: p5.jsとmicro:bit でフィジカルコンピューティング
permalink: /index2.html
has_toc: true
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# p5.jsとmicro:bitでフィジカルコンピューティング

本記事は、[Processing Advent Calendar 2022](https://adventar.org/calendars/7370)の9日目の記事です。

## はじめに

この記事では、p5.jsとmicro:bitを使ってプログラミング初学者でも簡単に物理的な入出力デバイスを作成する方法について解説します。
具体的には、p5.jsからWebUSBやWebBluetoothを使ってmicro:bitのボタンやセンサ入力のやり取りする方法を紹介します。

筆者が受け持っているシンガポール国立大学のインダストリアルデザイン学科での授業では、
このp5.jsとmicro:bitをつかって、プログラム経験ゼロから7週で下のような作品を作れるようにしています。

<iframe src="https://www.youtube.com/embed/rqI1p5iXJeo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

フィジカルコンピューティング教材で一般的なProcessingやArduinoではソフトとハードのやり取りにシリアル通信を用いますが、
50人を超えるプログラミング初学者向けクラスでは、
ドライバのインストールから通信ポートのセットアップやトラブルシュートで授業時間(と私の❤️)ががっつり削られてしまいます。

一方、p5.jsとmicro:bit間の通信にはWebブラウザから、有線のWebUSBと無線のWebBluetoothが使えます。
この場合、追加ドライバのインストールや設定が要らずWebブラウザだけで完結できるので楽勝です

また、micro:bitには標準で加速度、温度、光、磁気センサなどの基本的なセンサやボタン入力が装備されているため、
自作ハードウェア特有のセンサ値の不安定さに悩まされることなく、即センサ入力を利用したインタラクション設計を始めることができます。
初等STEM教育向けといった印象のあるmicro:bitですが意外とパワフルなのです。

というわけで、ここでは、WebUSBとWebBluetooth経由でp5.jsとmicro:bitの通信を可能にする
以下の2つのライブラリをご紹介いたします。

||||
|--|--|--|
|WebUSB| [https://github.com/nkymut/microbit-webusb-p5js](https://github.com/nkymut/microbit-webusb-p5js)|[microbit-webusb](https://github.com/bsiever/microbit-webusb) をクラス化、<br>複数インスタンス接続可にしたもの|
|WebBluetooth|[https://github.com/nkymut/microbit-webble-p5js](https://github.com/nkymut/microbit-webble-p5js)|[microBit.js by antefact ](https://antefact.github.io/microBit.js/)に[IAMAS小林茂さんのGist](https://gist.github.com/kotobuki/7c67f8b9361e08930da1a5cfcfb0653f)のコードをマージしてUART対応したもの|


## 環境と制限

いいことずくめのmicro:bit over WebUSB/WebBluetoothですが、p5.jsと使用するのにいくつか環境の制限があります。

### WebブラウザのWebUSB/WebBluetooth対応状況

2022年12月現在、WebUSBとWebBluetoothをサポートしているブラウザはChrome(とOpera)のみになっています。
特にSafariではセキュリティ上の理由で実装しないことが[明言されている](https://www.zdnet.com/article/apple-declined-to-implement-16-web-apis-in-safari-due-to-privacy-concerns/)ので今後のサポートも期待できないでしょう。

各Webブラウザでのサポート状況は以下の様になっています。([caniuse.com](caniuse.com)調べ)

|  環境 | [WebUSB](https://caniuse.com/webusb)  |　[WebBluetooth](https://caniuse.com/web-bluetooth)|
|---|:---:|:---:|
| Chrome (Win/Mac) / Edge  |○|○|
| Safari |×|×|
| Firefox |×|×|
| Opera |○|○|
| Chrome on Android |○|○|
| Safari on iOS|×|×|
| Chromium on Raspberry PI |_|_|


### p5.js 環境の対応状況

またp5js.org謹製エディタユーザーには悲しいお知らせなのですが、[editor.p5js.org](https://editor.p5js.org/ ) 
では現状[WebBluetoothが機能していない](https://github.com/processing/p5.js-web-editor/issues/1900)ようです。
[https://openprocessing.org/](https://openprocessing.org/)かローカル[VS Code上のLiveServerで試す](https://timrodenbroeker.de/how-to-use-p5-js-with-visual-studio-code/)のをオススメします。

|  環境 | WebUSB  |　WebBluetooth|
|---|:---:|:---:|
| [https://editor.p5js.org/](https://editor.p5js.org/ )  |○|×|
| [https://openprocessing.org/](https://openprocessing.org/) |○|○|
|  [https://glitch.com/](https://glitch.com/) |○|○|
|  [VSCode LiveServer](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) |○|○|


# WebUSBサンプル：光センサ入力

では早速、micro:bitの光センサで電球の色を変えてみましょう。

<style>
.p5div{
  padding:10px;
  background:#F0F0F0;
  display: flex;
  align-items: center;
  justify-content: center;
}  
.p5livesample{
  height: 430px;
  width: 400px;
  padding: 0px;
  margin: 0px;
  top: 0%;
  left: 0;
  overflow: hidden;
  border: none;
}
</style>

[https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/](https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/)

<div class="p5div">
<iframe class="p5livesample" allow="usb" src="https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/"> </iframe>
</div>


## micro:bitプログラム

まずはmicro:bitから光センサの値をシリアルUARTで送信するコードを用意します。
<br>下の様にアホほど簡単です。

プログラムのURL：[https://makecode.microbit.org/_c7AV2KYY6YH9](https://makecode.microbit.org/_c7AV2KYY6YH9)
- [[serial write line]](https://makecode.microbit.org/reference/serial/write-line)で[[light level]](https://makecode.microbit.org/reference/input/light-level)をUART経由で送信します。

![micro:bit 光センサコード](https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/microbit_code/SendLightSensor_microbit.png)

プログラムが作成できたら、Downloadボタン横のConnect deviceをクリックして指示に従うとmakecodeWebエディタとmicro:bitがWebUSB越しにペアリングされてプログラムを直接micro:bitにダウンロードできるようになります。

もしペアリングに失敗する場合、micro:bitのファームウェアが古い場合があるので

![エディタとペアリング](./assets/MicroBit_download_pair.png)



micro:bitのコードをロードしたらShowDataからコンソールのグラフ表示でセンサーが正常に動作していることを確認します。


micro:bitのプログラムもWebUSB経由で書き込み可能なので
ペアリングをしてしまえば書き込みボタン押し込みで自動的にアップデートされます。



![プログラムのセンサ入力値のチェック画像](./assets/Microsoft-MakeCode-Lightsensor.png)

コンソールで十分動作を確認したら、ブラウザのタブを閉じるか、アドレスバー左横の🔒アイコンをクリックしてmicro:bitのペアリングを解除します。これをしないと後にp5.js側とシリアルのバッファを奪い合う形になり通信速度が低下してラグが生じます。

![micro:bitのWebUSBペアリングを解除](./assets/resetWebUSB_connection.png)

## [ubitwebusb.js](https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js)ライブラリのロード

次に、p5.js側のindex.htmlファイルに以下のラインを追加して[ubitwebusb.js](https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js)ライブラリを追加します。


```html
  <script language="javascript" type="text/javascript" src="https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js"></script>
```

## WebUSBライブラリ のセットアップ
ライブラリを設定したら、以下の様にsetup()関数内でp5グローバル変数で宣言したmicroBitにuBitWebUsbオブジェクトのインスタンスを作成します。

```js
  let microBit; //グローバル変数

  microBit = new uBitWebUSB(); //microBit WebUSBインスタンスの作成
```
次にmicro:bitへの接続・切断用のボタンを作成します。`microBit.connectDevice();`で接続、`microBit.disconnectDevice();`で切断です。Chromeのセキュリティ上の理由からユーザーの入力なしに外部デバイスと接続できないようになっているのでボタン押下イベントコールバックとして設定します。

```js
  /* setup() 内*/
  //add connect button
  connectBtn = createButton("connect");
  connectBtn.mousePressed(connect);
  //add disconnect button
  disconnectBtn = createButton("disconnect");
  disconnectBtn.mousePressed(disconnect);
```

```js
/* sketch内のどこか */
//connect to microBit
function connect() {
  microBit.connectDevice();
}

//disconnect from microBit
function disconnect() {
  microBit.disconnectDevice();
}

```

p5.js とmicro:bitのやりとりはコールバック関数として設定します。
`microBit.onConnect()`と`microBit.onDisonnect()`では接続・切断時のふるまいを指定して、`microBit.setReceiveUARTCallback(callbackFunction)`ではmicro:bitからのデータを受信したときにどうするかを設定します。ここでは受信した光センサのデータを電球オブジェクトの明るさ値に設定しています。

```js
  /* setup() 内*/
  microBit.onConnect(function(){　// 接続成功コールバック
    console.log("connected");
  });

  microBit.onDisconnect(function(){ //切断時コールバック
    console.log("disconnected");
  });


  microBit.setReceiveUARTCallback( // UART受信コールバック
    function(receivedData){　
      let val = int(receivedData); //受信テキストを数値に変換
      bulb.brightness = val; //電球オブジェクトの明るさ変更
    
      fadeSlider.value(bulb.brightness); //フェーダーに明るさを表示
    }
  );

```

## 動作チェック

<div class="p5div">
<iframe class="p5livesample" allow="usb" src="https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/"> </iframe>
</div>


というわけで試してみましょう、Connectボタンを押すと、下のようなポップアップ表示が出るので、お目当てのmicro:bitを選択して接続します。

![](./assets/WebUSB_pairing.png)

micro:bitと接続が完了すると、光センサの値によって電球の明るさが変わるはずです。
<br>もし、反応がひどく鈍い場合、
- micro:bitのエディタがmicro:bitに接続しっぱなしになっていないか？
- 複数のタブでWebUSBを使うp5スケッチが開かれて両方ともmicro:bitに接続されていないか?
- console.log()やprint()でUART受信したデータのログを大量に吐いてないか？

を確認してください。

以下、コードの全体です。

[https://github.com/nkymut/microbit-webusb-p5js/blob/master/examples/RecvLightSensor/sketch.js](https://github.com/nkymut/microbit-webusb-p5js/blob/master/examples/RecvLightSensor/sketch.js)


---

# WebBluetoothサンプル：光センサ

WebUSBが動いたところで早速、同じコードをWebBluetoothで無線化してみましょう。

## micro:bitプログラム

[https://makecode.microbit.org/_WhKc1b3w82kx](https://makecode.microbit.org/_WhKc1b3w82kx)

WebBluetooth光センサ micro:bitコード v01用: [microbit-LightSensorBLEUARTv01.hex](https://nkymut.github.io/microbit-webble-p5js/examples/RecvUARTLightSensor/microbit_code/microbit-LightSensorBLEUARTv01.hex)

WebBluetooth光センサ micro:bitコード v02用:[microbit-LightSensorBLEUARTv02.hex](https://nkymut.github.io/microbit-webble-p5js/examples/RecvUARTLightSensor/microbit_code/microbit-LightSensorBLEUARTv02.hex)

### bluetooth uart service の設定
WebUSBサンプルで使用したコードに2点追加します。

　- [[bluetooth uart service](https://makecode.microbit.org/reference/bluetooth/start-uart-service)]: Bluetooth UART サービスを開始する。

 - [bluetooth uart service] はmicro:bitのBluetooth UART サービスを開始し、 
 - [bluetooth uart write line] でmicro:bitからテキストメッセージを送信します。
 
[bluetooth uart write line]  は、数値データを直接指定することができないのでText->[convert to text]ブロックを使って数値をテキストに変換する必要があります。 

![micro:bit BLE 光センサコード](./assets/WebBle_lightsensor.png)


これでも最低限動くのですが、
[on bluetooth connected]と[on bluetooth disconnected]を利用してBluetoothの接続状態を表示すると便利です。
しかし、LEDマトリクスの表示はメモリを大量に使用するらしく非力なv01(スピーカーやマイク入力のついていないタイプ)のmicro:bitだとError ２０を表示して止まってしまう場合があります。

![micro:bit BLE 光センサコード改良版](./assets/WebBle_lightsensor2.png)

### micro:bit のペアリングモード設定
そして最後に一番大事なプロセス、makeCodeエディタ右上の⚙️のProject Settingsを選択、のち`No Parining Required`オプションを選択してください。
これを選択しないと、p5スケッチのConnectボタンを押してもmicro:bitが一覧に出てこず大いに時間を無駄にします。

|||
|:-:|:-|
|![NoPairing 設定メニュー](./assets/MicroBit_MakecodeNoPairing.png)|![No Pairing 設定２](./assets/MicroBit_MakecodeNoPairing2.png)|



https://makecode.microbit.org/_F8DFrygkTRP1

## WebBluetoothライブラリのロード

次に、p5.js側のindex.htmlファイルに以下のラインを追加して[ubitwebble.js](https://nkymut.github.io/microbit-webble-p5js/ubitwebble.js)ライブラリを追加します。


```html
  <!-- WebUSBライブラリ -->
  <!-- <script language="javascript" type="text/javascript" src="https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js"></script> -->

  <!-- WebBluetooth ライブラリ -->
  <script language="javascript" type="text/javascript" src="https://nkymut.github.io/microbit-webusb-p5js/ubitwebble.js"></script>
```

## WebBluetoothライブラリ のセットアップ
ライブラリを設定したら、以下の様にsetup()関数内でmicroBitの`uBitWebUSB()`から`uBitWebBluetooth`オブジェクトのインスタンスを作成します。

```js
  let microBit; //グローバル変数

  microBit = new uBitWebBluetooth(); //microBit WebBluetoothインスタンスの作成
```

## 動作テスト

では実際に動作テストをしてみましょう！
<br>えっ他にコードを追加しなくてよいかって？
そうなんです、`uBitWebUSB`と`uBitWebBluetooth`
では同じAPIでUART通信できるようにしてあるので、インスタンスの変更だけで動きます。
あっと驚くタメゴローです。


<div class="p5div">
<iframe class="p5livesample" allow="usb" src="https://nkymut.github.io/microbit-webble-p5js/examples/RecvUARTLightSensor/"> </iframe>
</div>


# WebBluetoothサンプル：加速度センサ





## その他のサンプル



---
## まとめ

以上、p5.jsとmico:bitをWebUSBとWebBluetooth経由で通信させて簡単な物理入力デバイスを作る方法を紹介しました。

p5.jsでハードウェア入力を扱う方法としては他に
WebMidiという方法がありますが以前p5.soundを絡めたチュートリアルがここにありますので興味がある方はぜひ。
https://github.com/nkymut/ShapeOfSound/blob/main/tutorials/wk06/p5sound_tutorial.md#06-p5sound--midi-input




## 参考


### p5.js 環境でのWeb USB, WebBluetoothライブラリ

p5.js上でのWebUSBやWebBluetoothの各種ライブラリやサンプルについては、
ググってもWeb上にいまいち情報がまとまってないのでここにまとめます。

|プロトコル|  ライブラリ | 状況  |ライセンス|
|:---:|:---:|---|:---:|
|WebUSB|  [microbit-webusb](https://github.com/bsiever/microbit-webusb) |  WebUSB 安定してるがクラス化されてない  |MIT|
|WebBluetooth| [microBit.js by antefact ](https://antefact.github.io/microBit.js/) | WebBluetooth 一通り動くがI/O PinとUARTサポートしてない |LGPL-2.1|
|WebBluetooth|[IAMAS小林茂さんのGist](https://gist.github.com/kotobuki/7c67f8b9361e08930da1a5cfcfb0653f)|WebBluetooth UART p5.js サンプルコード|?|
|WebBluetooth| [p5.ble.js](https://itpnyu.github.io/p5ble-website/)  | p5.js汎用WebBluetoothライブラリ、<br>UUIDを直接指定する必要あり、初学者にはハードル高い  | MIT|
|WebBluetooth| [p5.toio](https://github.com/tetunori/p5.toio)|toio™をWebBlueetoh経由でp5から操作する|MIT|
|WebMIDI| |||
|WebSerial| |||
|WebSerial| |||


また、[p5js.org](https://p5js.org/libraries/)のライブラリ一覧にある[p5-serial](https://p5-serial.github.io/)を使うとシリアル通信を利用可能ですが、ホストPC側に別途ミドルウェアをインストールする必要があり、イマイチ不便なところがあります。




- micro:bit Bluetooth Profile <br>
https://lancaster-university.github.io/microbit-docs/resources/bluetooth/bluetooth_profile.html