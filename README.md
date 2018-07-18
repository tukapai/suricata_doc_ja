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

上記のテストのほとんどが受け入れテストとして使われていることを理解することは非常に重要です。もし何か間違いがあったとときのために、あなたのコードの中にアドレスを入れるかどうかはあなた次第ですが。

One step of the QA is currently run post-merge. We submit builds to the Coverity Scan program. Due to limitations of this (free) service, we can submit once a day max.
Of course it can happen that after the merge the community will find issues. For both cases we request you to help address the issues as they may come up.




### FAQ

__Q: Will you accept my PR?__

A: That depends on a number of things, including the code quality. With new features it also depends on whether the team and/or the community think the feature is useful, how much it affects other code and features, the risk of performance regressions, etc.


__Q: When will my PR be merged?__

A: It depends, if it's a major feature or considered a high risk change, it will probably go into the next major version.


__Q: Why was my PR closed?__

A: As documented in the Suricata Github workflow here https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Github_work_flow, we expect a new pull request for every change.

Normally, the team (or community) will give feedback on a pull request after which it is expected to be replaced by an improved PR. So look at the comments. If you disagree with the comments we can still discuss them in the closed PR.

If the PR was closed without comments it's likely due to QA failure. If the Travis-CI check failed, the PR should be fixed right away. No need for a discussion about it, unless you believe the QA failure is incorrect.


__Q: the compiler/code analyser/tool is wrong, what now?__

A: to assist in the automation of the QA, we're not accepting warnings or errors to stay. In some cases this could mean that we add a suppression if the tool supports that (e.g. valgrind, DrMemory). Some warnings can be disabled. In some exceptional cases the only 'solution' is to refactor the code to work around a static code checker limitation false positive. While frusterating, we prefer this over leaving warnings in the output. Warnings tend to get ignored and then increase risk of hiding other warnings.


__Q: I think your QA test is wrong__

A: If you really think it is, we can discuss how to improve it. But don't come to this conclusion to quickly, moreoften it's the code that turns out to be wrong.


__Q: do you require signing of a contributor license agreement?__

A: Yes, we do this to keep the ownership of Suricata in one hand: the Open Information Security Foundation. See http://suricata-ids.org/about/open-source/ and http://suricata-ids.org/about/contribution-agreement/
