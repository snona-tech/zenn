---
title: "【クラウドエンジニアのパパ育休#03】詐欺に遭いました。。。"
emoji: "🍼"
type: "idea"
topics: ["育児","パパ育休","仮想通貨","詐欺"]
published: true
published_at: 2023-08-14 10:30
---

:::message alert
本ブログ内で実際の詐欺に関する手口に触れますが、犯罪を助長する意図はありません。
また、詐欺サイトのドメインも公表していますが二次被害を防止するため、興味本位でも**絶対に詐欺サイトにはアクセスしないでください**。
:::

[株式会社カサレアル](https://www.casareal.co.jp/) プロフェッショナルソリューション技術部の S.N. です。

タイトル通り先日、詐欺に遭いまして。。。
タイトル詐欺ならいいんですけどね、、、残念ながら実話です笑

言い訳ですがやはり育児疲れ（とくに寝不足）で判断力が鈍っており、普段なら絶対に引っかからないはずの手口にまんまとやられました😅
自分への戒めとしてブログに残しておきます。。。

---

どういう経緯で詐欺に遭ったのかというと、TikTok というショートムービーをひたすら観れる（投稿もできる）アプリでオススメ動画に流れてきた動画から詐欺サイトに誘導されました。

TikTok 自体は悪質なアプリではないのでおススメです。
企業の PR 活動に TikTok が使われていたりもしますし、メジャーな SNS になりつつあるのかなと思っています。

話を戻して、問題の動画の内容はというと、「イーロン・マスクがビットコインを配るキャンペーンをしている！」「プロモーションコード入れたら`0.41 BTC`もらえた！ありがとう！」というテキストが表示されていて、実際にサイトにプロモーションコードを入力して、ウォレットに入金されているシーンが映し出されている動画でした。（すでに動画は削除されていました。）

:::message
本ブログ執筆時（2023/08/13）で 1.0 BTC = 約 427 万円 なので、
0.41 BTC = 約 175 万円 です。
:::

いやいやさすがにそんなうまい話ないだろ～笑って試しにサイトにアクセスしてアカウント登録＆プロモーションコードを入力してみたんですね。
するとなんと開設したウォレットに`0.41 BTC`入金されたんですよ！
この時の気持ちは「**人生イージー🤑**」でした笑
アホですね～、これでまんまとハマっていくわけです。

次に考えるのは当然、自分が普段使っているウォレットに送金です。
ここで自分宛てのアドレスに送金しようとすると次のようなメッセージが表示されるんですね。

> あなたは新規ユーザなので、外部に出金するためには`0.005 BTC`の入金をしてアカウントをアクティベーションする必要があります。

ほうほう、確かに登録したばっかのユーザが外部に送金できるとなるとマネーロンダリングに利用される可能性があるから信頼のために多少の入金が必要なんだろうなと。

いやこの時点で気づけよ！って思いますよね。。。
完全に判断力が鈍ってました。

そして言われるがままに`0.005 BTC`を入金したんですね。
当然、いくら待ってもアカウントはアクティベーションされません😨

あれ？動画ではこれでアクティベーションされてたのにまだされないぞ？
疑問に思ってサイト内のサポートに問い合わせました。
するとサポートのジェニファー（おそらく偽名）から、最近利用規約が変更されて、`0.01 BTC`入金してプレミアム会員にならないと出金できなくなりましたと返事が返ってきました。

あれれ～？おかしいぞ～？と、ようやくここで詐欺に気づきました😱
サポートのチャットにありったけの捨て台詞をはいてサイトを閉じました（もうこれくらいしか抵抗できないと悟ったので。。。）

---

こういう入念に作りこまれたサイトは、サイト自体に悪質なスクリプトが埋め込まれているわけではないので不審なサイトとしてはヒットしません。。。
悪評が広まっていれば詐欺サイトとしてマークされるかもしれませんが、ぽっと出のサイトでは情報が出回っていないことが多いです。

何となく怪しいなぁと思えるところは他にもあります。
👇は詐欺サイトの About us ページですが、2017年から頑張ってる的なことが書いてあってそれなりの歴史がありそうです。

![](/images/parenting-blog-03/stretax-about-us.png =500x)
*詐欺サイトの About us ページ*

更に、オフィスの拠点はオーストラリアと記されています。

![](/images/parenting-blog-03/stretax-office-located.png =500x)
*詐欺サイトの office located ページ*

このサイトのドメイン情報を取得するため、whois コマンドを叩くと次の通りドメインの作成日が`Creation Date: 2023-08-07T17:49:21Z`（ごく最近）なんですよ。。。
そしてオーストラリアとは縁もゆかりもなさそうなアイスランドのレイキャビク。。。（レジストラだから関係ないかな？）
今更ですがサイトの記載なんて尤もらしいこと書いてあっても何も信用ならないんだなぁと。

```
$ whois stretax.com
   Domain Name: STRETAX.COM
   Registry Domain ID: 2804014389_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.namecheap.com
   Registrar URL: http://www.namecheap.com
   Updated Date: 2023-08-07T18:09:12Z
   Creation Date: 2023-08-07T17:49:21Z
   Registry Expiry Date: 2024-08-07T17:49:21Z
   Registrar: NameCheap, Inc.
   Registrar IANA ID: 1068
   Registrar Abuse Contact Email: abuse@namecheap.com
   Registrar Abuse Contact Phone: +1.6613102107
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Name Server: ALEXIS.NS.CLOUDFLARE.COM
   Name Server: BLAKELY.NS.CLOUDFLARE.COM
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of whois database: 2023-08-13T04:46:32Z <<<

For more information on Whois status codes, please visit https://icann.org/epp

NOTICE: The expiration date displayed in this record is the date the
registrar's sponsorship of the domain name registration in the registry is
currently set to expire. This date does not necessarily reflect the expiration
date of the domain name registrant's agreement with the sponsoring
registrar.  Users may consult the sponsoring registrar's Whois database to
view the registrar's reported date of expiration for this registration.

TERMS OF USE: You are not authorized to access or query our Whois
database through the use of electronic processes that are high-volume and
automated except as reasonably necessary to register domain names or
modify existing registrations; the Data in VeriSign Global Registry
Services' ("VeriSign") Whois database is provided by VeriSign for
information purposes only, and to assist persons in obtaining information
about or related to a domain name registration record. VeriSign does not
guarantee its accuracy. By submitting a Whois query, you agree to abide
by the following terms of use: You agree that you may use this Data only
for lawful purposes and that under no circumstances will you use this Data
to: (1) allow, enable, or otherwise support the transmission of mass
unsolicited, commercial advertising or solicitations via e-mail, telephone,
or facsimile; or (2) enable high volume, automated, electronic processes
that apply to VeriSign (or its computer systems). The compilation,
repackaging, dissemination or other use of this Data is expressly
prohibited without the prior written consent of VeriSign. You agree not to
use electronic processes that are automated and high-volume to access or
query the Whois database except as reasonably necessary to register
domain names or modify existing registrations. VeriSign reserves the right
to restrict your access to the Whois database in its sole discretion to ensure
operational stability.  VeriSign may restrict or terminate your access to the
Whois database for failure to abide by these terms of use. VeriSign
reserves the right to modify these terms at any time.

The Registry database contains ONLY .COM, .NET, .EDU domains and
Registrars.

Domain name: stretax.com
Registry Domain ID: 2804014389_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: http://www.namecheap.com
Updated Date: 0001-01-01T00:00:00.00Z
Creation Date: 2023-08-07T17:49:21.00Z
Registrar Registration Expiration Date: 2024-08-07T17:49:21.00Z
Registrar: NAMECHEAP INC
Registrar IANA ID: 1068
Registrar Abuse Contact Email: abuse@namecheap.com
Registrar Abuse Contact Phone: +1.9854014545
Reseller: NAMECHEAP INC
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Registry Registrant ID:
Registrant Name: Redacted for Privacy
Registrant Organization: Privacy service provided by Withheld for Privacy ehf
Registrant Street: Kalkofnsvegur 2
Registrant City: Reykjavik
Registrant State/Province: Capital Region
Registrant Postal Code: 101
Registrant Country: IS
Registrant Phone: +354.4212434
Registrant Phone Ext:
Registrant Fax:
Registrant Fax Ext:
Registrant Email: 8bd990fbdb2148278b00f263a5e1eafb.protect@withheldforprivacy.com
Registry Admin ID:
Admin Name: Redacted for Privacy
Admin Organization: Privacy service provided by Withheld for Privacy ehf
Admin Street: Kalkofnsvegur 2
Admin City: Reykjavik
Admin State/Province: Capital Region
Admin Postal Code: 101
Admin Country: IS
Admin Phone: +354.4212434
Admin Phone Ext:
Admin Fax:
Admin Fax Ext:
Admin Email: 8bd990fbdb2148278b00f263a5e1eafb.protect@withheldforprivacy.com
Registry Tech ID:
Tech Name: Redacted for Privacy
Tech Organization: Privacy service provided by Withheld for Privacy ehf
Tech Street: Kalkofnsvegur 2
Tech City: Reykjavik
Tech State/Province: Capital Region
Tech Postal Code: 101
Tech Country: IS
Tech Phone: +354.4212434
Tech Phone Ext:
Tech Fax:
Tech Fax Ext:
Tech Email: 8bd990fbdb2148278b00f263a5e1eafb.protect@withheldforprivacy.com
Name Server: alexis.ns.cloudflare.com
Name Server: blakely.ns.cloudflare.com
DNSSEC: unsigned
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net/
>>> Last update of WHOIS database: 2023-08-12T21:46:55.78Z <<<
For more information on Whois status codes, please visit https://icann.org/epp
```

whois コマンドを実行できる環境をすぐに用意できなくても👇のサイトにドメインを入力することで同等の確認ができます。

https://www.whois.com/whois

---

いや、まだプレミアム会員になってみないと詐欺かどうかわからないんじゃないか？
開設された自分のウォレットに入金してるのだから詐欺集団も手が付けられないんじゃないの？
と思う方もいるかもしれませんが、おそらくそもそも開設はされておらず、詐欺集団のウォレットのアドレスがあたかも開設されたかのように表示されていたんじゃないかなと思います。
例えプレミアム会員になれる額を入金してもまた追加でなんやかんや入金させる流れな気がしました。

つまりは、自分の口座をつくったと思って入金していたら詐欺集団の口座に入金していたということだと思います。
これがお前らのやり方かぁあああああ🤬

---

疲れていたとはいえ、まさか自分が詐欺に遭うとは。。。
ライフイベント等で自分を取り巻く環境が大きく変わる時こそ気を緩めてはいけませんね。

今回の教訓は、

**人生はイージーではない、ドメイン情報くらい確認しろよ**

皆さんも他人事と思わず、気をつけなはれや！

それでは次回の投稿でまたお会いしましょう。
See you next バイバイ👋

# 関連記事

https://zenn.dev/casa_snona/articles/parenting-blog-00
https://zenn.dev/casa_snona/articles/parenting-blog-01
https://zenn.dev/casa_snona/articles/parenting-blog-02
https://zenn.dev/casa_snona/articles/parenting-blog-03
