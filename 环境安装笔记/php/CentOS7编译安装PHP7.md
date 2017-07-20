# CentOS7编译安装PHP7

[TOC]

## 前置操作

### 移除旧版本PHP

先本地查找已安装的php版本

```shell
yum list installed | grep php
```

因为之前本地装了php70w，所以先把它卸载干净

```shell
yum remove php70w php70w-bcmath php70w-cli php70w-common php70w-fpm php70w-gd php70w-mbstring php70w-mcrypt php70w-mysql php70w-pdo php70w-xml
```

## 编译安装PHP

### 下载解压

php版本下载链接：http://php.net/downloads.php

php7.1.7版本：http://php.net/distributions/php-7.1.7.tar.gz

进入目标目录（这里是/home/source）：

```shell
wget http://php.net/distributions/php-7.1.7.tar.gz # 下载
tar zxf php-7.1.7.tar.gz # 解压
```

###安装依赖项

```shell
yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```

进入源码目录，配置编译环境

```shell
./configure \
--prefix=/usr/local/php7 \
--with-config-file-path=/etc/php7 \
--enable-fpm \
--with-fpm-user=nginx  \
--with-fpm-group=nginx \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared  \
--enable-soap \
--with-libxml-dir \
--with-xmlrpc \
--with-openssl \
--with-mcrypt \
--with-mhash \
--with-pcre-regex \
--with-sqlite3 \
--with-zlib \
--enable-bcmath \
--with-iconv \
--with-bz2 \
--enable-calendar \
--with-curl \
--with-cdb \
--enable-dom \
--enable-exif \
--enable-fileinfo \
--enable-filter \
--with-pcre-dir \
--enable-ftp \
--with-gd \
--with-openssl-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir  \
--with-freetype-dir \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--with-gettext \
--with-gmp \
--with-mhash \
--enable-json \
--enable-mbstring \
--enable-mbregex \
--enable-mbregex-backtrack \
--with-libmbfl \
--with-onig \
--enable-pdo \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-zlib-dir \
--with-pdo-sqlite \
--with-readline \
--enable-session \
--enable-shmop \
--enable-simplexml \
--enable-sockets  \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--with-libxml-dir \
--with-xsl \
--enable-zip \
--enable-mysqlnd-compression-support \
--with-pear \
--enable-opcache
```

编译、安装

```shell
make && make install
```

安装完成后

```shell
Installing shared extensions:     /usr/local/php7/lib/php/extensions/no-debug-non-zts-20160303/
Installing PHP CLI binary:        /usr/local/php7/bin/
Installing PHP CLI man page:      /usr/local/php7/php/man/man1/
Installing PHP FPM binary:        /usr/local/php7/sbin/
Installing PHP FPM defconfig:     /usr/local/php7/etc/
Installing PHP FPM man page:      /usr/local/php7/php/man/man8/
Installing PHP FPM status page:   /usr/local/php7/php/php/fpm/
Installing phpdbg binary:         /usr/local/php7/bin/
Installing phpdbg man page:       /usr/local/php7/php/man/man1/
Installing PHP CGI binary:        /usr/local/php7/bin/
Installing PHP CGI man page:      /usr/local/php7/php/man/man1/
Installing build environment:     /usr/local/php7/lib/php/build/
Installing header files:          /usr/local/php7/include/php/
Installing helper programs:       /usr/local/php7/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php7/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php7/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.3
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.4.2
[PEAR] PEAR           - installed: 1.10.5
Wrote PEAR system config file at: /usr/local/php7/etc/pear.conf
You may want to add: /usr/local/php7/lib/php to your php.ini include_path
/root/source/php-7.1.7/build/shtool install -c ext/phar/phar.phar /usr/local/php7/bin
ln -s -f phar.phar /usr/local/php7/bin/phar
Installing PDO headers:           /usr/local/php7/include/php/ext/pdo/
```

## 配置环境变量

打开/etc/profile文件

```shell
vim /etc/profile
```

然后在/etc/profile文件后面添加

```shell
export PATH=/usr/local/php7/bin:$PATH # 以“:”分隔
```

然后保存退出，并使修改生效。

```shell
source /etc/profile
```

## 移动配置文件

```shell
cp php.ini-production /etc/php7/php.ini # php配置文件
cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf # php-fpm配置文件
cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf # 连接配置文件
cp sapi/fpm/php-fpm /usr/local/php7/bin/ # 复制php-fpm到可执行目录ls
```

操作到此，可以使用php-fpm命令来测试是否成功运行

```shell
php-fpm
lsof -i :9000
```

## php-fpm.service配置

创建`/lib/systemd/system/php-fpm.service`

```ini
[Unit]
Description=php-fpm 7
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/php7/var/run/php-fpm.pid
ExecStart=/usr/local/php7/sbin/php-fpm
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

然后启动

```shell
systemctl start php-fpm.service
lsof -i :9000
```

然后加入开机启动

```shell
systemctl enable php-fpm.service
```

