minIRum (+)
======

A minimal implementation of infrared sender/receiver like IRKit by ESP8266

(オリジナルのminIRumに、ステータスLEDを追加しました。またaJsonを標準のものからikakunsan/aJson-modに変えることで、255個以上の長いデータ（エアコン等）に対応できるようになります)

## Description

IRKitの GET,POST /messages 同様の機能**だけ**をESP8266で実現する最小限の実装です

ESP-WROOM-02で動作確認済み

Maybe it will work on other ESP* boards.

## How to use

### 1. Wiring

以下の通り接続

* GPIO 12 - Infrared LED
* GPIO 14 - Infrared receiver

![schematic](https://raw.githubusercontent.com/9SQ/minIRum/master/schematic.png)

赤外線リモコン受信モジュールは[OSRB38C9AA](http://akizukidenshi.com/catalog/g/gI-04659/)などの3.3V駆動可能のものを利用します

### 2. Flashing the firmware

Arduino IDEで minirum/minirum.ino を開き、ESP-WROOM-02に書き込みます。

Arduino core for ESP8266 WiFi chipはバージョン **2.3.0** を利用してください。

書き込み後、シリアルモニタを開いて `115200 bps` で接続するとログを確認できます。

### 3. Power ON & Setup

1. 電源を入れると設定モードで起動します。
2. PCやスマートフォンで SSID `minIRum_SETUP` の無線LANアクセスポイントに接続します。
3. Captive Portalにより設定画面が自動的に開きます。(開かない場合はブラウザで `http://192.168.1.1/` にアクセスしてください)
4. 既設の無線LANアクセスポイントのSSIDを選択し、パスワードを入力して `送信` をクリックすると設定が保存され、自動的に再起動します。
5. 再起動後、設定された無線LANアクセスポイントに自動的に接続、DHCPによりIPアドレスを取得します。
6. IPアドレスはシリアルモニタに表示されます。

アクセスポイントが見つからない等の理由で接続できなかった場合、再び設定モードになります。

### See also

回路図とパーツリストは以下のブログ記事を参照してください。

[ミニマルなIRKitクローンを作ってiOSから家電を制御する : Eleclog.](http://eleclog.quitsq.com/2016/09/minirum.html)

## Example Requests

mDNSに対応している場合 minirum-[MACアドレスの下位3オクテット].local のアドレスで接続できます。

**GET /messages**

```sh
curl -i http://minirum-a492cd.local/messages
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 383
Connection: close
Access-Control-Allow-Origin: *

{"format":"raw","freq":38,"data":[3550,1700,500,400,500,400,500,1300,500,1300,500,400,500,1300,500,400,500,400,500,400,500,1300,500,400,500,400,500,1300,500,400,500,1300,500,400,500,1300,500,400,500,400,500,1300,500,400,500,400,500,400,500,400,500,1300,500,400,500,1300,500,1300,500,400,500,1300,500,400,500,400,500,400,500,400,500,1300,500,400,500,400,500,1300,500,400,500,400,500]}
```

**POST /messages**

```sh
curl -i http://minirum-a492cd.local/messages -d'{"format":"raw","freq":38,"data":[3550,1700,500,400,500,400,500,1300,500,1300,500,400,500,1300,500,400,500,400,500,400,500,1300,500,400,500,400,500,1300,500,400,500,1300,500,400,500,1300,500,400,500,400,500,1300,500,400,500,400,500,400,500,400,500,1300,500,400,500,1300,500,1300,500,400,500,1300,500,400,500,400,500,400,500,400,500,1300,500,400,500,400,500,1300,500,400,500,400,500]}'
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 2
Connection: close
Access-Control-Allow-Origin: *
```

### simple web console

ブラウザでroot URLにアクセスすると簡易的なコンソールが利用できます。

![webconsole](https://raw.githubusercontent.com/9SQ/minIRum/master/webconsole.png)

## Requirements

* [Arduino core for ESP8266 WiFi chip](https://github.com/esp8266/Arduino) <= **2.3.0**
* [aJson](https://github.com/interactive-matter/aJson) == v1.5 ([aJson-mod](https://github.com/ikakunsan/aJson-mod)がオススメ)
* [IRremoteESP8266](https://github.com/markszabo/IRremoteESP8266) >= v2.5.0

## 追加
まず、すばらしいプロジェクトを公開してくださった9SQさんに感謝します。

ステータスの表示用に、[OSRGHC3132A](http://akizukidenshi.com/catalog/g/gI-06313/)のような、カソードコモンの2色LEDを追加できます。（[拡散キャップ](http://akizukidenshi.com/catalog/g/gI-01126/)を付けると落ち着いた光り具合になります）
上記のLEDの場合、緑のアノードを1kΩ経由で+5Vに、赤のアノードを4.7kΩ経由でGPIO 15に(各抵抗値はLEDの種類によります)、共通のカソードはGNDに接続します。
LEDは、電源を入れると一旦オレンジになり、設定を終えてWiFiに接続し、通常動作に入ると緑に変わります。また、赤外線LEDが発光している間もオレンジに変わります。

このインジケーターで、電源投入後、緑が点灯しない場合は電源が供給されていない、ずっと緑の場合はマイコンが正常に動いていない、オレンジから緑に切り替わらない場合はアクセスポイントに接続できていないか設定モードに入っている、といった切り分けができるようになります。

オリジナルのaJsonには、扱えるエレメントの最大値が255個までという制限があります。一般にエアコンのデータ長は非常に長く、ものによってはこの制限のためにリモコンデータが正しく学習できない場合があります。（うちのがそうでした (´・ω・`)）
aJsonにパッチを当ててこの制約を外したものを別途用意しましたので、オリジナルの代わりに使うとしあわせになれるかもしれません。
