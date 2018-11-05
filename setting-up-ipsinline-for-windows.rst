Setting up IPS/inline for Windows
=================================

このガイドでは、WinDivert for Windowsを使用してSuricataをレイヤ4インラインモードで使用する方法について説明します

最初にWinDivertのサポートでSuricataをコンパイルして起動します。,
`Windows Installation
<https://redmine.openinfosecfoundation.org/attachments/download/1175/SuricataWinInstallationGuide_v1.4.3.pdf>`_.
このドキュメントはWinDivert情報で更新されていないため、make
`configure`に以下のフラグを必ず付け加えてください。:

::

  --enable-windivert=yes --with-windivert-include=<include-dir> --with-windivert-libraries=<libraries-dir>


WinDivert.dllとWinDivert.sysは、Suricata実行可能ファイルと同じディレクトリになければなりません。
WinDivertは、実行時に自動的にドライバをインストールします。 WinDivertの詳細については以下をご確認ください。
https://www.reqrypt.org/windivert-doc.html.

To check if you have WinDivert enabled in your Suricata, enter the following
command in an elevated command prompt or terminal:

::

  suricata -c suricata.yaml --windivert [filter string]

For information on the WinDivert filter language, see
https://www.reqrypt.org/windivert-doc.html#filter_language

Suricataがゲートウェイ上で動作していて、ネットワークを保護するためのものである場合
そのゲートウェイでは、NETWORK_FORWARDレイヤーでWinDivertを実行する必要があります。
次のコマンドを使用して達成することができます:

SuricataでWinDivertが有効になっているかどうかを確認するには、次のように入力します
昇格したコマンドプロンプトまたは端末のコマンド：

::

  suricata -c suricata.yaml --windivert [フィルタ文字列]

WinDivertフィルタ言語の詳細については、次を参照してください。
https://www.reqrypt.org/windivert-doc.html#filter_language

Suricataがゲットウェイ上で動作していて、ネットワークを保護するためのものである場合
そのゲームでは、NETWORK_FORWARDレイヤーでWinDivertを実行する必要があります。
次のコマンドを使用して達成することができます：

::

  suricata -c suricata.yaml --windivert-forward [フィルタ文字列]

Suricataが自動的に停止すると、フィルタは自動的に停止し、通常のトラフィックが再開されます。
停止。

クイックスタートは、すべてのトラフィックを調べることです。その場合、次のものを使用できます
コマンド：

::

  suricata -c suricata.yaml --windivert [-forward] true

いくつかの追加例：

TCPトラフィックのみ：
::

  suricata -c suricata.yaml --windivert tcp

ポート80上のTCPトラフィックのみ：
::

  suricata -c suricata.yaml --windivert "tcp.DstPort == 80"

TCPおよびICMPトラフィック：
::

  suricata -c suricata.yaml --windivert "tcp or icmp"
