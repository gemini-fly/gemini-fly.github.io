---
layout: post
title: 安装php7.3.5
categories: linux
tags: centos7 php
author: Dong ops
abbrlink: 22157
date: 2021-05-08 11:16:54
---

# 前言
工作需要使用php7.3.5做报表系统
本文采用编译安装
<!--more-->
1、先安装依赖包
```
yum -y install epel-release libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel ncurses curl gdbm-devel db4-devel libXpm-devel libX11-devel gd-devel gmp-devel expat-devel xmlrpc-c xmlrpc-c-devel libicu-devel libmcrypt-devel libmemcached-devel libzip gcc-c++ sqlite-devel  oniguruma oniguruma-devel autoconf
```

2、开始编译
```
./configure \
--prefix=/usr/local/php \
--exec-prefix=/usr/local/php \
--bindir=/usr/local/php/bin \
--sbindir=/usr/local/php/sbin \
--includedir=/usr/local/php/include \
--libdir=/usr/local/php/lib/php \
--mandir=/usr/local/php/php/man \
--with-config-file-path=/usr/local/php/etc \
--with-mhash \
--with-openssl \
--with-mysqli=shared,mysqlnd \
--with-pdo-mysql=shared,mysqlnd \
--with-gd \
--with-iconv \
--with-zlib \
--enable-zip \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-xml \
--enable-redis\
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-pcntl \
--enable-calendar \
--enable-exif \
--enable-sockets \
--with-xmlrpc \
--with-libxml-dir \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \
--with-curl \
--with-jpeg-dir \
--with-png-dir \
--with-freetype-dir \
--with-bz2 \
--enable-opcache \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--without-gdbm \
--enable-fast-install \
--disable-fileinfo

make && make install 
```
添加配置文件
```
cd /usr/local/php/etc
cp php-fpm.conf.default php-fpm.conf
cd /usr/local/php/etc/php-fpm.d
cp www.conf.default www.conf
```
查找当前php使用ini的位置
```
php --ini
```
添加配置文件
```
mkdir /usr/local/php/conf
vim php.ini
[PHP]
engine = On
short_open_tag = Off
precision = 14
output_buffering = 4096
zlib.output_compression = Off
implicit_flush = Off
unserialize_callback_func =
serialize_precision = -1
disable_functions =
disable_classes =
zend.enable_gc = On
zend.exception_ignore_args = Off
expose_php = On
max_execution_time = 30
max_input_time = 60
memory_limit = 128M
error_reporting = E_ALL
display_errors = On
display_startup_errors = On
log_errors = On
log_errors_max_len = 1024
ignore_repeated_errors = Off
ignore_repeated_source = Off
report_memleaks = On
variables_order = "GPCS"
request_order = "GP"
register_argc_argv = Off
auto_globals_jit = On
post_max_size = 8M
auto_prepend_file =
auto_append_file =
default_mimetype = "text/html"
default_charset = "UTF-8"
doc_root =
user_dir =
enable_dl = Off
file_uploads = On
upload_max_filesize = 2M
max_file_uploads = 20
allow_url_fopen = On
allow_url_include = Off
default_socket_timeout = 60
extension=ldap
[CLI Server]
cli_server.color = On
[Date]
[filter]
[iconv]
[imap]
[intl]
[sqlite3]
[Pcre]
[Pdo]
[Pdo_mysql]
pdo_mysql.default_socket=
[Phar]
[mail function]
SMTP = localhost
smtp_port = 25
mail.add_x_header = Off
[ODBC]
odbc.allow_persistent = On
odbc.check_persistent = On
odbc.max_persistent = -1
odbc.max_links = -1
odbc.defaultlrl = 4096
odbc.defaultbinmode = 1
[MySQLi]
mysqli.max_persistent = -1
mysqli.allow_persistent = On
mysqli.max_links = -1
mysqli.default_port = 3306
mysqli.default_socket =
mysqli.default_host =
mysqli.default_user =
mysqli.default_pw =
mysqli.reconnect = Off
[mysqlnd]
mysqlnd.collect_statistics = On
mysqlnd.collect_memory_statistics = On
[OCI8]
[PostgreSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
[bcmath]
bcmath.scale = 0
[browscap]
[Session]
session.save_handler = files
session.use_strict_mode = 0
session.use_cookies = 1
session.use_only_cookies = 1
session.name = PHPSESSID
session.auto_start = 0
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_domain =
session.cookie_httponly =
session.cookie_samesite =
session.serialize_handler = php
session.gc_probability = 1
session.gc_divisor = 1000
session.gc_maxlifetime = 1440
session.referer_check =
session.cache_limiter = nocache
session.cache_expire = 180
session.use_trans_sid = 0
session.sid_length = 26
session.trans_sid_tags = "a=href,area=href,frame=src,form="
session.sid_bits_per_character = 5
[Assertion]
zend.assertions = 1
[COM]
[mbstring]
[gd]
[exif]
[Tidy]
tidy.clean_output = Off
[soap]
soap.wsdl_cache_enabled=1
soap.wsdl_cache_dir="/tmp"
soap.wsdl_cache_ttl=86400
soap.wsdl_cache_limit = 5
[sysvshm]
[ldap]
ldap.max_links = -1
[dba]
[opcache]
[curl]
[openssl]
[ffi]
```
安装php-redis
```
tar -zxvf owlient-phpredis-2.1.1-1-g90ecd17.tar.gz
cd owlient-phpredis-90ecd17
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
cd /usr/local/php//lib/php/extensions/no-debug-non-zts-20060613/
vim php.ini
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/"
extension = redis.so
```
安装php-grpc
```
wget http://pecl.php.net/get/grpc-1.20.0.tgz
tar -xzvf grpc-1.20.0.tgz
cd grpc-1.20.0
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config 
make && make install
vim php.ini
extension = redis.so
```
