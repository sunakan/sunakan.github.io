### Dockerの基礎

- Docker
  - Docker Image
    - Docker Containerの元
  - Docker Container
    - Docker on OSから見ると1プロセスとなるメイン

#### Docker Image

- 例えばCentOSをpull
- そして確認
- 詳細
- 削除

```
$ sudo docker pull centos
```

```
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              1e1148e4cc2c        2 weeks ago         202MB
$ sudo docker image ls
```

```
$ sudo docker inspect centos:latest
or
$ sudo docker inspect 1e1148e4cc2c
```

```
$ sudo docker rmi centos:latest
or
$ sudo docker rmi 1e1148e4cc2c
```

#### Docker Container

- 1プロセスを作って、(終了しても)捨てないで `とっておく` と考えるとわかりやすかった

- centos:latestのDocker Imageを元にDocker Containerを作って ` echo "Hello world" `
- 実行中のDocker Container一覧
- Docker Container一覧(ALL)
- Docker Containerの削除


```
$ sudo docker run centos:latest echo "Hello world"
```

```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ sudo docker container ls
```

実行が終了してたら何も出ない

```
$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS                     PORTS               NAMES
2821e499fb83        centos:latest       "echo 'HELLO WORLD'"   4 minutes ago       Exited (0) 4 minutes ago                       lucid_matsumoto
or
$ sudo docker container ls -a
```

```
$ sudo docker rm 2821e499fb83
```

#### 実行中のDocker Containerを操作

- CentOSのDocker Imageを元にfreeコマンド(メモリ状況の確認)を3秒毎に実行するDocker Containerを作成(バックグラウンドで動くように)

- Docker Container IDの指定は一意にできればよいので最初の3-4文字でもいける(GitのCommit ID的な)

- logの確認
- foregroundにもってくる(Ctrl + c で抜けられる)
- ストップ & スタート

```
$ sudo docker run -d centos free -s 3
```

```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
f79f7836d224        centos              "free -s 3"         About a minute ago   Up About a minute                       angry_mcclintock
$ sudo docker logs f79
```

```
$ sudo docker attach --sig-proxy=false f79
```

```
$ sudo docker kill f79
$ sudo docker start f79
```

#### Docker Containerに変更を加えて、Docker Imageを作成

- 新しくbashで実行させ、入る
- hogehoge.txtを追加
- Docker Imageを作成(名前は慣習的にn `ame/hogehoge` )
- 作成したDocker Imageを元にまたDocker Containerを作成し、確認

```
$ sudo docker run -it centos bash
$ touch hogehoge.txt
$ ls
```

```
$ sudo docker commit 9380a9df815 sunakan/hogehoge
sha256:961076db4c581c7dddfd038d7f7e117b9a812e74c1304cd830167f226b13722a
~$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sunakan/hogehoge    latest              961076db4c58        18 seconds ago      202MB
centos              latest              1e1148e4cc2c        2 weeks ago         202MB
```

```
$ sudo docker run -it sunakan/hogehoge bash
$ ls
# hogehoge.txtを確認できたらOK
```

#### Dockerfileを利用

- Docker Image(A)を元にDocker Containerを作成、変更を加える、それをまたDocker Image(B)にする
  - この作業を Docker Build と呼ぶ
  - スクリプト化可能(自動化かも可能)
    - 自動化するために、Dockerfileが必要

- ちょっと疑問
  - Dockerfileを使わなくても手動でDocker Image(B)を作成したら、Docker buildした言っていい？
    - 普通はDockerfileを使うことを指しそう

- Dockerfileを使って、Docker build(TAG名はsunakan/echoとする)
  - Dockerfileを指定したい場合は `-f` オプション(今回はわざと名前をDockerfile => Dockerfile01)
  - Dockerfileというファイル名なら指定する必要なし
- 確認

```
$ vim Dockerfile01
```

Dockerfile01の中身

```
# 何のDocker Imageを元にするか
FROM centos

# 誰が作成したか
# MAINTAINERは1.13から非推奨
LABEL  maintainer "sunakan <sunakan03@gmail.com>"

# RUN: buildする時に実行される
RUN echo "ビルド中..."

# CMD: runする時に実行される
CMD echo "実行中..."
CMD ["echo", "実行中..."]
```

```
$ sudo docker build -t sunakan/echo -f ./Dockerfile01 .
Sending build context to Docker daemon    746kB
Step 1/5 : FROM centos
 ---> 1e1148e4cc2c
Step 2/5 : LABEL  maintainer "sunakan <sunakan03@gmail.com>"
 ---> Running in 4b986ccc3516
Removing intermediate container 4b986ccc3516
 ---> b7e7fc65f37a
Step 3/5 : RUN echo "ビルド中..."
 ---> Running in ea3525905ae3
ビルド中...
Removing intermediate container ea3525905ae3
 ---> 420c08e69340
Step 4/5 : CMD echo "実行中..."
 ---> Running in 49cd874ac991
Removing intermediate container 49cd874ac991
 ---> 4a3e4b0f6cc7
Step 5/5 : CMD ["echo", "実行中..."]
 ---> Running in f622e4f217f4
Removing intermediate container f622e4f217f4
 ---> 93df0b4b97d1
Successfully built 93df0b4b97d1
Successfully tagged sunakan/echo:latest
```

```
$ sudo docker images
$ sudo docker run sunakan/echo
```

#### Dockerfileを利用してWebサーバを立ち上げる(最低限)

- Dockerfile-nginxを作成
- `docker build` で Docker Imageを作成
- 作成したDocker Imageを元にDocker Containerを作成
  - 作成した側のポート12345を作成されたDocker Containerのポート80にフォワーディング
- アクセス

Dockerfile-nginxの中身

```
FROM debian:latest
LABEL maintainer="sunakan <sunakan03@gmail.com>"

RUN apt-get update \
  && apt-get install -y nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```
$ vim Dockerfile-nginx
$ sudo docker build -t sunakan/nginx -f ./Dockerfile-nginx .
$ sudo docekr images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sunakan/nginx       latest              2a30fe09b56e        5 minutes ago       177MB
debian              latest              4879790bd60d        5 weeks ago         101MB
```

```
$ sudo docker run -p 12345:80 -d sunakan/nginx
$ sudo docker ps
```

- ブラウザで確認
  - 今回Vagrantでやっているため、 `ip a | grep "inet"`とかでip調べる
  - それの12345ポートにブラウザで開く
- 今回
  - ip:192.168.33.11
  - だから http://192.168.33.11:12345/
  - Welcome to nginx!が見えればOK

#### Dockerfileを利用してWebサーバを立ち上げる(DockerHubの利用)

- DockerHubに公式の人がDocker Imageを上げてたりする
  - それのDockerfileを見ると結構色々してて、参考になる
- NginxのDocker Imageを持ってきて上記と同じことする
  - Docker Imageでの指定で `nginx` とか指定すると、手元のDocker Imageになかった場合
    - 自動で探してきてそれのlatestを持ってきてくれる

```
$ sudo docker run -p 12345:80 -d nginx
```

- 最低限でいいのならコマンド1発で終わってしまうという手軽さ
- ブラウザで確認可能
