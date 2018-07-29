インストール
============

Suricataを使用する前に、インストールする必要があります。
Suricataはバイナリパッケージ：：ref： `install-binary-packages`を使って、さまざまなディストリビューションでインストールすることができます。

独自のソフトウェアのコンパイルに精通している方へは、Sourceからのインストールをおすすめします。

上級ユーザーは、高度なガイドを確認できます：ref： `install-advanced`を参照してください。

Source
------

ソースファイルからインストールすると、Suricataのインストールを最大限にコントロールすることができます。

Basic steps::

    tar xzvf suricata-4.0.0.tar.gz
    cd suricata-4.0.0
    ./configure
    make
    make install


これにより、Suricataが``/usr/local/bin/``にインストールされます。
デフォルトでは以下に設定とログ出力がされます。
設定ファイル：``/usr/local/etc/suricata/``
ログ出力：``/usr/local/var/log/suricata``


共通の設定オプション
^^^^^^^^^^^^^^^^^^^^^^^^

.. option:: --disable-gccmarch-native

    ハードウェアに対して最適化したビルドをしません。
    バイナリをポータブルなものにする、もしくはSuricataがVMで使用する場合はこのフラグを追加してください。

.. option:: --prefix=/usr/

    Suricataバイナリを``/usr/bin/``にインストールします。
    Default : ``/usr/local/``

.. option:: --sysconfdir=/etc

    ``/var/log/suricata/``にログインするためのSuricataを設定します。
    Default : ``/usr/local/etc/``

.. option:: --localstatedir=/var

    Setups Suricata for logging into /var/log/suricata/.
    Suricataのログが``/var/log/suricata/``に出力されるように設定します。
    Default : ``/usr/local/var/log/suricata``

.. option:: --enable-lua

    Enables Lua support for detection and output.
    検出と出力でLuaサポートを有効にします。

.. option:: --enable-geopip

    Enables GeoIP support for detection.
    検出でGeoIPサポートを有効にします。

.. option:: --enable-rust

    試験的なRustサポートを有効化します。

Dependencies
^^^^^^^^^^^^

Suricataのコンパイルには、以下のライブラリとその開発ヘッダーがインストールされている必要があります：

  libpcap, libpcre, libmagic, zlib, libyaml

次のツールが必要です。:

  make gcc (or clang) pkg-config

完全な機能を追加するには、次も追加します。:

  libjansson, libnss, libgeoip, liblua5.1, libhiredis, libevent

Rust サポート (experimental):

  rustc, cargo

Ubuntu/Debian
"""""""""""""

最小限::

    apt-get install libpcre3 libpcre3-dbg libpcre3-dev build-essential libpcap-dev   \
                    libyaml-0-2 libyaml-dev pkg-config zlib1g zlib1g-dev \
                    make libmagic-dev

推奨::

    apt-get install libpcre3 libpcre3-dbg libpcre3-dev build-essential libpcap-dev   \
                    libnet1-dev libyaml-0-2 libyaml-dev pkg-config zlib1g zlib1g-dev \
                    libcap-ng-dev libcap-ng0 make libmagic-dev libjansson-dev        \
                    libnss3-dev libgeoip-dev liblua5.1-dev libhiredis-dev libevent-dev

Extra for iptables/nftables IPS integration::

    apt-get install libnetfilter-queue-dev libnetfilter-queue1  \
                    libnetfilter-log-dev libnetfilter-log1      \
                    libnfnetlink-dev libnfnetlink0

Rust サポート(Ubuntu only)::

    apt-get install rustc cargo

.. _install-binary-packages:

バイナリパッケージ
---------------

Ubuntu
^^^^^^


Ubuntuの場合、OISFは常に最新の安定リリースを含むPPAの `` suricata-stable``を維持しています。

使用する場合::

    sudo add-apt-repository ppa:oisf/suricata-stable
    sudo apt-get update
    sudo apt-get install suricata

Debian
^^^^^^

Debian 9 (Stretch)で実行::

    apt-get install suricata

Debian Jessie のSuricataは古いですが、更新版はDebian Backports にあります。

Root権限で実行::

    echo "deb http://http.debian.net/debian jessie-backports main" > \
        /etc/apt/sources.list.d/backports.list
    apt-get update
    apt-get install suricata -t jessie-backports

Fedora
^^^^^^

::

    dnf install suricata

RHEL/CentOS
^^^^^^^^^^^

RedHat Enterprise Linux 7およびCentOS 7では、EPELリポジトリを使用できます。

::

    yum install epel-release
    yum install suricata


.. _install-advanced:

高度なインストール
---------------------

GITおよびその他のオペレーティングシステムからインストールするためのさまざまなインストールガイドは以下になります。:
https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Suricata_Installation
