# About Docker

- http://docker.com/
- コンテナ仮想化技術

# ツール

- docker-machine
- docker-compose
- docker-swarm

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

    brew install docker.io
    brew tap homebrew/devel-only
    brew install --devel docker-machine

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

##　golang-fastcgi-nginx

- nginxのfastcgiでgoを動かす
 - Goコンテナ
 - nginxコンテナ
- コンテナ間のlink
 - docker-compose.yml

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

