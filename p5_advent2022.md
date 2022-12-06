---
layout: page
title: About
permalink: /index2.html
---


# p5.js とmicro:bit でフィジカルコンピューティング

本記事は、[Processing Advent Calendar 2022](https://adventar.org/calendars/7370)の9日目の記事です。

## はじめに

この記事では、p5.jsとmicro:bitを使ってプログラミング初学者でも簡単に物理的な入出力デバイスを作成する方法について解説します。
具体的には、p5.jsからWebUSBやWebBluetoothを使ってmicro:bitのボタンやセンサ入力のやり取りする方法を紹介します。

筆者が受け持っているシンガポール国立大学のインダストリアルデザイン学科での授業では、
このp5.jsとmicro:bitをつかって、プログラム経験ゼロから7週で下のような作品を作れるようにしています。

<iframe src="https://www.youtube.com/embed/rqI1p5iXJeo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

フィジカルコンピューティングで一般的なProcessingやArduinoではソフトとハードのやり取りにシリアル通信を用いますが、
50人を超えるプログラミング初学者向けクラスでは、
ドライバのインストールから通信ポートのセットアップやトラブルシュートで授業時間(と私の心)ががっつり削られてしまいます。

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

いいことずくめのp5.jsとmicro:bitですが、使用するのにいくつか環境の制限があります。

### 各WebブラウザのWebUSB/WebBluetooth対応状況

2022年12月現在、WebUSBとWebBluetoothをサポートしているブラウザはChrome(とOpera)のみになっています。
特にSafariではセキュリティ上の理由で実装しないことが[明言されている](https://www.zdnet.com/article/apple-declined-to-implement-16-web-apis-in-safari-due-to-privacy-concerns/)ので今後のサポートも期待できないでしょう。

2022年12月現在現在の各Webブラウザでのサポート状況は以下の様になっています。

|  環境 | [WebUSB](https://caniuse.com/webusb)  |　[WebBluetooth](https://caniuse.com/web-bluetooth)|
|---|:---:|:---:|
| Chrome (Win/Mac) / Edge  |○|○|
| Safari |×|×|
| Firefox |×|×|
| Oper |○|○|
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


# WebUSBチュートリアル

では早速、micro:bitの光センサで電球の色を変えてみましょう。

https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/
<iframe allow="usb" src="https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/"> </iframe>


## micro:bitプログラム

まずはmicro:bitから光センサの値をシリアルUARTで送信するコードを用意します。
下の用にアホほど簡単です。

![micro:bit 光センサコード](https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/microbit_code/SendLightSensor_microbit.png)

プログラムのURL
https://makecode.microbit.org/_c7AV2KYY6YH9

micro:bitのコードをロードしたらShowDataからコンソールのグラフ表示でセンサーが正常に動作していることを確認します。micro:bitのプログラムもWebUSB経由で書き込み可能なので
ペアリングをしてしまえば書き込みボタン押し込みで自動的にアップデートされます。



![プログラムのセンサ入力値のチェック画像](./assets/Microsoft-MakeCode-Lightsensor.png){: width="50%" }

コンソールで十分動作を確認したら、ブラウザのタブを閉じるか、アドレスバー左横の🔒アイコンをクリックしてmicro:bitのペアリングを解除します。これをしないと後にp5.js側とシリアルのバッファを奪い合う形になり通信速度が低下してラグが生じます。

![micro:bitのWebUSBペアリングを解除](./assets/resetWebUSB_connection.png)

## [ubitwebusb.js](https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js)ライブラリのロード

次に、p5.js側のindex.htmlファイルに以下のラインを追加して[ubitwebusb.js](https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js)ライブラリを追加します。


```html
  <script language="javascript" type="text/javascript" src="https://nkymut.github.io/microbit-webusb-p5js/ubitwebusb.js"></script>
```

## WebUSBライブラリ のセットアップ
ライブラリを設定したら、以下の用にsetup()関数内でWebUSBのインスタンスを作成し

```js
//グローバル変数

let microBit;

// Buttons to connect/disconnect to micro:bit
let connectBtn;
let disconnectBtn;

```

```js

  microBit = new uBitWebUSB(); //microBit WebUSBインスタンスの作成

  microBit.onConnect(function(){　// 接続成功コールバック
    console.log("connected");
  });

  microBit.onDisconnect(function(){ //切断時コールバック
    console.log("disconnected");
  });


  microBit.setReceiveUARTCallback( // UART受信コールバック
    function(receivedData){　
      let val = int(receivedData);
      bulb.brightness = val;
    
      fadeSlider.value(bulb.brightness);
    }
  );

```
次に接続・切断用のボタンを作成します。`microBit.uBitConnectDevice();`で接続、`microBit.uBitDisconnect();`で切断です。Chromeのセキュリティ上の理由からユーザーの入力なしに外部デバイスと接続できないようになっているのでボタン押下イベントコールバックとして設定します。

```js
  //setup() 内
  //add connect button
  connectBtn = createButton("connect");
  connectBtn.mousePressed(connect);
  //add disconnect button
  disconnectBtn = createButton("disconnect");
  disconnectBtn.mousePressed(disconnect);
```

```js
  // sketch内のどこか
//connect to microBit
function connect() {
  microBit.uBitConnectDevice();
}

//disconnect from microBit
function disconnect() {
  microBit.uBitDisconnect();
}

```

##
というわけで試してみましょう、Connectボタンを押すと、下のようなポップアップ表示が出るので、お目当てのmicro:bitを選択して接続します。

![](./assets/WebUSB_pairing.png)

micro:bitと接続が完了すると、光センサの値によって電球の明るさが変わるはずです。
もし、反応がひどく鈍い場合、micro:bitのエディタが接続しっぱなしになっていないか？
複数のタブで同じスケッチが開かれて両方とも接続されていないかを確認してください。

<iframe allow="usb" src="https://nkymut.github.io/microbit-webusb-p5js/examples/RecvLightSensor/"> </iframe>

以下、コードの全体です。

---


# WebBluetoothチュートリアル
## WebUSBライブラリ のセットアップ







---
## まとめ

WebUSBとWebBluetoothの使い方を紹介しました。

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

また、[p5js.org](https://p5js.org/libraries/)のライブラリ一覧にある[p5-serial](https://p5-serial.github.io/)を使うとシリアル通信を利用可能ですが、ホストPC側に別途ミドルウェアをインストールする必要があり、イマイチ不便なところがあります。




- micro:bit Bluetooth Profile <br>
https://lancaster-university.github.io/microbit-docs/resources/bluetooth/bluetooth_profile.html