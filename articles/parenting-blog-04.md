---
title: "【クラウドエンジニアのパパ育休#04】Raspberry Pi でベビールームの温度監視をやってみた"
emoji: "🍼"
type: "idea"
topics: ["育児","パパ育休","raspberrypi"]
published: true
published_at: 2023-08-31 09:30
---

[株式会社カサレアル](https://www.casareal.co.jp/) プロフェッショナルソリューション技術部の S.N. です。

早いもので9月から職場復帰です。復帰といっても在宅なので状況はあまり変わらないですが。
娘も順調に成長していて、もうすぐ 6,000 g になります。
いよいよ片手で抱っこするのが大変になってきました。
今は絶賛腹這いの練習中です💪

---

ところで、赤ちゃんの子育てをしている方はベビールームどうしてますか？
我が家は赤ちゃんが落ち着いて寝られる部屋をつくって寝ているときはベビールームに居させています。
ベビールームでは常にエアコンをつけていて温度が一定になるようにしています。

ですがある日、娘の様子を見ようとベビールームに入ると「あれ？ちょっと熱くない？」と思う瞬間がありました。
急いで空調温度を下げましたがこれは万が一のことがあると大変だなぁと思い、ベビールームの監視方法について調べ始めました。（もっと高性能なエアコン買えよというのはさておき笑）

今の時代とても恵まれていて、赤ちゃんの様子を遠隔から見守ることができるベビーモニターなんて物があるのですね！

**でも、エンジニアはこれを自らの手でエンジニアリングします🙃（笑）**

---

ということで前置きが長くなりましたが、今回は `Raspberry Pi 4 Model B` に温湿度センサー `DHT22` を繋げてベビールームの監視をしてみた話です。

とくに難しいことはしておらず、単に3分間隔で基準の温度や湿度からある程度外れると LINE に通知が来るようにしてみただけです。
👇の画像のように通知が来るようにしました。

![](/images/parenting-blog-04/line-notify.jpg =500x)
*室温が高く湿度が低い時の LINE 通知*

雑ですがベビールームには👇のように Raspberry Pi を設置しています。

![](/images/parenting-blog-04/raspberrypi.jpg =500x)
*設置された Raspberry Pi*

---

テックブログらしくコードも一部紹介してみようかな笑

ということで Raspberry Pi で温湿度センサーから値を取得する方法はいろんな方がやっていますが、自分は👇のように取得しました。

```bash
pip install setuptools wheel adafruit-circuitpython-dht
```

```python:main.py
import adafruit_dht
import board

DHT_DEVICE: adafruit_dht.DHT22 = adafruit_dht.DHT22(board.D15)

try:
    temperature_c = DHT_DEVICE.temperature
    humidity_percent = DHT_DEVICE.humidity
except Exception as error:
    DHT_DEVICE.exit()
    raise error
```

これで `temperature_c` に℃の温度、`humidity_percent` に百分率の湿度の値が設定されます（float 型）。
デバイスのトラブルで何かしらの例外が発生する可能性があるので、一応例外をハンドリングしておくとよいでしょう。
例外発生時は、`DHT_DEVICE.exit()` で温湿度センサーのデバイスを終了させます。

```python
DHT_DEVICE: adafruit_dht.DHT22 = adafruit_dht.DHT22(board.D15)
```

で温湿度センサーの DOUT に繋げている汎用I/O（RXD）のピン15から観測データを取得していますが、素人すぎてココで何番を指定すればいいのか迷いました。。。

[温湿度センサー（DHT22）のページ](https://www.waveshare.com/wiki/DHT22_Temperature-Humidity_Sensor) の `Use with Raspberry Pi` の接続対応表とピンの対応を見て何とか頑張りました。
ココにたどり着くまでが結構大変でした😵‍💫

| センサー側 | Raspberry Pi 側 | 説明                |
| :--------: | :-------------: | :------------------ |
|    VCC     |      3.3V       | Power input         |
|    GMD     |       GND       | Power ground        |
|    DOUT    |      GP15       | Digital data output |


![](/images/parenting-blog-04/GPIO-Pinout-Diagram-2.webp =500x)
*Raspberry Pi 4 Model B のピン対応*

LINE への通知は次の通り、LINE が用意している通知用の API に POST リクエストしているだけです。

```python
import os
import requests

line_notify_token = os.environ.get("LINE_NOTIFY_TOKEN")
line_notify_api = "https://notify-api.line.me/api/notify"
headers = {"Authorization": f"Bearer {line_notify_token}"}
data = {"message": "＜通知内容の文字列＞"}
requests.post(line_notify_api, headers=headers, data=data)
```

上記のコードは環境変数に設定されたトークンを使用していますが、LINE 通知のトークンは👇にアカウント登録して発行することができます。

https://notify-bot.line.me/ja/

---

今回は Raspberry Pi に触れましたがやはり物理デバイスの配線なんかは初めてだと戸惑いますね。
でも貴重な経験ができました！
もし Raspberry Pi の案件がございましたらぜひ弊社へお声掛けください！笑

ちなみに、今回のように温度が一定値以上の幅から外れた際にアラームが鳴ってしかも赤ちゃんの様子がカメラで確認できるベビーモニターは 16,000 円くらいで購入することができます。
今回購入した Raspberry Pi 4 Model B と 温湿度センサーはというと。。。

今回の教訓は、

**コスパと品質を考えるならおとなしくベビーモニターを購入しよう！**

それでは次回の投稿でまたお会いしましょう。
See you next バイバイ👋

# 関連記事

https://zenn.dev/casa_snona/articles/parenting-blog-00
https://zenn.dev/casa_snona/articles/parenting-blog-01
https://zenn.dev/casa_snona/articles/parenting-blog-02
https://zenn.dev/casa_snona/articles/parenting-blog-03
