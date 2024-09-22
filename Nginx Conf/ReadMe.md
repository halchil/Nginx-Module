# はじめに
nginx.confの設定を行う

nginxのconfファイルは、**Nginx設定ファイル形式**と呼ばれる特別な形式で記載される。

## ディレクティブとセミコロン

設定項目は「ディレクティブ」と呼ばれ、通常はセミコロン（;）で終わります。ディレクティブは設定の具体的な指示を表す。

```
listen 80;
server_name localhost;

```

## ブロックとコンテキスト

ディレクティブを階層的に整理するために、「ブロック」が使用される。

ブロックは中括弧 {} で囲まれ、特定のコンテキスト（例えば http、server、location）内で適用される。

## 階層構造

設定は階層的に構造化されており、上位のコンテキストが下位のコンテキストに影響を与える。

例えば、http ブロック内に複数の server ブロックを配置し、各 server ブロック内に複数の location ブロックを配置することができる。


# 参考文献


# head.yaml
[head.conf](https://github.com/halchil/Nginx-Module/blob/main/Nginx%20Conf/head.conf)

## user nginx
Nginxがどのユーザー権限で実行されるかを指定する。

nginx はNginxのプロセスが動作する際に使われるLinuxユーザー名である。
セキュリティのため、Nginxをルートユーザー（管理者権限）で実行しないのが一般的。
通常、nginx という専用ユーザーが作成され、そのユーザーの権限でNginxが動作する。
これにより、Nginxがシステム全体に過度なアクセス権限を持つことを防ぐ。

### 確認方法1 ps コマンドを使用する

ps コマンドでNginxのプロセスとそれを実行しているユーザーを確認することができます。

```
[実行コマンド]
ps aux | grep nginx

[結果]
nginx      1234  0.0  0.1 123456  1234 ?        Ss   10:00   0:00 nginx: master process /usr/sbin/nginx
nginx      1235  0.0  0.1 123456  1234 ?        S    10:00   0:00 nginx: worker process
```

### 確認方法2 pgrep と ps を組み合わせる

pgrep を使ってNginxのプロセスIDを取得する。

```
[実行コマンド]
pgrep nginx
```

そのPIDを使って、ps コマンドで実行ユーザーを確認する。

```
[実行コマンド]
ps -o user= -p <PID>
```

<PID> は pgrep nginx で得られたプロセスIDを指定する。


### 確認方法3 systemctl で確認する（システムが systemd を使用している場合）

systemctl でNginxサービスの実行ユーザーを確認する方法。
通常、systemd 経由でNginxが起動されている場合は、このコマンドで確認できる。

```
[実行コマンド]
systemctl status nginx

[結果]
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-09-22 10:00:00 JST; 1h ago
  Process: 1234 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
 Main PID: 1234 (nginx)
    Tasks: 2 (limit: 4915)
   Memory: 5.5M
   CGroup: /system.slice/nginx.service
           ├─1234 nginx: master process /usr/sbin/nginx
           └─1235 nginx: worker process

```

上記の Main PID を使ってユーザーを確認できる。

## worker_processes auto
Nginxが使用するワーカープロセスの数を指定する。

ワーカープロセスは、クライアントからのリクエストを処理するためのプロセスである。

auto に設定すると、NginxはサーバーのCPUコア数に基づいて最適なワーカープロセス数を自動的に決定する。
たとえば、4コアのCPUがある場合、Nginxは4つのワーカープロセスを生成できる。

ワーカープロセスの数は、負荷やサーバーの性能によって調整が可能で、リクエストの処理能力に影響を与える。

## error_log /var/log/nginx/error.log warn
Nginxのエラーログファイルのパスとログレベルを指定する。

`/var/log/nginx/error.log` は、Nginxのエラーログを保存するファイルのパスである。

warn は、ログレベルを示している。
Nginxが記録するエラーログのレベルを制御し、ログレベルには以下の段階がある。

- debug: デバッグ用の詳細なログを記録
- info: 情報レベルのログ
- notice: 通知レベルのログ
- warn: 警告レベルのログ
- error: エラーが発生した場合のみ記録
- crit: クリティカルなエラー（重大なエラー）
 
 ここでは warn が指定されているので、警告以上のエラー（警告、エラー、クリティカルエラー）が発生した際にログが記録される。

## pid /var/run/nginx.pid
NginxのプロセスID（PID）ファイルの保存場所を指定する。

`/var/run/nginx.pid` は、NginxのプロセスIDが記録されるファイルである。

PIDは、現在実行中のプロセスに一意に割り当てられるIDで、システムがプロセスを管理する際に使われる。
Nginxの管理操作（再起動、停止など）を行う際に、このファイルからPIDを読み取り、対応するプロセスに対して操作を行う。

## head全体の役割

セキュリティ: user nginx; で非管理者権限のユーザーを指定することで、サーバーのセキュリティを向上させる。

パフォーマンス: worker_processes auto; により、サーバーのCPU性能に合わせた最適なリクエスト処理が可能になる。

トラブルシューティング: error_log でエラーログの保存先と詳細度を指定し、問題発生時に原因を追跡しやすくする。

プロセス管理: pid ファイルは、Nginxの制御を行うために重要となる。

# events.conf


events ブロックは、Nginxがリクエストをどのように処理するかに関する設定を行う部分である。

Nginxは、イベント駆動型のサーバーであり、同時に多数のクライアントリクエストを処理するために効率的なイベント処理が重要となる。

worker_connections は、各ワーカープロセスが同時に処理できる最大の接続数を指定する設定である。

Nginxは通常、複数の「ワーカープロセス」を使ってリクエストを処理しますが、その各プロセスが同時に対応できるクライアント接続の数を設定します。

```
events {
    worker_connections  1024;
}
```

この設定は「1つのワーカープロセスあたり、1024の同時接続を処理できる」ことを意味する。

# http.conf

## MIMEタイプの設定

```
include /etc/nginx/mime.types;
default_type application/octet-stream;
```

**include /etc/nginx/mime.types;**

このディレクティブは、MIMEタイプの定義ファイルを読み込む。

MIMEタイプ（Multipurpose Internet Mail Extensions）は、ファイルの種類を表すための形式である。
これは、ウェブサーバーがブラウザに対して「このファイルはどんな種類か」を教える役割を持っている。
例えば、画像なのか、動画なのか、テキストファイルなのか、ということを伝える。

`mime.types` ファイルには、拡張子とそれに対応するMIMEタイプのマッピングが記述されている。

これにより、Nginxはクライアントに返すコンテンツの種類を適切に設定できる。

`mime.types`がどこにあって、どのような記述なのか気になるので調べてみる。

以下のような形式で記載されている。

最初の列: MIMEタイプ
次の列: 拡張子（スペースで区切られている場合もあり）

```
# mime.types
# 確認できる MIME タイプの定義
text/html          html htm
text/css           css
text/javascript    js
application/javascript js
application/json    json
```


**default_type application/octet-stream;**

特定の拡張子に対応するMIMEタイプが見つからない場合に使用されるデフォルトのMIMEタイプを指定する。
`application/octet-stream` は汎用的なバイナリデータとして扱われます。



## ログフォーマットの設定

```
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';
```

**log_format mainについて**

main という名前のログフォーマットを定義しており、ログには以下の情報が含まれる。

- $remote_addr: クライアントのIPアドレス
- $remote_user: 認証されたユーザー名
- $time_local: リクエストを受けた日時
- $request: リクエストライン（メソッド、URI、プロトコル）
- $status: HTTPステータスコード
- $body_bytes_sent: 送信されたボディのバイト数
- $http_referer: リファラー情報
- $http_user_agent: ユーザーエージェント情報
- $http_x_forwarded_for: プロキシ経由の場合の元のクライアントIPアドレス

※気になること
他にどのような変数があるか、全部をログに記載することはできるか？
(一旦、他のものはChatGPTに聞くことで全体像を掴む)


## アクセスログの設定

```
access_log /var/log/nginx/access.log main;
```

アクセスログの出力先を /var/log/nginx/access.log に設定し、先ほど定義した main フォーマットを使用する。

## ファイル送信および接続維持の設定

```
sendfile on;
keepalive_timeout 65;
```
**sendfile onについて**

sendfile を有効にすると、カーネルの sendfile システムコールを使用して高速にファイルを送信できる。
これにより、ユーザースペースとカーネルスペース間のデータコピーが削減され、パフォーマンスが向上する。

keepalive_timeout 65;
クライアントとのアイドル状態の接続を65秒間維持する。
これにより、同一クライアントからの複数リクエストが同じ接続で処理され、接続の再確立によるオーバーヘッドが減少する。

※気になること
カーネルの sendfile システムコールとは？クライアントとのアイドル状態で65秒間で何が変わるかテストしたい。


## サーバーブロックの設定

```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

**listen 80; について**
サーバーがポート80でリクエストを待ち受ける。
ポート80はHTTPのデフォルトポート。

**server_name localhost;について**
サーバーの名前を localhost に設定する。
これにより、localhost というホスト名でアクセスされたリクエストをこのサーバーブロックが処理します。
→ここは仮想マシンのIPに変更しないといけないかも

## ロケーションブロックの設定

```
location / {
    root   /usr/share/nginx/html;
    index  index.html;
}
```
**location / { ... } について**
ルートパス / に対するリクエストを処理する。


```
root /usr/share/nginx/html;
index index.html;
```

ドキュメントルートを /usr/share/nginx/html に設定する。
つまり、http://localhost/ へのリクエストは /usr/share/nginx/html/index.html を返す。


ディレクトリへのリクエスト（例：http://localhost/）時にデフォルトで index.html を返すよう指定する。


つまり、`docker-compose.yaml`で指定したvolumeとディレクトリは一致させておかなければならない。


## エラーページの設定

```
error_page 500 502 503 504 /50x.html;
location = /50x.html {
    root /usr/share/nginx/html;
}
```

**error_page 500 502 503 504 /50x.html;について**
サーバー内部エラー（500系エラー）が発生した場合に /50x.html を表示するよう指定する。

**location = /50x.html { ... }について**
/50x.html への正確なリクエストを処理する。


**root /usr/share/nginx/html;について**
エラーページのファイル 50x.html が /usr/share/nginx/html ディレクトリに存在することを指定します。



# ケースに即した確認テスト

## Nginx実行ユーザの指定方法と確認方法
どういうシチュエーションで必要になるだろうか。

# 気になることリスト
カーネルの sendfile システムコールとは？クライアントとのアイドル状態で65秒間で何が変わるかテストしたい。

他にどのような変数があるか、全部をログに記載することはできるか？
