Suricata
========

Introduction
------------

Suricataはネットワーク型 IDS, IPS, NSM エンジンです。


インストール
------------

インストール方法については以下を参照してください。  
https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Suricata_Installation

ユーザーガイド
----------

ユーザーガイドに関しては [Suricata user guide](https://suricata.readthedocs.io/en/latest/)をご参照ください。  

廃止された（まだ有用性のある）ユーザーガイドについてはこちらを確認してください。   [available](https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Suricata_User_Guide).


貢献（コントリビュートについて）
------------

私たちは喜んでパッチの提供やその他の貢献をしています。  
詳細はこちらをご確認ください。 https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Contributing



Suricataは、ほとんどが信頼できない入力を扱う複雑なソフトウェアです。  
この入力を誤って処理すると深刻な結果になります。

* IPSモード中にクラッシュした場合ネットワークが切断される可能性があります。
* IDSパッシブモードで障害が発生した場合、機密データの喪失につながる可能性があります。
* 誤検知によるネットワーク障害を引き起こす可能性があります。


#TODO ここは後で考える  
In other words, we think the stakes are pretty high, especially since in many common cases the IDS/IPS will be directly reachable by an attacker.

この理由のために、我々は、非常に広範囲のQAプロセスを開発しました。Suricataへの貢献の結果が反映されるまでにはいくつかの長いステップを経る必要があります。

高水準のステップとは以下のとおりです。

1. プルリクエストが行われた際に自動的にTravis-CIによるビルドユニットテストが実行されます。

2. 開発者チームとコミュニティによるレビュー

3. QAの実行


### Overview of Suricata's QA steps

信頼できる開発者とコアチームのメンバーは、（半）公開のBuildbotインスタンスにビルドをサブミットできます。 既存の機能が壊れていないことを確認するために、一連のビルドテストとリグレッションスイートを実行されます。

The final QA run takes a few hours minimally, and is started by Victor. It currently runs:

最終的なQAはVictorによって実行され最低でも2.3時間ほど実行されます。

- 異なるOSのもの、コンパイラ、最適化レベル、機能設定を行うextensive build tests
- 静的コード分析を使用したcppcheck、ビルドのチェック
- valgrind、DrMemory、AddressSanitizer、LeakSanitizerを用いた実行時コード分析
- 過去のバグに対するリグレッションテスト
- ログ出力のバリデーション
- UNIXソケット・テスト
- pcapをベースとしたASANとLSANを用いるfuzzテスト。

Next to these tests, based on the type of code change further tests can be run manually:

- トラフィックリプレイ・テスト（マルチ・ギガビット）
- 大規模なpcap収集処理（マルチ・テラバイト）
- AFLをベースとしたfuzzテスト（数日、もしくは数週間必要かもしれない）
- pcapベースのパフォーマンス・テスト
- ライブ・パフォーマンステスト
- various other manual tests based on evaluation of the proposed changes

上記のテストのほとんどが受け入れテストとして使われていることを理解することは非常に重要です。  
もし何か間違いがあったとときのために、あなたのコードの中にアドレスを入れるかどうかはあなた次第です。

QAの1つのステップは、現在、マージ後に実行されます。  
Coverity Scanプログラムにビルドを提出場合は（無料）サービスの制限により、1日1回最大提出できます。  
マージ後にコミュニティが問題を見つけることがあります。 どちらの場合も、問題が発生した場合、対処してください。  



### FAQ

__Q: PRを受け入れますか?__

A：それはコードの品質を含む多くのことに依存します。新機能では、チームやコミュニティがその機能が有用だと思うかどうか、他のコードや機能にどれだけ影響を与えるか、パフォーマンスの低下などのリスクにも依存します。


__Q：PRはいつマージされますか？__

A：それは、それが主要なfeatureであるか、または高いリスクの変化と見なされる場合、おそらく次のメジャーバージョンに入るでしょう。


__Q：私のPRはなぜクローズされたのですか？__

A：Suricata Githubのワークフロー
https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Github_work_flow　に記載されているように、すべての変更に対して新しいプルリクエストが必要です。

通常、チーム（またはコミュニティ）はプルリクエストに関するフィードバックを行い、その後改善されたPRに置き換えられることが期待されます。だからコメントを見てください。あなたがコメントに同意しない場合、私たちはまだ閉じたPRでそれらを議論することができます。

PRがコメントなしで終了した場合、おそらくQAの失敗が原因です。Travis-CIチェックが失敗した場合は、すぐにPRを修正する必要があります。あなたがQAの失敗が間違っていると思わない限り、それについての議論は必要ありません。

__Q：コンパイラ/コードアナライザ/ツールが間違っていますが、今は何ですか？__

A：QAの自動化を支援するために、私たちは滞在するための警告やエラーを受け付けていません。場合によっては、ツールがそれをサポートしている場合（たとえばvalgrind、DrMemory）、抑制を追加することができます。いくつかの警告は無効にすることができます。いくつかの例外的なケースでは、唯一の「解決策」は、コードをリファクタリングして、静的コードチェッカーの制限偽陽性を回避することです。frusterating中、私たちは出力に警告を残す以上にこれを好む。警告は無視され、その後、他の警告を隠すリスクが高くなりがちです。


__Q：あなたのQAテストは間違っていると思う__

A：本当にそうだと思うなら、それを改善する方法について話し合うことができます。  
しかし、すぐにこの結論に至らないようにしてください。それは間違っていることが明らかになります。


__Q：コントリビュータライセンス契約に署名する必要がありますか？__

A：はい、私たちは、Suricataの所有権を一手に保つためにこれを行っています：
オープン情報セキュリティ財団。
http://suricata-ids.org/about/open-source/　および
http://suricata-ids.org/about/contribution-agreement/　を参照してください。
