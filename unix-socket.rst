Interacting via Unix Socket
===========================

Introduction
------------

SuricataはUNIXソケットを聞いて、ユーザーからのコマンドを受け入れることができます。 エクスキシング・プロトコルはJSONベースであり、メッセージのフォーマットは一般的なものになっています。

suricatascと呼ばれるサンプルスクリプトがソースに提供され、Suricataのインストール/更新時に自動的にインストールされます。

libjanssonが利用可能であれば、unixソケットはデフォルトで有効になっています。

You need to have libjansson installed:

* libjansson4 - JSONデータをエンコード、デコード、操作するためのCライブラリ
* libjansson-dev - JSONデータをエンコード、デコード、操作するためのライブラリ(開発版)
* python-simplejson - Pyhon用のシンプルで高速で拡張可能なJSONエンコーダ/デコーダ

Debian/Ubuntu::

   apt-get install libjansson4 libjansson-dev python-simplejson

libjanssonがシステムに存在する場合、unixソケットは自動的にコンパイルされます。

ソケットの作成は、Suricata YAML設定ファイルのunix-commandで 'yes'または 'auto'を有効に設定することによって管理されます:

::

  unix-command:
    enabled: yes
    #filename: custom.socket # use this to specify an alternate file

`` filename``変数は、別のソケットファイル名を設定するために使用できます。 ファイル名は、常にローカル状態のベースディレクトリからの相対パスです。

クライアントはいくつかの言語用に実装されており、カスタムスクリプトを書くためのコード例として使用できます:

* Python: https://github.com/inliniac/suricata/blob/master/scripts/suricatasc/suricatasc.in (provided with suricata and used in this document)
* Perl: https://github.com/aflab/suricatac (a simple Perl client with interactive mode)
* C: https://github.com/regit/SuricataC (a Unix socket mode client in C without interactive mode)

.. _standard-unix-socket-commands:

Commands in standard running mode
---------------------------------
suricatascをインストールしていない場合は、scripts / suricatascから次のコマンドを実行する必要があります。

::

  sudo python setup.py install

既存のコマンドのセットは次のとおりです:

* command-list: list available commands
* shutdown: this shutdown Suricata
* iface-list: list interfaces where Suricata is sniffing packets
* iface-stat: list statistic for an interface
* help: alias of command-list
* version: display Suricata's version
* uptime: display Suricata's uptime
* running-mode: display running mode (workers, autofp, simple)
* capture-mode: display capture system used
* conf-get: get configuration item (see example below)
* dump-counters: dump Suricata's performance counters
* reopen-log-files: reopen log files (to be run after external log rotation)
* ruleset-reload-rules: reload ruleset and wait for completion
* ruleset-reload-nonblocking: reload ruleset and proceed without waiting
* ruleset-reload-time: return time of last reload
* ruleset-stats: display the number of rules loaded and failed
* ruleset-failed-rules: display the list of failed rules
* memcap-set: update memcap value of an item specified
* memcap-show: show memcap value of an item specified
* memcap-list: list all memcap values available

これらのコマンドには、「suricatasc」という名前のサンプルスクリプトを使用してアクセスできます。
 `` suricatasc``との典型的なセッションは以下のようになります:

::

  # suricatasc
  Command list: shutdown, command-list, help, version, uptime, running-mode, capture-mode, conf-get, dump-counters, iface-stat, iface-list, quit
  >>> iface-list
  Success: {'count': 2, 'ifaces': ['eth0', 'eth1']}
  >>> iface-stat eth0
  Success: {'pkts': 378, 'drop': 0, 'invalid-checksums': 0}
  >>> conf-get unix-command.enabled
  Success:
  "yes"

Commands on the cmd prompt
--------------------------

コマンドプロンプトで直接suricatascを使用することができます:

::


  root@debian64:~# suricatasc -c version
  {'message': '2.1beta2 RELEASE', 'return': 'OK'}
  root@debian64:~#
  root@debian64:~# suricatasc -c uptime
  {'message': 35264, 'return': 'OK'}
  root@debian64:~#


**NOTE:**
あなたは複数の引数を含むコマンドを引用する必要があります:

::


  root@debian64:~# suricatasc -c "iface-stat eth0"
  {'message': {'pkts': 5110429, 'drop': 0, 'invalid-checksums': 0}, 'return': 'OK'}
  root@debian64:~#


Pcap processing mode
--------------------

このモードは、このコードの主な動機の1つです。この考えは、ファイル間でSuricataを再起動しなくても、異なるpcapファイルを扱うようにSuricataに依頼することができるようにすることです。シグネチャエンジンが初期化されるのを待つ必要がないため、時間が大幅に増えます。

このモードを使用するには、好きなYAMLファイルでsuricataを起動し、 `` --unix-socket``オプションを引数として指定します:

::

  suricata -c /etc/suricata-full-sigs.yaml --unix-socket

引数としてソケットファイル名を指定することも可能です:

::

  suricata --unix-socket=custom.socket

この最後のケースでは、 `` suricatasc``へのソケットへの完全なパスを提供する必要があります。
 そのためには、 `` suricatasc``の最初の引数としてファイル名を渡す必要があります:

::

  suricatasc custom.socket

Suricataが起動すると、提供されたスクリプト `` suricatasc``を使ってコマンドソケットに接続し、pcap処理を要求することができます:

::

  root@tiger:~# suricatasc
  >>> pcap-file /home/benches/file1.pcap /tmp/file1
  Success: Successfully added file to list
  >>> pcap-file /home/benches/file2.pcap /tmp/file2
  Success: Successfully added file to list
  >>> pcap-file-continuous /home/pcaps /tmp/dirout
  Success: Successfully added file to list

  結果を待たずに複数のファイルを追加することができます。
  生成されたログ/アラートファイルは、pcap-fileコマンドの第2引数として指定されたディレクトリに格納されます。 Suricataがスクリプトが実行された場所からわからないので、ファイルとディレクトリの絶対パスを指定する必要があります。 ファイルの代わりにディレクトリを渡すと、ディレクトリ内のすべてのファイルが処理されます。 `` pcap-file-continuous``を使ってディレクトリを渡すと、 `` pcap-interrupt``を使うかディレクトリを削除/移動するまで、新しいファイルが追加されているかどうか監視されます。

どのファイルが処理待ちになっているかを知るには:

::

  >>> pcap-file-number
  Success: 3

キューに入れられたファイルのリストを取得するには:

::

  >>> pcap-file-list
  Success: {'count': 2, 'files': ['/home/benches/file1.pcap', '/home/benches/file2.pcap']}

処理中のファイルを確認するには:

::

  >>> pcap-current
  Success:
  "/tmp/test.pcap"

ディレクトリを渡すと、最終的に処理された時間（最後のファイルの変更時間）がエポックからミリ秒単位で表示されます。:

::

  >>> pcap-last-processed
  Success:
  1509138964000

現在の状態を終了するディレクトリ処理を中断する:

::

  >>> pcap-interrupt
  Success:
  "Interrupted"

Build your own client
---------------------

プロトコルは次のページに記載されています
https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Unix_Socket#Protocol

The following session show what is send (SND) and received (RCV) by
the server. Initial negotiation is the following:

::

  # suricatasc
  SND: {"version": "0.1"}
  RCV: {"return": "OK"}

これが完了すると、コマンドを発行することができます:

::

  >>> iface-list
  SND: {"command": "iface-list"}
  RCV: {"message": {"count": 1, "ifaces": ["wlan0"]}, "return": "OK"}
  Success: {'count': 1, 'ifaces': ['wlan0']}
  >>> iface-stat wlan0
  SND: {"command": "iface-stat", "arguments": {"iface": "wlan0"}}
  RCV: {"message": {"pkts": 41508, "drop": 0, "invalid-checksums": 0}, "return": "OK"}
  Success: {'pkts': 41508, 'drop': 0, 'invalid-checksums': 0}

pcap-fileモードでは、以下のようになります。:

::

  >>> pcap-file /home/eric/git/oisf/benches/sandnet.pcap /tmp/bench
  SND: {"command": "pcap-file", "arguments": {"output-dir": "/tmp/bench", "filename": "/home/eric/git/oisf/benches/sandnet.pcap"}}
  RCV: {"message": "Successfully added file to list", "return": "OK"}
  Success: Successfully added file to list
  >>> pcap-file-number
  SND: {"command": "pcap-file-number"}
  RCV: {"message": 1, "return": "OK"}
  >>> pcap-file-list
  SND: {"command": "pcap-file-list"}
  RCV: {"message": {"count": 1, "files": ["/home/eric/git/oisf/benches/sandnet.pcap"]}, "return": "OK"}
  Success: {'count': 1, 'files': ['/home/eric/git/oisf/benches/sandnet.pcap']}
  >>> pcap-file-continuous /home/eric/git/oisf/benches /tmp/bench 0 true
  SND: {"command": "pcap-file", "arguments": {"output-dir": "/tmp/bench", "filename": "/home/eric/git/oisf/benches/sandnet.pcap", "tenant": 0, "delete-when-done": true}}
  RCV: {"message": "Successfully added file to list", "return": "OK"}
  Success: Successfully added file to list

注意すべき点が1つあります：Suricataメッセージは複数の送信操作で送信されます。 これにより、クライアント側で不完全な読み取りが行われる可能性があります。 回避策が悪いのは、recvコールを試す前に少し寝ることです。 もう1つの解決策は、非ブロッキングソケットを使用し、前のソケットが失敗した場合にrecvを再試行することです。

Pcap-file json フォーマット:

::

  {
    "command": "pcap-file",
    "arguments": {
      "output-dir": "path to output dir",
      "filename": "path to file or directory to run",
      "tenant": 0,
      "continuous": false,
      "delete-when-done": false
    }
  }

  `output-dir`と` filename`が必要です。 `tenant`はオプションで、ファイルまたはディレクトリがどのテナントで実行されるべきかを示す数字でなければなりません。 `continuous`はオプションであり、真または偽でなければならず、ファイルまたはディレクトリを指定する必要があります。
  `pcap-interrupt`が送信されるか、ctrl-cが呼び出されるまで実行されます。 `delete-when-done`はオプションであり、処理が完了したときに` filename`で指定されたディレクトリの下のファイルを削除する必要があることを示すtrue / falseでなければなりません。 `delete-when-done`のデフォルトはfalseで、ファイルは処理後も保持されます。
