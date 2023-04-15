# EC2でサンプルアプリケーションデプロイ

### EC2の環境構築

```
ruby:パッケージのアップデート、gitのインストール
$ sudo yum update -y
$ sudo yum install git
```

### rbenvのインストール
```
githubからclone
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
```

```
パスを通す
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```
### ruby buildのインストール
```
cloneして、インストール
$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ cd ~/.rbenv/plugins/ruby-build
$ sudo ./install.sh
$ rbenv install -l
```

###　rubyのインストール
```
$ sudo yum -y install gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel libffi-devel libxml2 libxslt libxml2-devel libxslt-devel sqlite-devel
```
### Node.jsのインストール
NVMを有効にする
```
$ . ~/.nvm/nvm.sh
```
最新のnodeをインストール
```
$ nvm install --lts
```
今回は V17.9.1の為、下記実行
```
$ nvm install 17.9.1
```

### yarn インストール

AWSのリポジトリに追加
```
$ curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
インストール
[ec2-user@ip-172-31-47-9 ~]$ sudo yum -y install yarn
```

### サンプルアプリクローン
サンプルアプリのディレクトリへ移動

bundler インストール
```
$ gem install bundle
$ gem install bundler
```
### MYSQL
mysql8.0のリポジトリを追加する

```
$ sudo yum localinstall -y https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

mysql8.0のリポジトリを有効化する

```
$ sudo yum-config-manager --disable mysql57-community
$ sudo yum-config-manager --enable mysql80-community

```

mysql-community-clientをインストールする

```
$ sudo yum install -y mysql-community-client
```

mysqlログイン（確認）

```
$ mysql -u ユーザー名 -p -h RDSのエンドポイント
#　パスワードを求められるのでRDSを作成した際に設定したパスワードを入力
```
### database.yml 書き換え
```
adapter:   使用するデータベース種類
encoding:  文字コード
reconnect: 再接続するかどうか
database:  データベース名
pool:      コネクションプーリングで使用するコネクションの上限
username:  ユーザー名
password:  パスワード
host:      MySQLが動作しているホスト名
```
