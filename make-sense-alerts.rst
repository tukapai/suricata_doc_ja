アラートの意味を理解する
==========================

アラートが発生したときは、その意味を理解することが重要です。
それは深刻なものですか？関連性はどうですか？偽陽性ですか？

#TODO　あとでやる
To find out more about the rule that fired, it's always a good idea to
look at the actual rule.

ルールで最初に調べるのは、msg"キーワードの後に続く説明です
例を考えてみましょう：

::

  msg:"ET SCAN sipscan probe";



「ET」は、ルールがEmerging Threats から来たものであることを示します。
「SCAN」は、ルールの目的が一部のルールと一致することを示します。
スキャンの形式。

The "ET" indicates the rule came from the Emerging Threats
project. "SCAN" indicates the purpose of the rule is to match on some
form of scanning. Following that a more or less detailed description
is given.

Most rules contain some pointers to more information in the form of
the "reference" keyword.

Consider the following example rule:

::


  alert tcp $HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS \
    (msg:"ET CURRENT_EVENTS Adobe 0day Shovelware"; \
    flow:established,to_server; content:"GET "; nocase; depth:4; \
    content:!"|0d 0a|Referer\:"; nocase; \
    uricontent:"/ppp/listdir.php?dir="; \
    pcre:"/\/[a-z]{2}\/[a-z]{4}01\/ppp\/listdir\.php\?dir=/U"; \
    classtype:trojan-activity; \
    reference:url,isc.sans.org/diary.html?storyid=7747; \
    reference:url,doc.emergingthreats.net/2010496; \
    reference:url,www.emergingthreats.net/cgi-bin/cvsweb.cgi/sigs/CURRENT_EVENTS/CURRENT_Adobe; \
    sid:2010496; rev:2;)

In this rule the reference keyword indicates 3 url's to visit for more
information:

::

  isc.sans.org/diary.html?storyid=7747
  doc.emergingthreats.net/2010496
  www.emergingthreats.net/cgi-bin/cvsweb.cgi/sigs/CURRENT_EVENTS/CURRENT_Adobe

いくつかのルールには"reference：cve、2009-3958;" のような情報が含まれていますが
それらはあなたのお気に入りの検索エンジン使って特定のCVEに関する情報を見つけることができます。

公開されている情報は必ずしも正確というわけではありません、また時にはその情報のすべてが一般に公開されているわけでもありません。 そういった場合はシグネチャサポートチャネルで質問すると、多くの人の助けになります。

doc:`../rule-management/suricata-update` を参照するとルールの詳細やソース、それらのドキュメントとサポート方法が見つかります。

多くの場合、問題を確認するためにトリガーとなったアラートとパケットだけを見るだけでは不十分です。
デフォルトのEVE設定を使用する場合、多くのメタデータがアラートに追加されています。

たとえば、Webアプリケーションが攻撃されたこと示すルールが起動された場合
メタデータを見ると、Webアプリケーションが404のエラーをが返しているだけかもしれません。
通常は攻撃が失敗したのだろうと考えますが、必ずしもそうであるとは限りません。

すべてのプロトコルがメタデータの生成につながるわけではないので、SuricataのようなIDSエンジンを実行する場合は、フルパケットキャプチャと組み合わせることをお勧めします。 Evebox、Sguil、Snorbyなどのツールを使用すると、完全なTCPセッションまたはUDPフローを検査できます。

Obviously there is a lot more to Incidence Response, but this should
get you started.



明らかにインシデントレスポンスにはさらに多くのものがありますが、これは
あなたを始めましょう。
