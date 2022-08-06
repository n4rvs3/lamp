# install apache

- `https://dlcdn.apache.org//httpd/httpd-2.4.51.tar.gz`
  - `tar zxf httpd-2.4.51.tar.gz`

## apr apr-util

- `wget https://dlcdn.apache.org//apr/apr-1.7.0.tar.gz`
- `wget https://dlcdn.apache.org//apr/apr-util-1.6.1.tar.gz`
  - `mv ダウンロードしたファイル ダウンロードしたapacheフォルダ/srclib/ダウンロードしたやつの名前`

## pcre

- `dnf install gcc gcc-c++ make perl pcre-devel`
- `wget https://sourceforge.net/projects/pcre/files/pcre/8.45/pcre-8.45.tar.gz`
  - `tar -> ./configure -> make`

## expat

- `wget https://github.com/libexpat/libexpat/releases/download/R_2_4_1/expat-2.4.1.tar.gz`
  - `tar -> ./configure -> make -> make install`

## openssl

- `wget https://www.openssl.org/source/openssl-1.1.1l.tar.gz`
  - `tar -> ./config -> make -> make install`

## apache build

- `cd /usr/local/src/httpd-2.4.51`
- `./configure --prefix=/usr/local/httpd --with-included-apr --with-pcre=/usr/local/src/pcre-8.45/pcre-config --enable-mods-shared=all --enable-ssl --with-ssl=/usr/local/ssl --with-mpms-shared=all`
  - `make -> make install`

## apache setting

- `groupadd apache`
- `useradd apache -g apache -s /sbin/nologin`
- `chown -R apache.apache .`
- `vi conf/httpd.conf`
- `vi /usr/lib/systemd/system/httpd.service`

```
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/httpd/bin/apachectl start
ExecReload=/usr/local/httpd/bin/apachectl graceful
ExecStop=/usr/local/httpd/bin/apachectl stop

[Install]
WantedBy=multi-user.target
```

## install firewalld

- `dnf install firewalld`
- `systemctl start firewalld`
- `systemctl enable firewalld`
- `firewall-cmd --add-service=http --zone=public --permanent`
- `firewall-cmd --add-service=https --zone=public --permanent`
- `firewall-cmd --reload`

## test apache

- `systemctl enable httpd --now` -> auto start and starting apache
- `curl localhost:80`
  - `It Works!`
    - ここで表示している内容は`httpd/htdocs/index.html`の内容である。変更可

# install mysql

### ここからは ubuntu を用いた場合。cent 等別 os の場合は適時読み替える。

[Click here for instructions.](https://dev.mysql.com/doc/refman/8.0/ja/installing-source-distribution.html)

- `apt install libpcre3 libncurses-dev cmake`
- `sudo wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-boost-8.0.27.tar.gz`
  - `tar ...`
- `cd mysql-${version}`
- `mkdir bld`
- `cd bld`
- `cmake .. -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/src`
- `make -j 8` ...`-j 並列処理を行うことでより早いコンパイルが可能。エラー(kill task)等が起こる場合は8から数値を下げることで対処。数字は使用しているpcのcpuのコア数で随時変更。`
- `make install`
- `cd /usr/local/mysql`
- `mkdir mysql-files`
- `groupadd mysql`
- `useradd -r -g mysql -s /bin/false mysql`
- `chown mysql:mysql mysql-files`
- `chmod 750 mysql-files`
- `bin/mysqld --initialize --user=mysql`
- `bin/mysql_ssl_rsa_setup`
- `bin/mysqld_safe --user=mysql &`

  - `別タブを開いて bin/mysqladmin password <new password> -u root -p`
    - `enter password` -> 変更前のパスワードを入力
      - `bin/mysql -u root -p`
        - `enter password` -> 先ほど設定したパスワードを入力、変更出来ているかを確認

- `ここまででエラーが発生した場合`
  - `tail <host_name>.err` -> エラーメッセージを確認する
- `https://dev.mysql.com/doc/refman/8.0/ja/postinstallation.html`
- `bin/mysqladmin -u root -p version`

  ```
  bin/mysqladmin  Ver 8.0.27 for Linux on x86_64 (Source distribution)
  Copyright (c) 2000, 2021, Oracle and/or its affiliates.
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Server version 8.0.27
  Protocol version 10
  Connection Localhost via UNIX socket
  UNIX socket /tmp/mysql.sock
  Uptime: 15 hours 36 min 6 sec

  Threads: 2 Questions: 12 Slow queries: 0 Opens: 139 Flush tables: 3 Open tables: 58 Queries per second avg: 0.000
  ```

- `bin/mysql -u root -p`
  - `show databases;`
    ```
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.02 sec)
    ```

# install php

- `apt install libxml2-dev sqlite3 zlib1g-dev`
- `wget https://www.php.net/distributions/php-7.4.27.tar.gz`
  - `tar ...`
- `cd php-${version}`
- `./configure --with-apxs2=/usr/local/httpd/bin/apxs --with-pdo-mysql`
- `make`
- `make test` <- 忘れない！！
- `make install`
- `cp -p php.ini-development /usr/local/php/php.ini`
- `cd /usr/local/php`
  - `vim php.ini`
    ```
    date.timezone = Asia/Tokyo
    mbstring.language = Japanese
    ```
    探す場合は vim を開いて`/`を入力後、検索したい文字を入れることで検索できる。
- `cd /usr/local/httpd`

  apache にて php ファイルを認識させる為の設定を追加する

  - `vim conf/httpd.conf`

    ```
    <IfModule mime_module>
        AddType application/x-httpd-php .php
    </IfModule>
    <IfModule dir_module>
        DirectoryIndex index.html index.php
    </IfModule>

    <!-- 追記 -->
    PHPIniDir "/usr/local/php/php.ini"
    ```

動作確認を行うために phpinfo を記述した index.php を配置する

- `vim htdocs/index.php`
  ```
  <?php echo phpinfo(); ?>
  ```

設定を反映する為に apache の再起度

- `systemctl restart httpd`

ブラウザを起動し、
`localhost:80/index.php`
