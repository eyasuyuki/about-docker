# About Docker

タイムインターメディア技術部会資料

# Dockerとは

- コンテナ仮想化技術
- http://docker.com/

# 動作環境

- Kernel 3.8以上の64bit Linux
 - ex. OS XではVirtualBox用のLinuxイメージを利用

# ツール

- docker-machine リモート管理
- docker-compose 複数コンテナのビルドや起動
- docker-swarm クラスタリング

# メリット

- 起動が速い
- 構成を静的に記述できる
 - Dockerfile
 - docker-compose.yml
- 既存のimageを利用できる
 - dockerhub

# デメリット

- http://deeeet.com/writing/2015/02/17/docker-bad-points/
 - root権限で動作する
 - imageに署名がない

# デモ

## インストール

    brew install caskroom/cask/brew-cask
    brew cask install virtualbox
    brew install docker docker-machine docker-compose

## VirtualBox image初期化

    docker-machine create -d virtualbox <vmname>

## 起動

    docker-machine start <vmname>

## 環境変数

    $(docker-machine env <vmname>)

## ログイン

    docker-machine ssh <vmname>

## 終了

	docker-machine stop <vmname>

## Dockerコマンド

    docker ps
    docker images
	docker inspect <imageid>

## docker-composeコマンド

    docker-compose ls
    docker-compose up -d
	docker-compose stop

## golang-fastcgi-nginx

- nginxのfastcgiでgoを動かす
 - Goコンテナ
 - nginxコンテナ
- コンテナ間のlink

### 概念図

    +-----------------+    +----------------+
    | nginx container |    |  go container  |
    |   172.17.0.5:80 | -- | 172.17.0.3:9991|
    +-----------------+    +----------------+
    +---------------------------------------+
    |                Docker                 |
    +---------------------------------------+
    +---------------------------------------+
    |             Linux Kernel              |
    +---------------------------------------+
    +---------------------------------------+
    |               Hardware                |
    +---------------------------------------+

### docker-compose.yml

    nginx:
      build: nginx
      links:
       - fcgihost
      ports:
       - "80:80"
      expose:
       - "80"

    fcgihost:
      build: golang-docker/hello
      ports:
       - "9001:9001"
      expose:
       - "9001"

#### /etc/nginx/sites-enabled/default (一部)

    location / {
        fastcgi_pass    fcgihost:9001;
        include     fastcgi_params;
    }

#### hello.go

    package main

    import (
        "fmt"
        "net"
        "net/http"
        "net/http/fcgi"

        "github.com/GoogleCloudPlatform/golang-docker/hello/vendor/internal"
        "github.com/gorilla/mux"
    )

    func handler(w http.ResponseWriter, r *http.Request) {
        http.Redirect(w, r, fmt.Sprintf("/%s", internal.Secret), http.StatusFound)
    }

    func hello(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "<html><body>Hello, %s! こんにちは、世界!</body></html>", mux.Vars(r)["who"])
    }

    func main() {
        l, err := net.Listen("tcp", ":9001")
        if err != nil {
            return
        }
        r := mux.NewRouter().StrictSlash(true)
        r.HandleFunc("/", handler).Methods("GET")
        r.HandleFunc("/{who}", hello).Methods("GET")

        fcgi.Serve(l, r)
    }

### cloneとbuild

    git clone git@github.com:eyasuyuki/golang-fastcgi-nginx.git
    cd golang-fastcgi-nginx
    git clone git@github.com:eyasuyuki/golang-docker.git
    git clone git@github.com:eyasuyuki/nginx.git
    docker-compose up -d

### HTTPアクセス

    curl -i http://$(docker-machine ip <vmname>)/

### コンテナへのログイン

    docker-machine ssh <vmname>
    docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
    sudo docker-enter `docker ps | grep "\"nginx\"" | cut -d " " -f 1`

### コンテナ間のlinkの確認

    grep "fcighost" /etc/hosts
	ping fcgihost

## docker-kahua

    git clone git@github.com:eyasuyuki/docker-kahua.git
    cd docker-kahua
    docker-compose up -d
    curl -i http://$(docker-machine ip <vmname>):8888/
