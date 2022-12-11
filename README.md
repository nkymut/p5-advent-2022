---
layout: page
title: README
permalink: /readme.html
author: Yuta Nakayama
summary: README
image: /assets/WebBle_lightsensor2.png
date: 09/12/2022
nav_order: 3
---

# p5.jsとmicro:bitでフィジカルコンピューティング

本レポジトリは、[Processing Advent Calendar 2022](https://adventar.org/calendars/7370)の9日目の記事レポジトリです。

[https://nkymut.github.io/p5-advent-2022/](https://nkymut.github.io/p5-advent-2022/)


この記事では、[p5.js](https://p5js.org/)と[micro:bit](https://microbit.org/)を使ってプログラミング初学者でも簡単に物理的な入出力デバイスを作成する方法について解説しています。

具体的には、p5.jsからWebUSBとWebBluetooth経由でmicro:bitとの通信を可能にする
以下の2つのライブラリを紹介しています。

||||
|--|--|--|
|WebUSB| [https://github.com/nkymut/microbit-webusb-p5js](https://github.com/nkymut/microbit-webusb-p5js)|[microbit-webusb](https://github.com/bsiever/microbit-webusb) をクラス化、<br>複数インスタンス接続可にしたもの|
|WebBluetooth|[https://github.com/nkymut/microbit-webble-p5js](https://github.com/nkymut/microbit-webble-p5js)|[microBit.js by antefact ](https://antefact.github.io/microBit.js/)に[IAMAS小林茂さんのGist](https://gist.github.com/kotobuki/7c67f8b9361e08930da1a5cfcfb0653f)のコードをマージしてUART対応したもの|

この記事は、[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa] のもとで公開されており、自由に複製・改変・共有できますが、商業誌やウェブ記事等へ転載される場合は nkymut★gmail.com までご一報ください。


# Physical computing with p5.js and micro:bit

This repository is the article repository for the 9th day of [Processing Advent Calendar 2022](https://adventar.org/calendars/7370).

[https://nkymut.github.io/p5-advent-2022/](https://nkymut.github.io/p5-advent-2022/)

This article explains how to use [p5.js](https://p5js.org/) and [micro:bit](https://microbit.org/) to create physical input/output devices easily.
Specifically, we will show how to communicate with micro:bit sensor inputs from p5.js via WebUSB and WebBluetooth.

||||
|--|--|--|
|WebUSB| [https://github.com/nkymut/microbit-webusb-p5js](https://github.com/nkymut/microbit-webusb-p5js)|[microbit-webusb](https://github. com/bsiever/microbit-webusb) as a class, <br>multi-instance connection possible|
|WebBluetooth|[https://github.com/nkymut/microbit-webble-p5js](https://github.com/nkymut/microbit-webble-p5js)|[microBit.js by antefact ](https://antefact.github.io/microBit.js/) with UART support by merging [IAMAS Shigeru Kobayashi's Gist](https://gist.github.com/kotobuki/7c67f8b9361e08930da1a5cfcfb0653f) code|
This article is published under the [Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa], and can be freely copied, modified, and shared. Please let us know at nkymut★gmail.com.

Copyright (c) 2022 Yuta Nakayama

 [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg