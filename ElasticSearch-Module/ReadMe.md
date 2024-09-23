# はじめに
ElasticSearchのモジュールを作成した。


# volumes.yaml解説

volumesは、Dockerの永続的なデータ保存領域を定義するセクションである。
コンテナが削除されたり再作成された場合でも、データが消えないように永続化するために使用される。

es_dataはボリュームの名前である。
この名前でホスト上にデータが保存され、Elasticsearchのデータが格納される。

driver: localは、Dockerがデフォルトで提供するローカルボリュームドライバを使用することを指定している。
このドライバは、ホストマシン上に直接データを保存する。

特にカスタマイズされない場合は、local ドライバがデフォルトで使用される。

## nginxのvolumeセクションとの違い

### バインドマウント (Bind Mount)

Nginxの設定は「バインドマウント」と呼ばれるもの。

```
volumes:
  - ./nginx.conf:/etc/nginx/nginx.conf:ro  # カスタム設定を使用する場合
  - ./html:/usr/share/nginx/html:ro  # 静的なHTMLファイルを提供する場合
```
`./nginx.conf:/etc/nginx/nginx.conf:ro: `ホスト側の ./nginx.conf ファイルをコンテナ内の `/etc/nginx/nginx.conf` に読み取り専用（ro）でマウントしている。
これは、ローカルファイルを直接コンテナに反映させるためのバインドマウントの一例である。

