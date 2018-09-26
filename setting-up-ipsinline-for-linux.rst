Setting up IPS/inline for Linux
================================

このガイドでは、レイヤ3インラインモードでSuricataを使用する方法と、その目的のためにiptablesを設定する方法について説明します。

NFQをサポートしたSuricataのコンパイルから始めます。 指示
`Ubuntuのインストール
<https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Ubuntu_Installation>`_.
For more information about NFQ and iptables, see
:ref:`suricata-yaml-nfq`.

SuricataでNFQが有効になっているかどうかを確認するには、次のコマンドを入力します。


::


  suricata --build-info

  フィーチャ間にNFQがあるかどうかを調べます。

  NFQモードでsuricataを実行するには、-qオプションを使用する必要があります。 このオプションは、Suricataにどのキュー番号を使用するかを指示します。


::


  sudo suricata -c /etc/suricata/suricata.yaml -q 0


Iptables configuration
~~~~~~~~~~~~~~~~~~~~~~

まず、Suricataに送信したいトラフィックを知ることが重要です。 コンピュータから送信されたトラフィックまたはコンピュータによって生成されたトラフィック。


.. image:: setting-up-ipsinline-for-linux/IPtables.png

.. image:: setting-up-ipsinline-for-linux/iptables1.png

If Suricata is running on a gateway and is meant to protect the computers behind that gateway you are dealing with the first scenario: *forward_ing* .
If Suricata has to protect the computer it is running on, you are dealing with the second scenario: *host* (see drawing 2).
These two ways of using Suricata can also be combined.

Suricataにトラフィックを送信するゲートウェイシナリオの場合の最も簡単なルールは次のとおりです。:


::


  sudo iptables -I FORWARD -j NFQUEUE

In this case, all forwarded traffic goes to Suricata.


In case of the host situation, these are the two most simple iptable rules;


::


  sudo iptables -I INPUT -j NFQUEUE
  sudo iptables -I OUTPUT -j NFQUEUE

It is possible to set a queue number. If you do not, the queue number will be 0 by default.

Imagine you want Suricata to check for example just TCP-traffic, or all incoming traffic on port 80, or all traffic on destination-port 80, you can do so like this:


::


  sudo iptables -I INPUT -p tcp  -j NFQUEUE
  sudo iptables -I OUTPUT -p tcp -j NFQUEUE

In this case, Suricata checks just TCP traffic.


::


  sudo iptables -I INPUT -p tcp --sport 80  -j NFQUEUE
  sudo iptables -I OUTPUT -p tcp --dport 80 -j NFQUEUE

In this example, Suricata checks all input and output on port 80.

.. image:: setting-up-ipsinline-for-linux/iptables2.png

.. image:: setting-up-ipsinline-for-linux/IPtables3.png

To see if you have set your iptables rules correct make sure Suricata is running and enter:

::


  sudo iptables -vnL

In the example you can see if packets are being logged.

.. image:: setting-up-ipsinline-for-linux/iptables_vnL.png

This description of the use of iptables is the way to use it with IPv4. To use it with IPv6 all previous mentioned commands have to start with 'ip6tables'. It is also possible to let Suricata check both kinds of traffic.

There is also a way to use iptables with multiple networks (and interface cards). Example:


.. image:: setting-up-ipsinline-for-linux/iptables4.png


::


  sudo iptables -I FORWARD -i eth0 -o eth1 -j NFQUEUE
  sudo iptables -I FORWARD -i eth1 -o eth0 -j NFQUEUE

The options -i (input) -o (output) can be combined with all previous mentioned options

If you would stop Suricata and use internet, the traffic will not come through. To make internet work correctly, you have to erase all iptable rules.

To erase all iptable rules, enter:


::


  sudo iptables -F
