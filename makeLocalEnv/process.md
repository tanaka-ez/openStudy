# VagrantでCentOS6.5の環境を構築する

## Vagrantの起動

- CentOS6.5のvagrant boxを登録

```sh
vagrant box add cent65 https://github.com/2creatives/vagrant-centos/releases/download/v0.1.0/centos64-x86_64-20131030.box
```

- Vagrantfileの作成

```
mkdir centos && cd centos
vagrant init cent65
```

- Vagrantfileを下記の通りに編集

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "cent65"
end
```

- vagrantの起動

```
vagrant up
```

- 終了

```
vagrant halt
```

## 仮想マシンにプライベートIPアドレスとホスト名を設定する

- Vagrantfileに下記のように修正

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "cent65"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = "centostest"
end
```

- 起動

```sh
vagrant up
```

- 仮想マシンにログインしてみる

```sh
vagrant ssh
```

# LAMP環境を構築してみる

## Apache,phpをインストール

- ライブラリのインストール

```sh
sudo yum install -y httpd php php-mysql
```

- Apache起動

```
sudo service httpd start
```

- Apacheのテストページが表示されるか確認

[http://192.168.33.10/](http://192.168.33.10/)

- index.phpを配置

```sh
sudo vi /var/www/html/index.php
```

下記のように記述

```php
<?php
  phpinfo();
?>
```

- phpのバージョンが表示されるか確認

[http://192.168.33.10/index.php](http://192.168.33.10/index.php)

## MySQLのインストール

- MySQLは公式のrpmパッケージでインストールする

```sh
sudo yum install -y http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
sudo yum install -y mysql-community-server
```

- MySQL起動

```sh
sudo service mysqld start
```

- スキーマとテーブルの作成

```
mysql -u root
create database test;
use test;
create table tmp(id int(5), text text);
insert into tmp(id, text) values(1, 'tmpText');
```

## phpからMySQLにアクセスし、結果を返す

- phpからMySQLにアクセスして結果を返すコードの記述

`/var/www/html/index.php`を下記の内容に修正

```php
<?php

  $link = mysql_connect('localhost', 'root', '');

  if (!$link) {
    die('接続失敗です。'.mysql_error());
  }

  $db_selected = mysql_select_db('test', $link);

  if (!$db_selected) {
    die('データベース選択失敗です。'.mysql_error());
  }

  $result = mysql_query('SELECT * FROM tmp');

  if (!$result) {
    die('クエリーが失敗しました。'.mysql_error());
  }

  $row = mysql_fetch_assoc($result);
  print($row['id']);
  print($row['text']);
  mysql_close($link);
?>
```

- ブラウザでの確認

[http://192.168.33.10/index.php](http://192.168.33.10/index.php)

## その他コマンド

```sh
# 登録したvagrant boxを削除する
vagrant box remove Box名 PROVIDER
```

