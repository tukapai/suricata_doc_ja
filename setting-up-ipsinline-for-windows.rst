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


WinDivert.dllとWinDivert.sysは、Suricata実行可能ファイルと同じディレクトリになければなりません。 WinDivertは、実行時に自動的にドライバをインストールします。 WinDivertの詳細については以下をご確認ください。
https://www.reqrypt.org/windivert-doc.html.

To check if you have WinDivert enabled in your Suricata, enter the following
command in an elevated command prompt or terminal:

::

  suricata -c suricata.yaml --windivert [filter string]

For information on the WinDivert filter language, see
https://www.reqrypt.org/windivert-doc.html#filter_language

If Suricata is running on a gateway and is meant to protect the network behind
that gateway, you need to run WinDivert at the NETWORK_FORWARD layer. This can
be achieved using the following command:

::

  suricata -c suricata.yaml --windivert-forward [filter string]

The filter is automatically stopped and normal traffic resumes when Suricata is
stopped.

A quick start is to examine all traffic, in which case you can use the following
command:

::

  suricata -c suricata.yaml --windivert[-forward] true

A few additional examples:

Only TCP traffic:
::

  suricata -c suricata.yaml --windivert tcp

Only TCP traffic on port 80:
::

  suricata -c suricata.yaml --windivert "tcp.DstPort == 80"

TCP and ICMP traffic:
::

  suricata -c suricata.yaml --windivert "tcp or icmp"
