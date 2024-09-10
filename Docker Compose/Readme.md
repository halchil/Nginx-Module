# はじめに

Docker Compose設定ファイルを分割した。
各分割したパートは、モジュールと呼ぶ。

# 参考文献
<!--Docker,Docker Composeのバージョン対比表を入れる -->


# versionについて
[version.yaml](https://github.com/halchil/Nginx-Module/blob/main/Docker%20Compose/version.yaml)

docker-composeのversionは、Compose ファイルのフォーマットバージョンを指している。
Compose ファイルのバージョン: Docker Composeには複数のバージョン(例: 2, 2.1, 3, 3.7, 3.8, など)があり、それぞれのバージョンには特定の機能セットや構文ルールがある


# servicesについて
[services.yaml](https://github.com/halchil/Nginx-Module/blob/main/Docker%20Compose/services.yaml)


ここでは、主にサービス名とコンテナ名を定義することになる。

両者の違いを説明する

**サービス名**について

定義場所は、`docker-compose.yaml`ファイル内のservices:セクションである。
service名はDocker Composeで定義された一連の設定（イメージ、ボリューム、ネットワークなど）に対してつけられた名前である。
複数のコンテナをまとめて管理するためのグループ名のようなもの。

例を以下に示す
```
services:
  web:  # これがservice名
    image: nginx:latest
```
上記の例では、webがservice名となる。


**コンテナ名**について


まとめると、サービス名はDocker Compose全体で定義される名前で、特定のサービスを管理・参照するために使われる。
一方で、コンテナ名はDockerエンジン上の個々のコンテナを識別するための名前である。


また、一対多の関係となっている。
1つのサービスは複数のコンテナを持つことが可能である。

例えば、スケーリングする場合、webサービスとして3つのNginxコンテナを立ち上げると、webというサービス名に対してweb_1, web_2, web_3のようなコンテナが起動する。


# networksについて
[networks.yaml](https://github.com/halchil/Nginx-Module/blob/main/Docker%20Compose/networks.yaml)

Docker Composeファイルの networks セクションでは、コンテナ間で通信するために使用するネットワークを定義する。

ecosystemはネットワーク名である。
ここでは、コンテナが接続されるネットワークの名前として ecosystem を指定しています。

`external: true` の設定は、「このネットワークは外部で既に作成されているものなので、Composeはそのネットワークを新たに作成しない」という意味である。
つまり、docker-compose up を実行した際に、Composeは ecosystem という名前のネットワークが既に存在していると仮定してそのネットワークを使用する。
