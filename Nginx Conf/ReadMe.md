# はじめに
nginx.confの設定を行う

# 参考文献

# head.yaml

## user nginx
説明: Nginxがどのユーザー権限で実行されるかを指定する。

nginx はNginxのプロセスが動作する際に使われるLinuxユーザー名である。
セキュリティのため、Nginxをルートユーザー（管理者権限）で実行しないのが一般的。
通常、nginx という専用ユーザーが作成され、そのユーザーの権限でNginxが動作する。
これにより、Nginxがシステム全体に過度なアクセス権限を持つことを防ぐ。

### 確認方法① ps コマンドを使用する

ps コマンドでNginxのプロセスとそれを実行しているユーザーを確認することができます。

```
[実行コマンド]
ps aux | grep nginx

[結果]
nginx      1234  0.0  0.1 123456  1234 ?        Ss   10:00   0:00 nginx: master process /usr/sbin/nginx
nginx      1235  0.0  0.1 123456  1234 ?        S    10:00   0:00 nginx: worker process
```

###  pgrep と ps を組み合わせる

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


### systemctl で確認する（システムが systemd を使用している場合）

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










2. worker_processes auto;

    説明: Nginxが使用するワーカープロセスの数を指定します。
    詳細:
        ワーカープロセスは、クライアントからのリクエストを処理するためのプロセスです。
        auto に設定すると、NginxはサーバーのCPUコア数に基づいて最適なワーカープロセス数を自動的に決定します。たとえば、4コアのCPUがある場合、Nginxは4つのワーカープロセスを生成します。
        ワーカープロセスの数は、負荷やサーバーの性能によって調整が可能で、リクエストの処理能力に影響を与えます。高トラフィックのサーバーでは、この値をチューニングすることがパフォーマンス向上に繋がります。

3. error_log /var/log/nginx/error.log warn;

    説明: Nginxのエラーログファイルのパスとログレベルを指定します。
    詳細:
        /var/log/nginx/error.log は、Nginxのエラーログを保存するファイルのパスです。
        warn は、ログレベルを示します。Nginxが記録するエラーログのレベルを制御します。ログレベルには以下の段階があります:
            debug: デバッグ用の詳細なログを記録。
            info: 情報レベルのログ。
            notice: 通知レベルのログ。
            warn: 警告レベルのログ。
            error: エラーが発生した場合のみ記録。
            crit: クリティカルなエラー（重大なエラー）。
        ここでは warn が指定されているので、警告以上のエラー（警告、エラー、クリティカルエラー）が発生した際にログが記録されます。

4. pid /var/run/nginx.pid;

    説明: NginxのプロセスID（PID）ファイルの保存場所を指定します。
    詳細:
        /var/run/nginx.pid は、NginxのプロセスIDが記録されるファイルです。
        PIDは、現在実行中のプロセスに一意に割り当てられるIDで、システムがプロセスを管理する際に使われます。Nginxの管理操作（再起動、停止など）を行う際に、このファイルからPIDを読み取り、対応するプロセスに対して操作を行います。

このセクション全体の役割:

    セキュリティ: user nginx; で非管理者権限のユーザーを指定することで、サーバーのセキュリティを向上させます。
    パフォーマンス: worker_processes auto; により、サーバーのCPU性能に合わせた最適なリクエスト処理が可能になります。
    トラブルシューティング: error_log でエラーログの保存先と詳細度を指定し、問題発生時に原因を追跡しやすくします。
    プロセス管理: pid ファイルは、Nginxの制御を行うために重要です。