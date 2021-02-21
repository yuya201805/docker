
# dockerで、ローカル環境の自作をします。


## まとめ

参考  
https://qiita.com/azul915/items/5b7063cbc80192343fc0  
webpackerとyarnの設定は、以下が参考になった。   
https://qiita.com/kodai_0122/items/795438d738386c2c1966  
プライベートリポジトリを組み込みなら以下の方法もありかも。    
https://www.jonki.net/entry/2018/03/24/054407  



## dockerの概要

### コンテナの仕組みと利点

#### 1 隔離された実行環境を提供する。

##### コンテナとは?
```
互いに影響されない隔離された実行環境を提供する技術
```
#### 複数のシステムが同居するうときの問題点
```
 ディレクトリの競合
```
#### コンテナの利点
```
それぞれのディクトリツリーを持つので、競合しない
```
#### dockerコンテナを一実行するためには？
```
linux上に、docker engineをインストールする。
docker engine 上で dockerコンテナが動く
docker engineの dockerコマンドを使用して、 dockerコンテナを動かす
```
#### dockerコンテナを作成するには？
```
まっさらな状態から コンテナを作り込んでいくの大変なので、
dockerイメージ(コンテナの元)を使用する。
dockerイメージは、docker hubから落とす
```
#### dockerイメージについて
```
 docker イメージには、 2種類あり、
 基本的な linux ディストリビューションだけのイメージと
 アプリが入りのイメージ 
```
#### 業務で使用する際は？
```
linuxのみのイメージを使用することがほとんど。
基本的なイメージに、自社のシステムを組み込んでいく。
```
#### dockerは linux 上で、動くので、windowsやmacでは、どう使用するのか？
```
dockerデスクトップを使用する
docker デスクトップは、内部に　 linuxカーネルを持っており、
その上で、 docker　engineを動かし、 docker engineの上に、 dockerコンテナを動かせる
```
### 仮想サーバと コンテナの違い
```
仮想サーバは、あくまで、物理的なサーバーの中に、仮想サーバを分割しているが、仮想サーバにはそれぞれ OSがインストールされその上で動く。
対して、 コンテナは、サーバはあくまでも１台。制御するOSは１つしかない。
```




## 作成手順まとめ
```
　注意: build 時と、create時などが遅いです。改善しないと。
```

### プロジェクトをclone
```
baukis_for_conoha のアプリを git cloneしてきます。
```
### 環境変数の設定
```
docker-compose.ymlのDBの設定に、環境変数を使用したので、ローカルに設定してあるか確認します。(ここも要改善)
ここで設定したものが、dockerのDBに使用されます。
以下２つを好きに設定してだくさい。(例)
MYSQL_DATABASE="root"
MYSQL_ROOT_PASSWORD"password"

アプリに、.envを使用しているので、.envファイルをアプリにおきます。(環境変数を受けわたす方法が他にあると思うので、要改善)
上で設定したdockerのDBにアプリから入るために使用します。
以下２つを好きに設定してだくさい。(例)
DB_USER=root
DB_PASSWORD="password"
```

### buildの実施
```
docker-compose.yml のディレクトリで、docker-compose build を実行
```

### upを実施
```
docker-compose up　を実行
```


### エラー対応。ここはコマンドにあとで入れよう
```
yarn checkを求められるので、web側で実行します。
docker-compose run web yarn install --check-files
```

## dbの作成とテストデータ
```
 docker-compose run web bin/rake db:create db:migrate db:seed
 
 もしくは一つずつのコマンド
 docker-compose run web bundle exec rails db:create
 docker-compose run web bundle exec rails db:migrate
 docker-compose run web bundle exec rails db:reset
```

### 接続確認
```
localhost:3000 につながるか確認(遅いです)
確認できました。
```

### 終了
```
control c で閉じて、docker-compose down　して終わりです。
```

### 備考
```
以下の webpackerのインストールはしなくても繋がった。
docker-compose run web bundle exec rails webpacker:install
```

課題はいくつか残っているので、もっと良い方法に改善していこうー。


### コンテナの中に入って作業
```
web, dbのコンテナにそれぞれ入れます。
 docker ps で確認
 docker exec -it moco_web_1 /bin/bash
 docker exec -it docker_web_1 /bin/bash
 docker exec -it docker_db_1 /bin/bash
```









## とりあえず、詰まった箇所など以下に殴り書きます。後ほど、まとめを作成すること。
```
webpacker　と　yarn　に関しては以下の記事が役に立った。 
https://qiita.com/kodai_0122/items/795438d738386c2c1966

以下どちらも成功した。
docker-compose run web bundle exec rails webpacker:install
docker-compose run web yarn install --check-files

git clone してきたファイルに追加
.env
環境変数の設定

dbのエラー

docker-compose up した状態で、
docker-compose run web bundle exec rails db:create
docker-compose run web bundle exec rails db:migrate


コンテナへ入る
docker exec -it docker_web_1 /bin/bash

```


参考
https://www.jonki.net/entry/2018/03/24/054407
https://qiita.com/azul915/items/5b7063cbc80192343fc0

### docker 作成時のエラー一覧
docker install 時に、  yarn isntall check　のエラーが発生
以下の記事を参考に、yarn のチェックをなくしたら、無事にとおった 
https://qiita.com/yama_ryoji/items/1de1f2e9e206382c4aa5


Can't connect to local MySQL server through socket '/var/run/mysqld/mysql のエラー。
touch /var/run/mysqld/mysqld.sock した。
https://qiita.com/yama_ryoji/items/1de1f2e9e206382c4aa5

後、dabase.ymlの host: db　をdbに変更。compose.ymlの値を参照するようにした。　　
https://qiita.com/yama_ryoji/items/1de1f2e9e206382c4aa5

上の２つを実施したら、DBのエラーが消えた。



docker での env を扱いはどうなる? 
コンテナ、dockerfile, docker-compose.yml 内で読み込みが異なる
いかが参考になる
今回は、.envを後から追加した。
https://qiita.com/KEINOS/items/518610bc2fdf5999acf2

docker-comose.yml は、ローカル側で設定した環境変数を使用することに



manifile
https://qiita.com/negisys/items/2bf88659f584fe45b686


docker 作成時に実施すること
```
git clone 
アプリ内に.env ファイルの設置
ローカルに、環境変数を設置(既存で環境変数が存在すれば問題ない)
```


