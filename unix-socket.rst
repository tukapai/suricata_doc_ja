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

You can access to these commands with the provided example script which
is named ``suricatasc``. A typical session with ``suricatasc`` will looks like:

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

You can use suricatasc directly on the command prompt:

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

To use this mode, start suricata with your preferred YAML file and
provide the option ``--unix-socket`` as argument:

::

  suricata -c /etc/suricata-full-sigs.yaml --unix-socket

It is also possible to specify the socket filename as argument:

::

  suricata --unix-socket=custom.socket

In this last case, you will need to provide the complete path to the
socket to ``suricatasc``. To do so, you need to pass the filename as
first argument of ``suricatasc``:

::

  suricatasc custom.socket

Once Suricata is started, you can use the provided script
``suricatasc`` to connect to the command socket and ask for pcap
treatment:

::

  root@tiger:~# suricatasc
  >>> pcap-file /home/benches/file1.pcap /tmp/file1
  Success: Successfully added file to list
  >>> pcap-file /home/benches/file2.pcap /tmp/file2
  Success: Successfully added file to list
  >>> pcap-file-continuous /home/pcaps /tmp/dirout
  Success: Successfully added file to list

You can add multiple files without waiting the result: they will be
sequentially processed and the generated log/alert files will be put
into the directory specified as second arguments of the pcap-file
command. You need to provide absolute path to the files and directory
as Suricata doesn’t know from where the script has been run. If you pass
a directory instead of a file, all files in the directory will be processed. If
using ``pcap-file-continuous`` and passing in a directory, the directory will
be monitored for new files being added until you use ``pcap-interrupt`` or
delete/move the directory.

To know how many files are waiting to get processed, you can do:

::

  >>> pcap-file-number
  Success: 3

To get the list of queued files, do:

::

  >>> pcap-file-list
  Success: {'count': 2, 'files': ['/home/benches/file1.pcap', '/home/benches/file2.pcap']}

To get current processed file:

::

  >>> pcap-current
  Success:
  "/tmp/test.pcap"

When passing in a directory, you can see last processed time (modified time of last file) in milliseconds since epoch:

::

  >>> pcap-last-processed
  Success:
  1509138964000

To interrupt directory processing which terminates the current state:

::

  >>> pcap-interrupt
  Success:
  "Interrupted"

Build your own client
---------------------

The protocol is documented in the following page
https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Unix_Socket#Protocol

The following session show what is send (SND) and received (RCV) by
the server. Initial negotiation is the following:

::

  # suricatasc
  SND: {"version": "0.1"}
  RCV: {"return": "OK"}

Once this is done, command can be issued:

::

  >>> iface-list
  SND: {"command": "iface-list"}
  RCV: {"message": {"count": 1, "ifaces": ["wlan0"]}, "return": "OK"}
  Success: {'count': 1, 'ifaces': ['wlan0']}
  >>> iface-stat wlan0
  SND: {"command": "iface-stat", "arguments": {"iface": "wlan0"}}
  RCV: {"message": {"pkts": 41508, "drop": 0, "invalid-checksums": 0}, "return": "OK"}
  Success: {'pkts': 41508, 'drop': 0, 'invalid-checksums': 0}

In pcap-file mode, this gives:

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

There is one thing to be careful about: a Suricata message is sent in
multiple send operations. This result in possible incomplete read on
client side. The worse workaround is to sleep a bit before trying a
recv call. An other solution is to use non blocking socket and retry a
recv if the previous one has failed.

Pcap-file json format is:

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

`output-dir` and `filename` are required. `tenant` is optional and should be a
number, indicating which tenant the file or directory should run under. `continuous`
is optional and should be true/false, indicating that file or directory should be
run until `pcap-interrupt` is sent or ctrl-c is invoked. `delete-when-done` is
optional and should be true/false, indicating that the file or files under the
directory specified by `filename` should be deleted when processing is complete.
`delete-when-done` defaults to false, indicating files will be kept after
processing.
