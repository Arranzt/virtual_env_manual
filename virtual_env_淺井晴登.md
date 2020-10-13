# 環境構築手順書  

***
|インストールが必要なソフト|バージョン|
|CentOS|7.8.2003|
|Nginx|1.16.1|
|PHP|7.3|
|Laravel|6.0|
|MySQL|5.7|

***

## ①Vagrantを用いた仮想環境構築について

### 仮想環境とは  
現在のOS上で別のOSを動作させることができるようになる技術です。  
ネットワーク上に公開されているWebアプリケーションと同様のアプリケーションを自分のPC上で動かして動作検証などに使用するために用います。今回はVagrantを用いて環境構築を行っていきます。  
なお、アプリケーション公開までの一連の流れを**プロビジョニングフレームワーク**と言い、その中で最下層に当たる**Bootstrappping**（OSを起動して利用可能な状態になるまでに自動実行される処理）の構築を行うための設定が今回行っていく設定になります。

### VirtualBoxのインストールについて 

**VirtualBox**とは
使用しているPCにインストールされているOSとは別の仮想的なOSを立ち上げるためのソフトウェアです。
例えば、Macを使用しているのであればメモリやストレージなどのリソースを使用して 内蔵されている既存のMacOSとは別でLinuxやWindowsのOSを動かすことができます。
この内蔵されている既存のOSを ホストOS、仮想化ソフトウェアを使用して追加されたOSをゲストOSといいます。
VirtualBoxはこのゲストOSを立ち上げるために使用します。

MacとWindowsでVirtualBox・Vagrantそれぞれのインストールの方法が異なりますので自分が使用しているOSに対応した箇所を読み進めてください。

下記のサイトからそれぞれのdmgファイルをダウンロード後、インストールを進めてください。

[Virtual Box公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)

※ Vagrantの最新バージョンがVirtualBoxの最新バージョンに対応していないため、Virtual Boxはver6.0.14をインストールするようにしてください。

<Macの方>

`OS X hosts` を選択しましょう。  
以下のコマンドを実行してVirtualBoxのウィンドウが表示されれば正常にインストールされています。
```
$ virtualbox
```
コマンド実行後は入力を受け付けない状態となるため、`Control + c`を押してください。

<Windowsの方>

`Windows hosts` を選択しましょう。  
デスクトップに `Oracle VM VirtualBox` のアイコンがあるか確認してください。あればインストールは完了しています。

### Vagrantのインストールについて 

**Vagrant**とはVirtualBoxなどを利用してVMを効率的に構築するためのソフトウェアです。  
指定したOSとソフトウェアをVMにインストールする作業を全自動で行ってくれます。  

全自動でVMを構築出来るということは、同じ指定をすれば正確に同じ環境を再現することが可能ということです。  
そのため、VMを作り直してクリーンな作業環境に戻したり、複数人が同等の環境を利用したりといったことが出来るようになります。

<Macの方>

下記コマンドで簡単にインストールすることが可能です。
```
$ brew cask install vagrant
```
※ `brew cask`  
macOSアプリケーション、フォント、プラグイン、そしてオープンソースではないソフトウェアなどをインストールする時に利用するコマンドになります。
実際には、`brew cask install アプリ名`といった形式でコマンドを実行します。

<Windowsの方>

下記のサイトからインストールします。

[Vagrant公式](https://www.vagrantup.com)

インストールが終わったら vagrant コマンドが使用可能かどうか確認するため以下のコマンドを実行しましょう。
```
$ vagrant -v
```
バージョンの確認ができたら問題なく使用が可能な状態です。  
なお、のちに起動したゲストOSにログインしていきます。  
その際に 本来であれば `ssh` というコマンドを使用するのですが、Windowsのコマンドプロンプトには `OPENSSH` というソフトウェアがインストールされていないため、 `ssh` コマンドが使用できません。

`OPENSSH` ソフトウェアをインストールして、`ssh`コマンドを使用できるようにするという方法もありますが、SSH接続専用のターミナルソフトを導入するほうが、より手軽のためここでは [こちら](http://nanno.dip.jp/softlib/man/rlogin/#INSTALL) から `Rlogin` というターミナルソフトウェアをインストールしておきましょう。
64bit版の実行プログラムzipをダウンロードして解凍・インストールしてください。

### vagrant boxのインストールについて

`vagrant box`とは仮想環境を構築するための元となるOSのイメージファイルになります。  
`Vagrant`を使用すればboxという形でOSファイルをダウンロードし、それを元にコマンドで仮想環境を構築・管理することができます。

それでは実際にインストールを行います。 
今回利用するOSはLinuxのCentOSのバージョン7になります。  
下記のコマンドを実行して下さい

```
vagrant box add centos/7
```

※ `vagrant box add box名`  
boxを追加するためのコマンドになります。  
今回は`centos/7`という項目がbox名となります。

実行後、下記のような選択を求められるかと思います。  
今回使用するソフトは`VirtualBox`のため、3を選択してenterを押しましょう。
```
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```

下記のように表示されるか確認しましょう。
```
Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!
```

最後に、下記コマンドを実行して、バージョンが確認できればOKです。
```
[vagrant@localhost ~]$ cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
```

### vagrant用ディレクトリの作成  
以下のいずれかのディレクトリの下に、vagrant_test_appという名前でディレクトリを作成下さい。  
- 自分の作業用ディレクトリ
- デスクトップ  

ここで作成したディレクトリを元に、今後必要なデータを扱っていきます

それでは以下のコマンドを実行してみましょう。
```
$ cd 自分の作業用ディレクトリorデスクトップへのパス
$ mkdir vagrant_test_app
```

※ `cd`  
 「change directory」の略で、ディレクトリやファイル間を移動する時に利用するコマンドになります  
※ `mkdir`  
 「make directory」の略で、そのままディレクトリを新規作成するコマンドになります

***
## ②ipアドレスの設定とvagrantの起動  
### vagrantの起動準備について

以下のコマンドを実行し、vagrant_test_appディレクトリ下でvagrantを起動するための準備を行って下さい。
```
$ cd vagrant_test_app
$ vagrant init centos/7
```
※ `vagrant init`  
vagrantを初期化するコマンドになります。  
コマンドを打つと、`Vagrantfile`という設定ファイルが作成され、このファイルを元にvagrantが起動します。  
実際の実行時については、initの後にbox名がくるため、`vagrant init box名`といったコマンドとなります。  
今回は先ほどダウンロードしたboxを使用することになりますので、box名＝`centos/7`としてコマンドを実行します。  

実行後問題なければ以下のような文言が表示されます
```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Vagrantfileの編集について

vagrantが無事に初期化出来ましたら、vagrant_test_appのフォルダ内を検索し、`Vagrantfile`という名前のファイルを編集します。  

下記コマンドを実行し、Vagrantfileの編集画面を開いて下さい。
```
$ vi Vagrantfile
```

今回行う編集は、三箇所です。
```
config.vm.network "forwarded_port", guest: 80, host: 8080

config.vm.network "private_network", ip: "192.168.33.19"
```
上記二箇所の # が付いているのを外して下さい。

※ipとポートについて  
インターネットは元々、「IPアドレス＋ポート番号」で通信を行っています。  
上記の設定にて、サーバ側のポート番号とIPアドレスを指定することになります。  
なお、今回は`192.168.33.19`というipを用いて環境構築を行っていきます。  

また以下の箇所はコメントインし、変更を加えてください。
```
config.vm.synced_folder "../data", "/vagrant_data"
↓
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```

※ `config.vm.synced_folder`  
ホストOSとゲストOSのフォルダを同期する設定を行うコマンドになります。  
この設定にて、ホストOS (Mac or Windows) の`vagrant_test_app`ディレクトリ内とゲストOS (Vagrant) の `/vagrant` のディレクトリ内をリアルタイムで同期出来るようになります。  
なお、`./` はカレントディレクトリ(vagrant_test_app)を示しています。

### Vagrantを使用したゲストOSの起動

以上で仮想環境を構築する準備は完了となります。  
`Vagrantfile`があるディレクトリにて以下のコマンドを実行して、ゲストOSを起動して下さい。
```
$ vagrant up
```

### ゲストOSへのログイン

起動が正常に終了すれば、皆さんが使用しているPCのOSであるホストOSの上に全く別のゲストOSが立ち上がったことになります。
では実際にゲストOSにログインしていきましょう。  
ここでもMacとWindowsでログインの方法が異なりますので、自分の使用しているPCに対応する箇所を読み進めてください。

<Macの方>

`ssh`は「Secure Shell」の略で、リモートコンピュータと通信するためのプロトコルです。
主にゲストOSへのログインを行う際に利用します。
今回は、VirtualBoxが提供するネットワークを通じてターミナル上でホストOSからゲストOS(リモートマシン)にログインします。
作成した `vagrant_test_app`ディレクトリに移動して下記のコマンドを実行しましょう。
```
$ vagrant ssh
```
コマンドを実行した後、以下のような表記になっていればゲストOSにログインしていることになります。
```
[vagrant@localhost ~]$
```

<Windowsの方>

前のセクションでインストールしたターミナルソフトウェアを使用します。
以下は、`RLogin`でのログイン方法です。

まずは `Rlogin`を起動後 「作成」ボタンを押下しましょう。
その後、下記手順で入力を進めて下さい。  
①ホスト名→`127.0.0.1`と入力  
②TCPポート→`2222`と入力  
③ログインユーザ名→`vagrant`と入力   
④SSH認証鍵をクリック  
⑤「SSH認証鍵」のボタンを押下すると以下の画面が開かれるため`vagrant_test/.vagrant/machines/default/virtualbox` 下のファイル名を`private_key` を指定して開くボタンを押しましょう。

以上で設定が完了したので、設定した接続情報を選択して「OK」ボタンでゲストOSにログインしましょう。

※初回ログイン時に注意が表示されますが、OKを押してください。

ログインが完了しましたら、Macと同様で以下の文言が表示されれば完了となります。
```
[vagrant@localhost ~]$
```

***

## ③DBのインストール

今回インストールするデータベースは`MySQL`となります。  
versionは5.7を使用します。

`centos7`は、デフォルトで`MariaDB`というデータベースがインストールされていますが、`MariaDB`は`MySQL`と互換性があるDBなので気にせず、`MySQL`のインストールを進めていきます。

`rpm`に新たにリポジトリを追加し、インストールを行います。
```
[vagrant@localhost ~]$ sudo wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
[vagrant@localhost ~]$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
[vagrant@localhost ~]$ sudo yum install -y mysql-community-server
```
※ `sudo`  
「substitute user do」の略や，「superuser do」の略と言われているコマンドです。  
「指定したコマンドを別のユーザー権限で実行する」時に利用します。
今回の場合、`vagrant ssh` (Windowsの場合はuserとpasswordをvagrantとして) を実行してゲストOSにログインすると、ログインユーザーは`vagrant`となります。
しかしながら、`vagrant`というユーザーは、`root`ユーザーによって作成されたユーザーであり、`root`ユーザーにしか許されていない操作を実行することが出来ません。
そのため、`root`ユーザーの権限を一時的に借りる必要があることから、`sudo`コマンドの実行を行っています。
ただし、システムの根幹に関わるようなファイルやディレクトリには、システム管理者の`root`ユーザーにしか操作できないような適切な権限が設定されている場合が多いため、`sudo`を付けて取り返しのつかない操作をしてしまった場合は回復が出来ない可能性があります。このコマンドを実行する際には必ず慎重な操作を心がけて下さい。

※ `wget`  
「World Wide Web get」の略であり、Webサーバなどからファイルをダウンロードする際に実行するコマンドとなります。
`wget　URL`という書式を用いることで、Web上に上がっている、アーカイブファイルやソースファイルなどをダウンロードすることができます。

※　`rpm`  
「RedHat Package Manager」の略であり、RPMパッケージを管理、操作する時に利用するコマンドになります。
正確には、RedHat社が開発したパッケージ管理システムのことをRPMといい、そのRPMで扱うパッケージのことをRPMパッケージと呼びます。従って、そのRPMパッケージを管理、操作するコマンドがrpmとなります。
`rpm [オプション] [パッケージ名]または[パッケージファイル名]または[ファイル名]`といった書式でコマンドを実行することで、RPMパッケージのインストール、アンインストール、情報の照会、検査などなど、RPMパッケージに対して色々な操作ができます。
なお、今回`-Uvh`というオプションを付与しています。このオプションはパッケージのアップグレードを行う際に利用するオプションになります。

※ `yum`  
「Yellowdog Updater Modified」の略で、LinuxのRPM Package Managerのパッケージを管理するためのコマンドになります。
元々は、Yellow Dog LinuxというRedHat系Linuxディストリビューションがあり、そこで使われていたパッケージ管理システムはYellowdog Updaterでした。そのため、`yum`というコマンド名は、Yellowdog Updaterを修正した（Modified）ところからきています。
`yum [サブコマンド] [パッケージ名]`という書式を用いることで、パッケージのインストール、アンインストール、アップデート、検索などなど、パッケージに対して様々な操作ができる他、ネットワーク上のパッケージを探して、ネットワーク経由でインストールなどを行ったり、依存関係にあるパッケージも一緒にインストールしてくれます。

インストールが完了しましたら、下記コマンドを実行してみましょう。
バージョンが確認出来たらインストール完了となります。
```
[vagrant@localhost ~]$ mysql --version
```

次にMySQLを起動し接続を行います。
接続の際、事前にパスワードを確認しておく必要がありますので、下記コマンドを実行し、一旦、MySQLを起動後、デフォルトのパスワードを確認して下さい。
```
[vagrant@localhost ~]$ sudo systemctl start mysqld
[vagrant@localhost ~]$ sudo cat /var/log/mysqld.log | grep 'temporary password' 
```
※　`cat`  
「concatenate（連結する）」の略で、ファイルの内容を標準出力したり、複数のファイルを連結して、標準出力するときに用いるコマンドです。
具体的には、`cat [ファイル名]`にてファイルの内容を標準出力、`cat [ファイル名] [ファイル名]`にて複数のファイルを連結して、標準出力します。

※ `grep`  
「global regular expression print(ファイル全体から/正規表現に一致する行を/表示する)」の略で、左辺の実行結果の中から、grepで指定した文字列と一致する行や文字列を持つファイルやディレクトリのみを出力するコマンドになります。

※　`|`  
`|` はパイプラインといって`左辺 | 右辺`と指定してコマンドを実行することで、左辺の実行結果を右辺に引き渡して右辺を実行することができる演算子になります。

コマンド実行後、下記のように表示されれば、パスワードの確認完了となります。  
`hogehoge`と記載されている箇所に存在するランダムな文字列がパスワードとなります。
```
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```

では先程出力したランダム文字列をコピー後、再度以下のコマンドを実行し、パスワード入力時にペーストしてください。
```
[vagrant@localhost ~]$ mysql -u root -p
[vagrant@localhost ~]$ Enter password:
```

下記のように表示されれば、MySQLへのログイン完了となります。
```
mysql >
```

次にpasswordの変更を行います。
MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。

```
[vagrant@localhost ~]$ sudo vi /etc/my.cnf
```
```
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加下さい
validate-password=OFF
```

編集後はMySQLサーバの再起動が必要です。

```
[vagrant@localhost ~]$ sudo systemctl restart mysqld
```

以上で設定が変更となりましたので、再度MySQLにログインし、新しいパスワードを設定しましょう。
```
mysql > set password = "新たなpassword";
```

以上で、MySQLの導入と設定が完了となります。

### データベースの作成

実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。
```
mysql > create database laravel_test_app;
```
`Query OK`と表示されたら作成は完了となります。

***
## ④Nginxのインストール

Nginxの最新版をインストールしていきます。
Nginxでは[公式](http://nginx.org/en/linux_packages.html)に `yum` のリポジトリを公開しており、その `yum` リポジトリからインストールする方法を今回は採用します。
viエディタを使用して以下のファイルを作成して下さい。
```
[vagrant@localhost ~]$ sudo vi /etc/yum.repos.d/nginx.repo
````
書き込む内容は以下になります。
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。
```
[vagrant@localhost ~]$ sudo yum install -y nginx
[vagrant@localhost ~]$ nginx -v
```
Nginxのバージョンが確認出来たら、インストール完了となります。  
次に、Nginxの起動をしましょう。
```
[vagrant@localhost ~]$ sudo systemctl start nginx
```

***
## ⑤PHP version7.3のインストール

次はPHPをインストールしていきます。

yumコマンドを使用してPHPをインストールした場合、古いバージョンのPHPがインストールされてしまいます。
Laravelを動作させるにはPHPのバージョン7以上をインストールする必要があるため
yumではなく外部パッケージツールをダウンロードして、そこからPHPをインストールする方法を採用します。

まずはdevelopment tools というグループパッケージを指定してインストールします。
このコマンドを実行することによりgitなどの開発に必要なパッケージを一括でインストールできます。
```
[vagrant@localhost ~]$ sudo yum -y groupinstall "development tools"
```

次に、`epel`をインストールします。
`epel`とは「Extra Packages for Enterprise Linux」の略で、標準のリポジトリでは提供されないパッケージを使うことができるようになります。  

> Extra Packages for Enterprise Linux (or EPEL) is a Fedora Special Interest Group that creates, maintains, and manages a high quality set of additional packages for Enterprise Linux, including, but not limited to, Red Hat Enterprise Linux (RHEL), CentOS and Scientific Linux (SL), Oracle Linux (OL).  

```
[vagrant@localhost ~]$ sudo yum -y install epel-release wget
```

次に、`Remi`をインストールしましょう。
`Remi`とは、Remi Collect氏がメンテナンスされているリポジトリで、最新のPHPが使え、その他のソフトウェアを、できる限りそのままのバージョンのまま利用できるようにするためのパッケージとなります。
```
[vagrant@localhost ~]$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
[vagrant@localhost ~]$ sudo rpm -Uvh remi-release-7.rpm
```
最後に、phpのバージョン7.3をインストールしましょう。
```
[vagrant@localhost ~]$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
```
さて、PHPのバージョンが確認できれば、作業は完了となります。  
```
[vagrant@localhost ~]$ php -v
```

***
## ⑥Laravel version6.0のインストール

### Laravelの環境構築に必要な事前のセットアップ  

<MacOS Xの方>

### composerのインストール

まずはcomposerという依存関係管理のためのパッケージをインストールしましょう。  
下記のコマンドを順に実行してください。
```
[vagrant@localhost ~]$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
[vagrant@localhost ~]$ php -r "if (hash_file('sha384', 'composer-setup.php') === '795f976fe0ebd8b75f26a6dd68f78fd3453ce79f32ecb33e7fd087d39bfeb978342fb73ac986cd4f54edd0dc902601dc') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
[vagrant@localhost ~]$ php composer-setup.php
[vagrant@localhost ~]$ php -r "unlink('composer-setup.php');"
```

その後、composerをどのファイルからでも利用できるように、パスを通します。  
コマンドは[公式ドキュメント](https://getcomposer.org/doc/00-intro.md#globally)に記載があります  
```
[vagrant@localhost ~]$ mv composer.phar /usr/local/bin/composer
```
無事に実行できれば、パス通しが完了となります。
下記コマンドにて、バージョンを確認しましょう。
```
[vagrant@localhost ~]$ composer -v
```

なお、失敗してしまった場合、下記のようなコマンドが出る場合があります。
```
mv: cannot move 'composer.phar' to '/usr/local/bin/composer': Permission denied
```
上記、パーミッションが原因でエラーとなっていますので、適宜、パーミッションも変更する必要があります。  

まず、現在のパーミッションを確認しましょう
```
[vagrant@localhost ~]$ ls -l
```

以下のような情報が表示されたでしょうか
```
-rwxr-xr-x. 1 vagrant vagrant 1994202 Oct  7 15:54 composer.phar
```

`-rwxr-xr-x`がファイルモードと呼ばれる部分になります。  
3文字ずつが1つのブロックになっており、それぞれのパーミッションの状態が示されています。  
`rwx`:オーナーのパーミッション  
`r-x`:グループのパーミッション  
`r-x`:その他のユーザのパーミッション  

適宜、下記コマンドで権限を付与しましょう
```
[vagrant@localhost ~]$ chmod g+w composer.phar
```
※「誰に、どのような権限を追加するのか」をコマンドで指定します。    
`g+w`は`g`がgroup、`+`が権限の追加、`w`がw（書き込み）権限の追加を表します。  

権限付与後、再度`composer -v`にてバージョン確認をしてみましょう。  
パーミッションの変更後もエラーが出た場合には、sudoの付与によるコマンドの実行で解決できる場合がありますので、下記コマンドを実行ください。
```
[vagrant@localhost ~]$ sudo mv composer.phar /usr/local/bin/composer
[vagrant@localhost ~]$ composer -v
```

<Windows7 or 10の方>

Composerをインストールします。[こちらから](https://getcomposer.org/download/)ダウンロードし、インストールを実行しましょう。インストール完了後、コマンドプロントにて `composer -v` のコマンドを実行し、Composerのバージョンが確認できれば完了です。

Node.jsをインストールします。[こちらから](https://nodejs.org/en/download/)ダウンロードし、インストールを実行しましょう。インストールのウィザードに沿って実行すればインストールが完了します。

コマンドプロントにて`cd /`を打ち、Cドライブ直下に移動して下さい。そこで`node -v`のコマンドを打ちましょう。もしNode.jsのバージョン情報が表示されなかったら環境変数設定にて、右記を追記して下さい。`C:\Program Files\nodejs`（やり方は同じです。）

追記したら、再度コマンドプロントを開き直し、`node -v`と`npm -v`で、Node.jsをnpmのバージョンの確認を行ってください。次は問題なく表示されると思います。

### Laravelのインストール

Install方法は、2種類ありますが今回は、下記のコマンドで実行します。  
詳細は[公式ホームページ](https://readouble.com/laravel/6.x/ja/installation.html)の「Composer Create-Project」をご覧下さい。
```
[vagrant@localhost ~]$ composer create-project laravel/laravel --prefer-dist laravel_test_app 6.0
```
今回は laravel_test_app という名前のプロジェクトにしました。  
なお、GitHubなどでリポジトリを配信している場合、`git clone`でソースを落としてくる(prefer-source)か、zipでダウンロードする(prefer-dist)か選ぶことができます。コマンド内にあるオプションで`--prefer-dist`というのがありますがこれは、後者を指しており、高速でダウンロード出来るという特徴を持っています。

プロジェクト作成後、下記コマンドでバージョンが確認できればLaravelのプロジェクトの作成完了となります。
```
[vagrant@localhost laravel_test_app]$ php artisan --version
```
### 認証機能の追加

Laravelのversion5.X系とversion6.X系では、認証機能実装までの設定方法が異なります。  
詳細については[Laravel公式](https://laravel.com/docs/6.x/frontend#introduction)ページに記載がありますが、ここではその方法のみをかいつまんで紹介していきます。

まずは、Laravelを構築する上で大切な`Bootstrap`と`Vue`を利用出来るように`laravel/ui`というパッケージをインストールします。

>The Bootstrap and Vue scaffolding provided by Laravel is located in the laravel/ui Composer package, which may be installed using Composer:

```
composer require laravel/ui:^1.0 --dev
```
上記コマンドにより、`ui`というartisanコマンドが実行出来るようになります。  
`php artisan ui フレームワーク名`といった書式にて、vueやbootstrap、reactといったフロントエンドのスキャフォールディングをインストールすることが出来ます。また、`--auth`というオプションを付与することで、認証機能の実装も可能です。

それでは、実際に`ui`を実行してみましょう。
```
php artisan ui vue --auth
```
上記コマンドにて、ユーザー登録や認証関連のビューファイルを生成することが出来るようになります。    
下記のようなメッセージが表示されれば、インストール成功になります。
```
Vue scaffolding installed successfully.
Please run "npm install && npm run dev" to compile your fresh scaffolding.
Authentication scaffolding generated successfully.
```

続いて、`node.js`と`npm`のインストールを行います。  
フロントに必要なPackageがnode.jsとnpmになります。  
#### Vagrantfileとlaravel_test_appの位置関係について

今後、仮想環境の構築に当たって、各ファイルの位置関係が別ディレクトリに位置していない場合、予期しないエラーや、仮想環境外にファイルがあるとみなされ、環境破壊時にデータのバックアップ等ができなくなってしまう場合があります。Vagrantファイルと同じディレクトリ内にlarabel_test_appが位置しているか確認しましょう。

```
[vagrant@localhost vagrant]$ ls
Vagrantfile  laravel_test_app
```

もし、位置が異なる場合には、下記コマンドでディレクトリを変更下さい。
```
[vagrant@localhost vagrant]$ mv 変更元のディレクト 変更先のディレクトリ
```

ディレクトリの位置関係も問題ない設定になりましたら、下記コマンドを実行しましょう
```
[vagrant@localhost laravel_test_app]$ sudo yum install nodejs
```

実行後、バージョンが確認できればインストール完了です。
```
[vagrant@localhost laravel_test_app]$ node -v
[vagrant@localhost laravel_test_app]$ npm -v
```

以上にて、認証機能を利用するための環境構築が完了となります。

***
## ⑦Laravelプロジェクトの起動

WebサーバであるNginxを起動させLaravelのTodoアプリケーションを動かしていきます。
Nginxを使用してPHPアプリケーションを動かす場合、必ず`php-fpm`というPHPのモジュールが必要になります。
そのため、Nginxは`php-fpm`というモジュールとセットで機能することになります。
なお、Nginx自体はすでにこれまでの操作にてインストール済となります。

### Nginxの設定ファイルの編集

Nginxの設定ファイルを編集していきます。
使用しているOSがCentOSの場合、`/etc/nginx/conf.d`ディレクトリ下の`default.conf`ファイルが設定ファイルとなります。
```
$ sudo vi /etc/nginx/conf.d/default.conf
```
以下が編集が必要な内容となります。
コメントアウトを忘れずに行って下さい。
```
server {
  listen       80;
  server_name  192.168.33.19;
  root /vagrant/laravel_test_app/public; # 追記
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
  # 省略
```

Nginxの設定ファイルの変更は、以上です。  
次に php-fpm の設定ファイルを編集していきます。
```
$ sudo vi /etc/php-fpm.d/www.conf
```
変更箇所は以下になります。
なお、user名はデフォルトの値としてhogehogeとしています。  
適宜お読み替え下さい。
```
;24行目近辺
user = hogehoge
↓ 変更
user = nginx
group = hogehoge
↓ 変更
group = nginx
```
設定ファイルの変更に関しては、以上となります。  
では早速再起動しましょう。  
なお、そのまま再起動を行うと、変更後のファイルが適用されない場合があるため、今回は一度`vagrant　halt`により停止後、`vagrant up`にて起動するといった流れをとります。
```
[vagrant@localhost vagrant]$ exit
PC名:laravel_test_app arranzt$ vagrant halt
PC名:laravel_test_app arranzt$ vagrant up
PC名:laravel_test_app arranzt$ vagrant ssh
[vagrant@localhost ~]$ sudo systemctl restart nginx
[vagrant@localhost ~]$ sudo systemctl start php-fpm
```
再度ブラウザにて、`http://192.168.33.19`を入力して確認してください。

画面は表示されますが、以下のようなLaravelのエラーが表示される方とそうでない方がいるかと思います。
エラーが表示されなかった方は次の内容に進み、エラーが表示された方は以下の手順で操作下さい。
```
403 Forbidden
```
上記エラーは`SELinux（Security-Enhanced-Linux）`というカーネルの制御機能が働いている場合に発生する可能性があります。こちらのシステムにより、細かいパーミッション管理がされていることが原因です。

### SELinuxとは
システムにアクセス可能なユーザーをより詳細に制御できるようにする、Linuxシステム用のセキュリティ・アーキテクチャです。
システム上のアプリケーション、プロセス、ファイルのアクセス制御を定義するほか、アクセス制御にはセキュリティポリシーを使用します。セキュリティポリシーは一連のルールで構成されており、アクセス可能なものとそうでないものを SELinux が識別できるようにします。

まずはSELinuxが有効化されているかを確認しましょう
```
[vagrant@localhost ~]$ getenforce
Enforcing
```
※ SELinuxのステータスについて  
`Enforcing`：SELinuxが有効な状態  
`Permissive`:SELinuxは有効なものの、SELinuxポリシーが強制されず、オペレーションの実行が出来る状態  
`Disabled`：SELinuxが無効な状態  

それでは、今回は`Permissive`の状態で動かしてみましょう。
下記コマンドで`/etc/selinux/config`のファイルを編集しましょう。
```
省略

SELINUX=permissive

省略
```

設定が変更できたら、再起動を行います。
```
[vagrant@localhost ~]$ reboot
[vagrant@localhost ~]$ getenforce
Permissive
```
`getenforce`の実行結果が、`Permissive`になっていれば、`SELINUX`の設定が変更できたことになります。
ブラウザで`http://192.168.33.19`を入力してみましょう。
下記のようなエラーが表示されたでしょうか。

```
The stream or file "/vagrant/laravel_test_app/storage/logs/laravel-2020-10-11.log" could not be opened in append mode: failed to open stream: Permission denied
```

これは 先程php-fpmの設定ファイルの user と group を nginx に変更したと思いますが、ファイルとディレクトリの実行 user と group に nginx が許可されていないため起きているエラーです。

以下のコマンドを実行して nginx というユーザーでもログファイルへの書き込みができる権限を付与してあげましょう。
```
[vagrant@localhost ~]$ cd /vagrant/laravel_test_app
[vagrant@localhost laravel_test_app]$ sudo chmod -R 777 storage
```

※　`chmod -R`  
指定したディレクトリ以下のディレクトリ・ファイル全ての権限を一括変更できるオプションです。  

`777`  
8進数で表現したパーミッションです。rwxのパーミッションを付与します。  

では再度`http://192.168.33.19`にアクセスして正常にLaravelのWelcome画面の表示が出来るか確認下さい。

### DBの設定変更について

laravel_test_appディレクトリ下の .env ファイルの内容を以下に変更してください。
```
DB_DATABASE=laravel_lesson
↓
DB_DATABASE=laravel_test_app

DB_PASSWORD=
↓
DB_PASSWORD=登録した新しいパスワード
```
その後、laravel_test_appディレクトリに移動して `php artisan migrate` を実行します。
マイグレーションが問題なく実行できれば、DBの設定は完了となります。

### 新規登録とログインについて

それでは実際にユーザ登録からログインまで行っていきましょう。  
`http://192.168.33.19`にアクセスし、`Register`からユーザ情報を登録しましょう。

登録時、下記のようなエラーが出ることがあります。

```
file_put_contents(/vagrant/laravel_test_app/storage/framework/sessions/Av21BxSHxQCWTHtclWEwNkJ7BdorIL8oP3ucFvQU): failed to open stream: Permission denied
```

こちらもパーミッションの設定が原因となります。
下記コマンドを実行することで、権限の付与が完了します。
```
[vagrant@localhost laravel_test_app]$ sudo chmod -R 777 /vagrant/laravel_test_app/storage/framework/sessions/Av21BxSHxQCWTHtclWEwNkJ7BdorIL8oP3ucFvQU
```

それでは、登録完了後、`You are logged in!`と表示されていれば、全ての作業が完了となります。

環境構築所感  
・想定外のエラーが多発してしまい、構築に時間がかかってしまった。４回ほど作り直しを行い、やっとログインまで到ることが出来た。  
・CUIで構成されるLinux系の構成について学習が進んだ。  
・コーディングとは異なる難しさがあると感じた。特に、依存関係にあるパッケージのインストールでは、どのファイルがどんな意味合いをもつファイルであるかを理解できないままにインストールが進むので、エラーを出した時に原因が非常にわかりにくい。環境構築はプロでも難しいと言われるのを聞いたことがあるが、Linux系のエラーを吐いた時に内部の処理が見えてこないままだと解決までの道筋が追えず、苦しかった。  

参考書籍  
『コピペでOK!VMを作りながら学ぶ　Webサーバー構築の基礎for Mac』（team-aries編）  
『nginx実践ガイド』（著：渡辺高志）  

参考サイト  
『GizTech』http://giztech.gizumo-inc.work/categories  
『Qiita』https://qiita.com/yu01_y/items/70d4ad6c73a6b4cd4cfe  
https://qiita.com/Yorinton/items/c51ea54866b462943ffc  
『「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典』https://wa3.i-3-i.info/word11.html  
『技術評論社』https://gihyo.jp/admin/serial/01/ubuntu-recipe/0410  
『TECH Projin』https://tech.pjin.jp/blog/2017/01/12/  lpicyokuarushitsumonsyudai2kaikomandoyuraihensono3/  
『wikipedia』https://ja.wikipedia.org/wiki/Grep  
『nginx』http://nginx.org/en/linux_packages.html  
『phpマニュアル』https://www.php.net/downloads  
『fedora WIKI』https://fedoraproject.org/wiki/EPEL  
『Architect Note』http://blog.tojiru.net/article/440339824.html  
『Laravel LLC』https://laravel.com/docs/6.x/frontend#introduction  
『Red Hat』https://www.redhat.com/ja/topics/linux/what-is-selinux  