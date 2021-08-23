# 手順書

## バージョン一覧

ソフトウェア|バージョン
---|---
PHP|*7.3*
Nginx|*1.20.1*
MySQL|*5.7*
Laravel|*6.0*


## 環境構築の流れ
1. [Vagrantインストール](#vagrantインストール)
1. [boxという形でOSファイルをダウンロード](#boxという形でosファイルをダウンロード)
1. [作業ディレクトリ下で設定ファイルの生成](#作業ディレクトリ下で設定ファイルの生成)
1. [設定ファイルを編集](#設定ファイルを編集)
1. [Vagrantのプラグインをインストール](#vagrantのプラグインをインストール)
1. [Vagrantの起動](#vagrantの起動)
1. [ゲストOSへのログイン](#ゲストosへのログイン)
1. [パッケージのインストール](#パッケージのインストール)
1. [PHPのインストール](#phpのインストール)
1. [Composerのインストール](#composerのインストール)
1. [Laravelアプリケーションのコピー作成](#laravelアプリケーションのコピー作成)
1. [Nginxのインストール](#nginxのインストール)
1. [Laravelを動かす](#laravelを動かす)
1. [データベースのインストール](#データベースのインストール)
1. [データベースの作成](#データベースの作成)
1. [Laravelを動かす](#laravelを動かす)

### Vagrantインストール
```sh
$ brew install --cask vagrant
$ vagrant -v
```

### boxという形でOSファイルをダウンロード
今回はvirtual boxを使用
```sh
$ vagrant box add centos/7
```

### 作業ディレクトリ下で設定ファイルの生成
```
$ vagrant init centos/7
```

### 設定ファイルを編集
\#を外し、ipを変更する
```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
config.vm.network "private_network", ip: "192.168.33.19”

config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
# /vagrant のディレクトリ内をリアルタイムで同期するための設定
```

### Vagrantのプラグインをインストール
＜vagrant-vbguest＞
初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグイン
```sh
$ vagrant plugin install vagrant-vbguest
```
```sh
$ vagrant plugin list
# 他のインストールしているプラグインを見れる
```
・・・仮想環境を構築する準備が終わり

### Vagrantの起動
```sh
$ vagrant up
```

起動に成功すれば、ホストOSの上に別のゲストOSが立ち上がった事になる
[もしマウントに失敗したら](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)

・・・ゲストOSへのログイン
### ゲストOSへのログイン
```sh
$ vagrant ssh
```

### パッケージのインストール
gitなどの開発に必要なパッケージを一括でインストールできる。
```sh
$ sudo yum -y groupinstall "development tools"
```

### PHPのインストール
```sh
$ sudo yum -y install epel-release wget
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo rpm -Uvh remi-release-7.rpm
$ sudo yum -y install --enablerepo=remi-php72 php php-pdo php-mysqlnd
$ php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
$ php -v　# バージョン確認
```
PHPのバージョンが出力されればOK
```sh
PHP 7.2.34 (cli) (built: Jun 28 2021 11:21:49) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```

### Composerのインストール
```sh
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行う
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -v　
```
composerのバージョンが出ればOK
```sh
Composer version 2.1.6 2021-08-19 17:11:08
```

### Laravelアプリケーションのコピー作成
作業ディレクトリ下でlaravelのアプリケーションのコピーを作成
```sh
$ cp -r laravel_appディレクトリまでの絶対パス ./
```

### Nginxのインストール
ゲストOSにログイン
viエディタを使用して以下のファイルを作成
```sh
$ sudo vi /etc/yum.repos.d/nginx.repo
```
以下は書き込む内容

```nginx
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```

インストールする
```sh
$ sudo yum install -y nginx
```
Complete!が出ればOK

Nginxを起動し、　
ブラウザにて[http://192.168.33.19]を押して、　　
NginxのWelcomeページが表示されたらOK
```sh
$ sudo systemctl start nginx
```

###　Laravelを動かす
Nginxの設定ファイルを編集
```sh
$ sudo vi /etc/nginx/conf.d/default.conf
```
```nginx
server {
  listen       80;
  server_name  192.168.33.10; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  # ApacheのDocumentRootにあたります
  root /vagrant/laravel_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }
```

php-fpm の設定ファイルを編集
```sh
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```
### データベースのインストール
今回インストールするデータベースはMySQLとなります。versionは5.7を使用
rpmに新たにリポジトリを追加し、インストールを行う
```sh
$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
$ sudo yum install -y mysql-community-server
$ mysql --version
```
バージョンが確認できればOK

MySQLを起動し接続
デフォルトでrootにパスワードが設定されてしまっている。
まずはpasswordを調べ、接続しpassswordの再設定を行っていく必要がある。
```sh
$ sudo systemctl start mysqld
$ sudo cat /var/log/mysqld.log | grep 'temporary password'  #root@localhost: hogehoge [hogehoge]が今回のパスワード
$ mysql -u root -p
Enter password:
```
```sh
mysql > set password = "新たなpassword";
```

MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、  
以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更
```sh
$ sudo vi /etc/my.cnf
```

```sh
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```

編集後はMySQLサーバの再起動が必要
```sh
$ sudo systemctl restart mysqld
```

### データベースの作成
LaravelのTodoアプリケーションを動かす上で使用するデータベースの作成
```sh
$ mysql > create database laravel_app;
```

### Laravelを動かす
laravel_appディレクトリ下の .env ファイルの内容を以下に変更
```sh
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```

laravel_appディレクトリに移動してテーブルを作成
```sh
$ php artisan migrate
```

## 所感
環境構築の流れはイメージできたが、
実際の現場で行うときにするイメージがまだあまりできていない状況です。
[Vagrant][docker]や各サーバなどがどのような現場でどのような使い方をされるのかをもっと知るべきだと感じました。

## 参考サイト
[Laravel 6.x - ReaDouble](#https://readouble.com/laravel/6.x/ja/)

[MySQLが起動しない（The server quit without updating PID file ~~~）
](#https://zenn.dev/ogakuzuko/articles/1d9d20fb5cbef8)

[Laravel & Docker 環境構築 with Laradock](#https://qiita.com/shunichi_com/items/9b09c5949233b88b9a4a)
