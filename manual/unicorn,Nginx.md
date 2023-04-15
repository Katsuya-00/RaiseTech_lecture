# WEBサーバー、APサーバー

## unicornインストール
gem 追加後、bundle install
unicorn.rbの編集

```
listen '/home/vagrant/myapp/tmp/unicorn.sock'
pid    '/home/vagrant/myapp/tmp/unicorn.pid'
listen 8080
```
を追加し、
```
bundle exec unicorn_rails -c config/unicorn.rb
```
http://localhost:8080

unicornでのアプリケーション動作確認。（最後に消した）


## EC2にnginxをセットアップ
yum updateを実行し、インスタンスをアップデート
```
$ sudo yum update
```
nginxのyumの有効化
```
$ sudo amazon-linux-extras enable nginx1
```
nginxのインストール
```
$ sudo yum -y install nginx
```
インストールの確認
```
$ nginx -v
```
自動起動の設定
```
$ sudo systemctl enable nginx
```
起動
```
$ sudo systemctl start nginx.service
```
状態確認
```
sudo systemctl status nginx.service
```
```
   Active: active (running)
```
であればOK

### nginx.confの設定
```
$ sudo vim /etc/nginx/nginx.conf
```
### .confの書き換え
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    gzip on;
    gzip_http_version 1.0;
    gzip_proxied any;
    gzip_min_length 500;
    gzip_disable "MSIE [1-6]\.";
    gzip_types text/plain text/xml text/css
               text/comma-separated-values
               text/javascript application/x-javascript
               application/atom+xml;
}
```
### conf.d配下にファイル作成
```
$ cd /etc/nginx/conf.d
$ sudo vim raisetech.conf
```
記載
```
upstream unicorn_server {
  # Unicornと連携させるための設定。
  # config/unicorn.rb内のunicorn.sockを指定する
  server unix:/home/ec2-user/raisetech-live8-sample-app/unicorn.sock;
}
server {
  listen 80;
  # 接続を受け付けるリクエストURL ここに書いていないURLではアクセスできない
  server_name ●●●;
  client_max_body_size 2g;
  # 接続が来た際のrootディレクトリ
  root /home/ec2-user/raisetech-live8-sample-app/public;
  # assetsファイル(CSSやJavaScriptのファイルなど)にアクセスが来た際に適用される設定
#  location ^~ /assets/ {
 #   gzip_static on;
  #  expires max;
   # add_header Cache-Control public;
 #  }
  try_files $uri/index.html $uri @unicorn;
  location @unicorn {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://unicorn_server;
  }
  error_page 500 502 503 504 /500.html;
}
```

