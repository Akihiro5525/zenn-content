---
title: ".envを使用したdocker-compose.ymlの環境変数設計"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker-compose.yml",".env"]
published: true
---
共通する値を一元管理してDokcer運用を楽にするのが目的です
定義方法は色々[^1]ありますが .envファイル内で宣言する想定で書いていきます

[^1]:4つの定義方法
・docker-compose.ymlファイル内で宣言
・コンテナ実行時にコマンドで渡す
・.envファイル内で宣言
・独自ファイルを用意して宣言



# 環境変数の流れ
まずは全体の流れから...

.env (環境変数の一元管理)
↓
dokcer-compose.yml
↓
Dockerfile (不変値の宣言)
↓
コンテナ (内のRailsなど)


:::message
.envファイルは、docker-compose.ymlと同じディレクトリにあるものが自動で読まれます
:::

定義するには**変数名=初期値**です

```js:.env
WORLDIR=app
```
<br>

# docker-compose.ymlから環境変数を渡す2つの方法

１_docker-compose.yml　=>　Dockerfile
２_docker-compose.yml　=>　コンテナ

## １_docker-compose.yml ⇒ Dockerfile

buildのargsキーを使います
宣言は、**キー(変数名): 値(app)**

```ruby:docker-compose.yml
build:
  args:
    WORKDIR: $WORKDIR

    # もしくは
    - WORKDIR=$WORKDIR
```


上記の変数はARG命令で受け取ることができます↓

```ruby:Dockerfile
ARG WORKDIR
```
流れとしては...
.env ⇒ docker-compose.yml args(渡す) ⇒ Dockerfile(受け取る)


## ２_docker-compose.yml ⇒ コンテナ
docker-compose.ymlからコンテナへ環境変数を渡す方法は2つあります


### 2-1_扱う環境変数が少ない場合はenvironment
```ruby:docker-compose.yml
build:
  environment:
    WORKDIR: $WORKDIR

    # もしくは
    - WORKDIR=$WORKDIR
```
<br>

### 2-2_扱う環境変数が多い場合はenv_file
env_fileには値としてファイルパスを渡します
```ruby:Dockerfile
services:
  api:
    ...
    env_file: .env # 絶対パス or 相対パス
```

複数ファイルを指定する場合
```ruby:Dockerfile
services:
  api:
    ...
    env_file:
        - ./.env
        - ./api/envfile
        - ./front/.envfile
```
<br>
以上です！
間違いがありましたらマサカリ等いただけると嬉しいです。