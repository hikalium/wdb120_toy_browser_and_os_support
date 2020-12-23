<h1>自作OS×自作ブラウザで学ぶ<br>Webページが表示されるまで<br><small>サポートページ</small></h1>

<h2 id="about"><a href="#about">About</a></h2>

このページは、[Web+DB Press Vol.120](https://gihyo.jp/magazine/wdpress/archive/2021/vol120)に掲載されている記事「自作OS×自作ブラウザで学ぶ Webページが表示されるまで」に関する補足情報をまとめたページです。

本ページの内容に関するお問い合わせは、著者である[@hikalium](https://github.com/hikalium)および[@d0iasm](https://github.com/d0iasm)まで、[Issues](https://github.com/hikalium/wdb120_toy_browser_and_os_support/issues)経由でお願いします。

<h2 id="notation"><a href="#notation">プロンプトの表記について</a></h2>

誌面および本ページ内では、各コマンドの前に、どの環境でそのコマンドを実行するべきなのかわかるよう、プロンプトをつけている場合があります。

```
$ something                 # 通常のLinuxホスト上で実行するコマンド
(host)$ something           # ホスト上で実行することを明確にしたい場合
(liumos)$ something         # liumOS上で実行するコマンド
(liumos-builder)$ something # Dockerインスタンス上のLinuxで実行するコマンド
```

このシェル（`$`以前）の部分は実際には入力する必要はありません。

<h2 id="sections"><a href="#sections">各章ごとの補足</a></h2>

<h3 id="prerequisites"><a href="#prerequisites">事前準備</a></h3>

liumOSのレポジトリは[github:hikalium/liumos](https://github.com/hikalium/liumos)から取得することができます。

```bash
(host) git clone https://github.com/hikalium/liumos.git
```

以下の各コマンドについては、[wdb_120](https://github.com/hikalium/liumos/tree/wdb_120)ブランチで動作することを確認しています。

環境構築の手間を省くため、Dockerを利用する方法を誌面では説明しています。Dockerのインストール手順については[公式ページの情報](https://docs.docker.com/engine/install/)をご参照ください。

Docker環境の外でアプリケーションやOSをビルドする場合には、[README.md](https://github.com/hikalium/liumos/blob/wdb_120/README.md)に記載の環境構築を事前に行ってください。

<h3 id="ch1"><a href="#ch1">第1章</a></h3>

Dockerのインストールについては[事前準備](#prerequisites)を参照してください。

Dockerなしで実験したい場合には、README.mdの[Setup tap interface (for linux)](https://github.com/hikalium/liumos#setup-tap-interface-for-linux)の項に記載の手順で設定を行ってください。ただし、この方法はコンピューターのネットワーク設定を変更するため、特にリモートマシンで試される際には細心の注意を払って行ってください。

<h3 id="ch2"><a href="#ch2">第2章</a></h3>

この章で実装する`ping.bin`のソースコードは[app/ping/ping.c](https://github.com/hikalium/liumos/tree/wdb_120/app/ping/ping.c)から確認できます。

liumOS上で動作確認をしたい場合は

```
(host) make run_docker
```

を実行してliumOSを起動します。初回起動時は、Dockerコンテナのイメージをダウンロードするため、多少時間がかかりますが、次回以降は数秒で完了します。

次に、起動したliumOSのシリアルコンソールに、以下のコマンドでアタッチします。

```
(host) telnet localhost 1235
```

接続した直後はプロンプトが見えませんが、Enterキーを押すと以下のようにプロンプトが出てくるはずです。（出ない場合は自作OSの起動に失敗している可能性がありますので、再度`make run_docker`を実行してください。）

```
$ telnet localhost 1235
Trying ::1...
Connected to localhost.
Escape character is '^]'.
# ここでEnterキーを押す

Not a command or file: 
(liumos)$ # プロンプトが出てきた
```

その後、pingコマンドを実行してみます。

```
(liumos)$ ping.bin 10.0.2.2
ping.bin 10.0.2.2
Ping to 10.0.2.2...
kernel: sys_socket: socket (fd=3) created (IPv4, DGRAM, ICMP)
kernel: dst is in the same subnet.
kernel: ARP request sent to 10.0.2.2...
kernel: ARP entry found!
recvfrom returned: 8
00 00 FF FF 00 00 00 00 
ICMP packet recieved from 10.0.2.2 ICMP Type = 0
```

うまくいかない場合は、一度すべてのシェルを閉じて、最初の`make run_docker`のステップからやり直してみてください。


<h3 id="ch3"><a href="#ch3">第3章</a></h3>

この章で言及するソースコードの全体像は、下記から確認できます。

- `udpserver.bin`
  - [app/udpserver/udpserver.c](https://github.com/hikalium/liumos/blob/wdb_120/app/udpserver/udpserver.c)
- `udpclient.bin`
  - [app/udpclient/udpclient.c](https://github.com/hikalium/liumos/blob/wdb_120/app/udpclient/udpclient.c)
- システムコールハンドラの実装
  - [src/syscall.cc](https://github.com/hikalium/liumos/blob/wdb_120/src/syscall.cc)
- virtio-netドライバの実装
  - [src/virtio_net.cc](https://github.com/hikalium/liumos/blob/wdb_120/src/virtio_net.cc)
  - [src/virtio_net.h](https://github.com/hikalium/liumos/blob/wdb_120/src/virtio_net.h)
- ネットワーク関連の実装（本文では触れていないが参考までに）
  - [src/network.cc](https://github.com/hikalium/liumos/blob/wdb_120/src/network.cc)
  - [src/network.h](https://github.com/hikalium/liumos/blob/wdb_120/src/network.h)


(liumos) udpclient.bin -> (Linux) udpserver.bin
```
(host)$ make run_docker
(host)$ docker exec -it liumos-builder0 /bin/bash
(liumos-builder)$  app/udpserver/udpserver.bin 8888
(host)$ telnet localhost 1235
(liumos)$ udpclient.bin 10.0.2.2 8888 Hello!
```

(liumos)$ udpserver.bin <- (Linux) udpclient.bin
```
(host)$ make run_docker
(host)$ docker exec -it liumos-builder0 /bin/bash
(host)$ telnet localhost 1235
(serial)$ udpserver.bin 8889
(liumos-builder)$ app/udpclient/udpclient.bin 127.0.0.1 8889 hello
```

<h3 id="ch4"><a href="#ch4">第4章</a></h3>

第4章で使用するソースコードは以下の通りです。

- `httpserver.bin`
  - [app/httpserver/httpserver.c](https://github.com/hikalium/liumos/blob/master/app/httpserver/httpserver.c)
- `httpclient.bin`
  - [app/httpclient/httpclient.c](https://github.com/hikalium/liumos/blob/master/app/httpclient/httpclient.c)

#### liumOS上の`httpclient.bin`と、Linux上の`httpserver.bin`の通信

```sh
(host)$ make run_docker

// server
(host)$ docker exec -it liumos-builder0 /bin/bash
(liumos-builder)$ /liumos/app/httpserver/httpserver.bin --port 8888

// client
(host)$ telnet localhost 1235
(liumos)$ httpclient.bin --ip 10.0.2.2 --port 8888 --path /index.html
```

#### Linux上の`httpclient.bin`と、liumOS上の`httpserver.bin`の通信

```sh
// server
(host)$ telnet localhost 1235
(liumos)$ httpserver.bin --port 8888

// client
(host)$ docker exec -it liumos-builder0 /bin/bash
(liumos-builder)$ /liumos/app/httpclient/httpclient.bin --ip 127.0.0.1 --port 8888 --path /index.html
```

#### ローカルのLinux環境で試す場合

```sh
// server
$ ./httpserver.bin --port 8888

// client
$ ./httpclient.bin --ip 127.0.0.1 --port 8888 --path /index.html
```

もし、ポート番号が他のアプリケーションですでに使用されていると、`Error: Failed to bind a socket`というメッセージが出ます。その場合は`--port`で指定するポート番号を変えてください。

#### 訂正

第4章でHTTPサーバを実装するさいのリクエストを待ち受けるコードに間違いがありました。`address`変数を`bind()`する必要があります。

間違い（誌面）

```c
address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;
address.sin_port = htons(port);

recvfrom(socket_fd, request, SIZE_REQUEST, 0,
         (struct sockaddr*) &address, &addrlen);
```

訂正

```c
address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;
address.sin_port = htons(port);

// 訂正箇所: bind関数によって、address変数の情報とsocketを結びつける必要があります。
if (bind(socket_fd, (struct sockaddr *) &address, addrlen) < 0) {
  Println("Error: Failed to bind a socket");
  exit(EXIT_FAILURE);
}

// 補足: recvfrom関数によってaddress変数はクライアントの情報へと上書きされます。後にsendto関数によってクライアントへレスポンスを送る際にaddress変数を再度使用します。
recvfrom(socket_fd, request, SIZE_REQUEST, 0,
         (struct sockaddr*) &address, &addrlen);
```

<h3 id="ch5"><a href="#ch5">第5章</a></h3>

第5章で使用するソースコードは以下の通りです。

- `browser.bin`
  - [app/browser/browser.c](https://github.com/hikalium/liumos/blob/master/app/browser/browser.c): コマンドライン引数やURLをパースしたり、HTTPクライアントの役割をする。`main`関数を含む。
  - [app/browser/tokenize.c](https://github.com/hikalium/liumos/blob/master/app/browser/tokenize.c): HTML文字列のトークナイズを行う。
  - [app/browser/parse.c](https://github.com/hikalium/liumos/blob/master/app/browser/parse.c): トークン列からツリーを構築する。
  - [app/browser/rendering.c](https://github.com/hikalium/liumos/blob/master/app/browser/rendering.c): ツリーを走査してMarkdownを出力する。
- - `httpserver.bin`
  - [app/httpserver/httpserver.c](https://github.com/hikalium/liumos/blob/master/app/httpserver/httpserver.c): 第4章で使用したものと同じ。

#### liumOS上のブラウザとLinux上のサーバで試す場合

```sh
// run docker
(host)$ make run_docker

// server
(host)$ docker exec -it liumos-builder0 /bin/bash
(liumos-builder)$ /liumos/app/httpserver/httpserver.bin --port 8888

// client
(host)$ telnet localhost 1235
(liumos)$ browser.bin --url http://10.0.2.2:8888/index.html
```

その他のURLを使用した`browser.bin`のコマンド
- `browser.bin --url http://10.0.2.2:8888/page1.html`
- `browser.bin --url http://10.0.2.2:8888/page2.html`
- `browser.bin --url http://10.0.2.2:8888/not_exist.html`

#### ローカルのLinux環境で試す場合

```sh
// server
$ ./httpserver.bin --port 8888

// client
$ ./browser.bin --url http://127.0.0.1:8888/index.html
```

もし、ポート番号が他のアプリケーションですでに使用されていると、`Error: Failed to bind a socket`というメッセージが出ます。その場合は`--port`で指定するポート番号を変えてください。

## 既知の問題

### Docker上でping.binが動作しない

```
$ docker exec -it liumos-builder0 /bin/bash
(liumos-builder)$ app/ping/ping.bin 8.8.8.8
Ping to 8.8.8.8...
socket() failed
```

おそらく筆者が用意したDocker環境の問題です。解決策が見つかり次第修正します。

それまでは、Linux環境でDockerを使わずにliumOSの環境を整備して直接ビルドして実験するか、Linux上で動くかどうかのテストはスキップして、自作OS上でping.binを実行してみてください。(自作OS上で動かす方が簡単だなんて不思議ですね…。)

### Virtio::Net not initialized yetというエラーでOSが起動しないんだけど…

自作OS側のバグで、現在修正中です。

[liumos/issues/54](https://github.com/hikalium/liumos/issues/54)

再起動すれば多くの場合でうまくいくはずです。お手数をおかけしますが、もう一度お試しください。（もしくは、自作OSのデバッグの大変さを味わうというのもまた一興でしょう。）

<h2><a href="#author">Author</a></h2>

- 第1章-第3章
  - [@hikalium](https://github.com/hikalium)
- 第4章-第5章
  - [@d0iasm](https://github.com/d0iasm)

<h2><a href="#license">License</a></h2>

- このページの内容: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ja)
- このページのソース: [MIT License](https://github.com/hikalium/wdb120_toy_browser_and_os_support/blob/main/LICENSE)
  - Based on [jekyll/minima](https://github.com/jekyll/minima)
