### Docker on Vagrant on WSL

#### 参考

- [Windows10+VirtualBox+VagrantでDockerはじめました](http://www.nct-inc.jp/engineer_blog/2807/)
- [いまさらDockerに入門したので分かりやすくまとめます](https://qiita.com/gold-kou/items/44860fbda1a34a001fc1)

#### 大きいゴール

- Dockerを使えるようになる

#### 大きいゴールを一旦達成したとみなすための小さいゴール

- Dockerを使って、RSpecのテストしたい(docker-composeを使うのかも知らない)
- docker-composeでRailsとか動かしたい
- docker-composeを使って、redisとrailsの2つをつなげたい

#### 記事を読んで、新しい知見につながったキーワード、文

- Linuxカーネルのnamespaceの機能
  - コンテナ毎の区画化
  - namespace毎に管理してるであろう `MOUNT` に関して
  - namespace毎に管理してるであろう `UTS` に関して
  - namespace毎に管理してるであろう `IPS` に関して
- Linuxカーネルのcgroupの機能
  - プロセスをグループ化してグループごとにリソース使用制限をかける
- ネットワーク
  - NIC
  - ブリッジ(docker0)
  - インターフェース
    - コンテナ側から見ると物理NIC(eth0)のように見える
    - ホストOS側から見ると実態はveth
    - vethはL2の仮想NIC
    - コンテナのNIC(eth0)とホストOSのブリッジ(docker0)間でトンネリング
  - NAPT

#### Dockerのイメージを取得

```
$ sudo docker pull debian
# デフォルトはtag: latest(最新版)が入る
...
```

#### 確認

hello-worldはDockerが入ったか確認用で公式にあったコマンド叩いたら入ったらしい

```
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              4879790bd60d        4 weeks ago         101MB
hello-world         latest              4ab4c602aa5e        3 months ago        1.84kB
```

#### 休憩:docker コンテナを全部消す

何回か `sudo docker run hello-world` したので、imageは1つでもコンテナはrunした回数だけ増えている

```
$ sudo docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
641b45a201f1        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       clever_driscoll
020e50b4fbcd        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       xenodochial_liskov
b8afe7b6ad53        hello-world         "/hello"            2 hours ago         Exited (0) 2 hours ago                         youthful_mirzakhani
c331c4dca8b1        hello-world         "/hello"            2 hours ago         Exited (0) 2 hours ago                         cranky_austin
decd3f46aec2        hello-world         "/hello"            2 hours ago         Exited (0) 2 hours ago                         festive_chatterjee
...
```

このことでわかるのは

- `docker image` から `docker container` を作成している..?

予想できるのは、

- `docker container` を消しても `docker image` は消えない
- `docker image` を消しても `docker container` は消えない

##### `docker container` を消しても `docker image` は消えない検証

```
$ sudo docker rm $(sudo docker container ls -aq)
641b45a201f1
020e50b4fbcd
b8afe7b6ad53
c331c4dca8b1
decd3f46aec2

$ sudo docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
# 何も出ない(containerは消した)

# imageが消えてないと予想したが、確認
$ sudo docker images -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              4879790bd60d        4 weeks ago         101MB
hello-world         latest              4ab4c602aa5e        3 months ago        1.84kB
```

##### `docker image` を消しても `docker container` は消えない検証

```
$ sudo docker run hello-world
$ sudo docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
646e0d2c418e        hello-world         "/hello"            41 seconds ago      Exited (0) 40 seconds ago                       suspicious_mestorf
$ sudo docker image ls -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              4879790bd60d        4 weeks ago         101MB
hello-world         latest              4ab4c602aa5e        3 months ago        1.84kB
$ sudo docker image rm 4ab4c602aa5e
Error response from daemon: conflict: unable to delete 4ab4c602aa5e (must be forced) - image is being used by stopped container 646e0d2c418e
```

- `docker container` が残っていると、それの元になった `docker image` は消せない
  - forceオプションとかを使って無理やり消すと、`docker container` も消えると予想試す

```
$ sudo docker image rm 4ab4c602aa5e -f
Untagged: hello-world:latest
Untagged: hello-world@sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Deleted: sha256:4ab4c602aa5eed5528a6620ff18a1dc4faef0e1ab3a5eddeddb410714478c67f
$ sudo docker image ls -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              4879790bd60d        4 weeks ago         101MB
$ sudo docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
646e0d2c418e        4ab4c602aa5e        "/hello"            11 minutes ago      Exited (0) 11 minutes ago                       suspicious_mestorf
```

- 消えてない。。。当初の予想が正解
  - `docker container` を消しても `docker image` は消えない
  - `docker image` を消しても `docker container` は消えない
