# heartbeats 様提供課題(LAMP 構築)

## 今回の構築を行ったうえでの総まとめは別途 Matome.md に記載。

## 実行記録

1.  (L)AMP のインストール

    - OS は centOS を利用
    - wsl2 で稼働中の docker にコンテナを作成
    - image は[公式リポジトリ](https://hub.docker.com/_/centos)より取得

    以降はコンテナ上で作業するものとする

2.  L(A)MP のインストール

- `cd /usr/local/src` - Apache 本体編
- `curl -OL https://dlcdn.apache.org//httpd/httpd-2.4.51.tar.gz` Apache 本体のダウンロード(リンクは公式サイト記載のミラーリンク)
- `OLオプション`について
- `O`・・・標準出力無しでファイルをダウンロードする場合に利用
- `L`・・・リダイレクト処理
- `tar zxf httpd-2.4.51.tar.gz` ダウンロードしたファイルを解凍
- `zxf`について
- `z`・・・gzip 形式で圧縮する、または圧縮されたアーカイブを展開する(今回は展開)
- `x`・・・アーカイブからファイルを抽出する
- `f`・・・アーカイブファイル名を指定する(今回でいうと公式から curl してきたファイル名)　ほぼ必須と思われる

        - Apache サポートライブラリ編
            - APR APR-utilのインストール
                - `curl -OL https://dlcdn.apache.org//apr/apr-1.7.0.tar.gz` Apache HTTP ServerのサポートライブラリであるAPRのインストール
                    - APRを利用することにより環境の違いを吸収するAPIを提供
                - `tar zxf apr-1.7.0.tar.gz` APRの解凍
                - `mv apr-1.7.0 httpd-2.4.51/srclib/apr` [こちらに従い](https://httpd.apache.org/docs/current/en/install.html#:~:text=building%20Apache%20httpd%3A-,APR%20and%20APR%2DUtil,-Make%20sure%20you)解凍したAPRを`httpd-{version}/srclib/apr`に移動
                - `curl -OL https://dlcdn.apache.org//apr/apr-util-1.6.1.tar.gz` APR-utilのダウンロード
                - `mv apr-util-1.6.1 httpd-2.4.51/srclib/apr-util` 上記APRと同じ手順で`httpd-{version}/srclib/apr-util`に移動

            - ボーナス編(dnfのインストール)
                - [公式サイト](https://httpd.apache.org/docs/current/en/install.html#download)と[提示して頂いたサイト](https://suzuki-kengo.net/lamp-sourceinstall/)を両方見ながら進めていたところ、`dnf`というコマンドを発見。調べてみたところ、Python2系で作成されている`yum`の後継となるPython3系で作成されたパッケージ管理が`dnf`らしい。`centOS:7`には標準搭載されていなかったので、後に困らないようにこの時点でインストールしておく。
                    - `yum -y install dnf`
                    - `dnf update`

            - PCREのインストール
                - `dnf install gcc gcc-c++ make`・・・後ほど必要となる前提ライブラリ等のインストール
                    - 注意ポイント！！！！！！！！
                        - 現在、pcreをインストールする場合、`curl`や`wget`では直接ダウンロードできない様子？[PCRE公式サイト](http://www.pcre.org/#:~:text=Note%20that%20the%20former%20ftp.pcre.org%20FTP%20site%20is%20no%20longer%20available.%20You%20will%20need%20to%20update%20any%20scripts%20that%20download%20PCRE%20source%20code%20to%20download%20via%20HTTPS%2C%20Git%2C%20or%20Subversion%20from%20the%20new%20home%20on%20GitHub%20instead.)によると
                        `ftp.pcre.org`の利用は出来なくなり、現在ダウンロードする場合は[こちらのGithub](https://github.com/PhilipHazel/pcre2)からダウンロードする必要がある模様。

                        - 今回の環境は`Docker` + `wsl2` なので
                            - githubよりファイルをダウンロード
                            - wsl2上で `cd ${マウントされているホスト側のダウンロードディレクトリ}`
                            - `docker cp ./pcre2-10.39.tar.gz c346df0f214e:/usr/local/src` でコンテナ内の`/usr/local/src`ディレクトリにダウンロードしたファイルをコピーできる。

                - `tar zxf pcre2-10.39.tar.gz` 解凍
                - `mv pcre2-10.39 httpd-2.4.51/srclib`
                - `cd httpd-2.4.51/srclib/pcre2-${hogehoge}` → `./configure`
                    - **※ここでエラー**
                        - `configure: error: pcre-config for libpcre not found. PCRE is required and available from http://pcre.org/`
                        - どうやら、PCRE2はサポートされていない模様。。。無論、独自に設定を書き換えて導入することも可能なようだが今回の趣旨とはずれるので割愛。大人しく対応している物をインストールしなおす。[以下参考文献](https://www.spinics.net/lists/apache-users/msg113046.html)
                - PCREの再ダウンロード・コピー(手順は上記と同様の為割愛)
                - `./configure` -> `make`
                - rootに戻って `./configure` -> `make` -> `make install`
                    - ここでエラー
                        `xml/apr_xml.c:35:19: fatal error: expat.h: No such file or directory`
                        `expat`がないといわれているのでインストールする(次項)

            - Expatのインストール
                - 先ほどdnfをインストールした際に`wget`もインストールしていたので、今回はwgetを用いてダウンロードする
                - `wget https://github.com/libexpat/libexpat/releases/download/R_2_4_1/expat-2.4.1.tar.gz`
                - `tar zxf expat-2.4.1.tar.gz`
                - `mv expat-2.4.1 ./httpd-2.4.51/srclib`
                - `cd ./httpd-2.4.51/srclib` -> `./configure` -> `make` -> `make install`

            - OpenSSLのインストール
                - `wget https://www.openssl.org/source/openssl-1.1.1l.tar.gz`
                    - エラー発生　`ERROR: cannot verify www.openssl.org's certificate, issued by '/C=US/O=Let\'s Encrypt/CN=R3':

        Issued certificate has expired.`証明書が期限切れと言われてしまった。認証を無視するオプションがある為今回はそちらを利用するが、構築終了後改めて確認しておきたい。 - 改めて`wget --no-check-certificate https://www.openssl.org/source/openssl-1.1.1l.tar.gz` - `tar zxf openssl-1.1.1l.tar.gz` - `mv openssl-1.1.1l ./httpd-2.4.51/srclib` - `cd ./httpd-2.4.51/srclib/openssl-1.1.1l` -> `./config` -> `make` -> `make install`

        - Apache ビルド編
             - `./configure \

        --prefix=/usr/local/httpd \
        --with-included-apr \
        --with-pcre=/usr/local/src/httpd-2.4.51/srclib/pcre-config/pcre-config \
        --enable-mods-shared=all \
        --enable-ssl \
        --with-ssl=/usr/local/ssl \
        --with-mpms-shared=all`

            - `make`
            - `make install` -> エラー `error: cannot install `libaprutil-1.la' to a directory`
                - 直接的な原因が不明 -> ひとまず `make clean`
                    - 再度 `./configure` -> `make` -> `make install` で無事通過！

        - Apache 設定編
            - `cd /usr/local/httpd`
            - `groupadd apache` ・・・Linuxでのグループ作成
            - `useradd apache -g apache -s /sbin/nologin`
                - `-g` ・・・ユーザーの所属グループを指定できる。今回の場合は先ほど作成したapacheグループ
                - `-s` ・・・ユーザーのログインシェルを指定できる。今回指定したnologinに関しては[こちら](https://qiita.com/LostEnryu/items/9b0c363877581dc1171f#false%E3%81%A8nologin)が分かりやすかった。
            - `chown -R apache.apache .` ・・・所有権を変更。今回はユーザー所有権とグループ所有権両方ともにapacheに変更
            - `vi conf/httpd.conf` ・・・ `User` と `Group` の値をdeamonからapacheに変更。


        **ここでdocker上でcentOSを動作させる場合systemctlが初期状態では使えない事実が発覚**

        - 永続化等していなかった(コンテナは構築終了後削除する予定だった)ので一から導入し直すハメに・・・
        知識の定着も兼ねて時間を図りつつもう一度一から設定し直す。

        - 結果先ほどまでに進んでいた工程まで20分ほどで戻ってこれた。無論、ライブラリのダウンロード時間であったりmake中の時間が半分ほどを占めているのだが。しかし、先ほどとは打って変わってコマンドの意味を調べずとも理解し、その上で実行することが出来たので少しではあるが定着していると思われる。では続く・・・

        - Apache 設定編(再度)
            - `vi /usr/lib/systemd/system/httpd.service` ・・・apacheのユニット定義ファイル。ここでsystemctlの設定を定義する(?)(よくわかっていない)
            - firewalldが入っていないのでインストールする
                - `dnf install firewalld`
                - `systemctl start firewalld`
                - `systemctl enable firewalld`
            - `firewall-cmd --add-service=http --zone=public --permanent`
            - `firewall-cmd --add-service=https --zone=public --permanent`
            - `firewall-cmd --reload`
            - `systemctl enable httpd --now` ・・・apacheの自動起動＋今すぐ起動する

            - `curl localhost:80`
                - 参考サイトでは`curl http://192.168.10.5`となっているが、今回はコンテナ側のポート80番をホスト側の80番に割り当てているので`localhost:80`で確認

3.  LA(M)P のインストール

    - 当項目に関しては今井さんが躓いたとおっしゃっていたので、参考サイトを見ずに、ドキュメント一本で実装してみようと思う。
    - 準備編
      [こちら公式サイト](https://dev.mysql.com/doc/refman/8.0/en/source-installation-prerequisites.html)に記載の前提条件を満たして行く。

      - `dnf`のコマンドを調べる

        - [こちら](https://atmarkit.itmedia.co.jp/ait/articles/2001/09/news018.html)を参考に、依存パッケージを dnf でインストール可能か否かを検証
        - `dnf info cmake` -> 可能
        - `boost`というライブラリが必要らしいが、コンパイルには特定の一致したバージョンが必要らしい・・・？
          例によると`-DWITH_BOOST=/usr/local/boost_version_number`というオプションがあるらしい。詳細はよくわからないがおそらくインストール済みのバージョン指定を行うオプションであると推測できる
        - `ncurses` ・・・CUI においてのキー入力、カーソル等を制御するライブラリとある。[公式サイト](<https://invisible-island.net/ncurses/ncurses.html#:~:text=diff%27s%20(main%20site)-,Download%20tack,-HTTP>)を見ると`development source`とあるが開発用だろうか？
          - `dnf info ncurses-dev`みたいに入力しても出てこない。
          - [こちら](https://pkgs.org/search/?q=ncurses)で見つけた、`ncurses-devel`なるものがあるらしい。
        - dnf にて上記をインストールしようとすると、どうやら dns の設定が変わってコンテナから外部へのアクセスができなくなっていた模様。応急処置として resolve.cnof を編集。
        - `cmake`と`ncurses-devel`をインストールしたので mysql をインストールしていく
        - `wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27.tar.gz` -> `tar zxf mysql-8.0.27.tar.gz`
        - [この部分](https://dev.mysql.com/doc/refman/8.0/en/installing-source-distribution.html#:~:text=%E5%9F%BA%E6%9C%AC%E7%9A%84%E3%81%AA%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%B7%E3%83%BC%E3%82%B1%E3%83%B3%E3%82%B9%E3%81%AF%E6%AC%A1%E3%81%AE%E3%82%88%E3%81%86%E3%81%AB%E3%81%AA%E3%82%8A%E3%81%BE%E3%81%99%E3%80%82)を参考にしていく

        - `cmake`部分でエラー。`CMake Warning at CMakeLists.txt:82 (MESSAGE): Please use cmake3 rather than cmake on this platform` -> おそらく私の環境の場合は cmake3 を利用せよとのエラーだと思われるので早速インストール
        - `dnf install cmake3` -> エラー。cmake3 といったパッケージはないとの記載。公式のダウンローダーからインストール。
        - `wget https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1.tar.gz` -> `tar zxf ...`

        ```
        CMake Warning at CMakeLists.txt:347 (MESSAGE):
        Could not find devtoolset compiler/linker in /opt/rh/devtoolset-10


        CMake Warning at CMakeLists.txt:349 (MESSAGE):
        You need to install the required packages:

        yum install devtoolset-10-gcc devtoolset-10-gcc-c++ devtoolset-10-binutils



        CMake Error at CMakeLists.txt:351 (MESSAGE):
        Or you can set CMAKE_C_COMPILER and CMAKE_CXX_COMPILER explicitly.
        ```

        cmake でのエラーが発生してしまった。おそらくインストールの仕方が良くなかったのかと思われる。また、ネットで調べてみると dnf で cmake の 3 系がインストールできるとのこと。自身の環境では 2.8 系が最新と出る。。。

        `https://pkgs.org/search/?q=cmake`にて確認してみると
        centOS8 系では 3 がインストールできるようなので 3 回目の再構築を行う。

        再度インストール等行った結果、cmake は 3 系にアップデートされたのを確認できたが今度は systemctl が使えなくなってしまった。ので 4 回目のコンテナ作成。。。

        [こちら](https://djeeeno.blogspot.com/2019/03/20190321-01-docker-failed-to-early-mount-api-filesystems-freezing.html)や[こちらの質問](https://teratail.com/questions/83479)を参考に docker-compose.yml を編集、実行してみたが結局うまくいかず。

        ここで膠着してしまうのは本望では無いので環境を移行してみる。また、centos7 系では追記が必要ではあったものの問題なく動いていた為 8 系で通らない理由は後ほど検証・調査する

## WSL2 上の ubuntu20 系に LAMP 環境構築

    [こちらのissue](https://github.com/Ibuki-Matsumoto/LAMP_sourceinstall/issues/1)にも記載の通り、wsl上ではsystemctl等のsystemdを必須とするコマンドが利用できない為、wsl2を用いた開発環境を断念。以下macOS上で構築することとする。

## 追記(12/16)

[こちら](https://github.com/Ibuki-Matsumoto/LAMP_sourceinstall/blob/master/README.md#:~:text=cmake%20%E3%81%A7%E3%81%AE%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%8C%E7%99%BA%E7%94%9F%E3%81%97%E3%81%A6%E3%81%97%E3%81%BE%E3%81%A3%E3%81%9F%E3%80%82%E3%81%8A%E3%81%9D%E3%82%89%E3%81%8F%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%AE%E4%BB%95%E6%96%B9%E3%81%8C%E8%89%AF%E3%81%8F%E3%81%AA%E3%81%8B%E3%81%A3%E3%81%9F%E3%81%AE%E3%81%8B%E3%81%A8%E6%80%9D%E3%82%8F%E3%82%8C%E3%82%8B%E3%80%82%E3%81%BE%E3%81%9F%E3%80%81%E3%83%8D%E3%83%83%E3%83%88%E3%81%A7%E8%AA%BF%E3%81%B9%E3%81%A6%E3%81%BF%E3%82%8B%E3%81%A8%20dnf%20%E3%81%A7%20cmake%20%E3%81%AE%203%20%E7%B3%BB%E3%81%8C%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%A7%E3%81%8D%E3%82%8B%E3%81%A8%E3%81%AE%E3%81%93%E3%81%A8%E3%80%82%E8%87%AA%E8%BA%AB%E3%81%AE%E7%92%B0%E5%A2%83%E3%81%A7%E3%81%AF%202.8%20%E7%B3%BB%E3%81%8C%E6%9C%80%E6%96%B0%E3%81%A8%E5%87%BA%E3%82%8B%E3%80%82%E3%80%82%E3%80%82)にて記述している cmake のインストールミスではないかとの推測は間違いであった。きちんとエラーメッセージを確認すると依存パッケージが足りていないとの記述があった。
こうしたミスはきちんとエラーメッセージを確認することで今後無くしていきたい。

また、構築にあたっての流れを記した[こちらのファイル](./chart_build.md)も作成した。

## MySQL のインストール(再度)

実行記録については[こちら](./chart_build.md)を参照。ここではエラーハンドリング・コマンドの解説についてのみ記載。
結論だけ先に述べると、centOS->ubuntu に移行したことでエラーが極端に減った。os の差があるのかもしれない。

- `No mysys timer support detected!`

  - [こちらの issue](https://github.com/Ibuki-Matsumoto/LAMP_sourceinstall/issues/2)に記載しているが、centOS8 系でエラーが発生。おそらく system 周りのパッケージ、もしくはライブラリが不足している物と思われた。issue にも記載している通り、os を ubuntu に変更することで回避。原因は不明。

- `Your password has expired. To log in you must change it using a client that supports expired passwords.`

  - パスワードの有効期限が切れているとのこと。変更を行う。詳細は`chart_buildの98行目移行を参照`

- `-DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/src`
  - `cmake`のオプション。文字通りだが、`-DDOWNLOAD_BOOST=1`で自動で mysql のバージョンに合った boost をダウンロードしてくれる。`-DWITH_BOOST=/usr/local/src`でどこに boost があるのかを指定。

## PHP のインストール

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
![phpinfo](https://imgur.com/a/EkpjD7P)

動作確認完了！
