##################################################################################################
#											编译安装											 #
##################################################################################################

一、安装必备工具
1.1、安装前检查是否已经安装
find / -name nginx

1.2、如果系统已经安装了nginx，那么就先卸载
yum remove nginx

1.3、安装必备工具
yum -y install gcc gcc-c++ autoconf automake install zlib zlib-devel openssl openssl-devel pcre-devel

二、编译安装Nginx 1.4.7
2.1、下载nginx源码包
wget -c http://nginx.org/download/nginx-1.4.7.tar.gz
tar -xzvf nginx-1.4.7.tar.gz
cd nginx-1.4.7	
./configure --prefix=/usr/local/nginx
make && make install

2.2、检测nginx配置文件是否正确
/usr/local/nginx/sbin/nginx -t

2.3、启动nginx服务
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s reload #重启命令

三、编译安装PHP 7.2.6
3.1、安装依赖
yum -y install libxml2 libxml2-devel

3.2、下载php源码包
wget -c http://php.net/get/php-7.2.6.tar.gz
tar -xzvf php-7.2.6.tar.gz
cd php-7.2.6
./configure  --prefix=/usr/local/php --enable-fpm --with-pdo-mysql --with-openssl --enable-mbstring
make && make install

3.3、创建配置文件，并将其复制到正确的位置
cp php.ini-development /usr/local/php/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp sapi/fpm/php-fpm /usr/local/bin

3.4、如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块，以避免遭受恶意脚本注入的攻击
vi /usr/local/php/php.ini
cgi.fix_pathinfo=0

3.5、修改 php-fpm.conf 配置文件，确保 php-fpm 模块使用 webuser 用户和 webuser 用户组的身份运行
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/default.conf
vim /usr/local/php/etc/php-fpm.conf
user = webuser
group = webuser

3.6、创建WEB用户及用户组：
/usr/sbin/groupadd webuser
/usr/sbin/useradd -g webuser webuser

3.7、启动 php-fpm 服务
/usr/local/bin/php-fpm

3.8、配置 Nginx 使其支持 PHP 应用
vim /usr/local/nginx/conf/nginx.conf
#修改默认的 location 块，使其支持 .php 文件
location / {
    root   html;
    index  index.php index.html index.htm;
}
#下一步配置来保证对于 .php 文件的请求将被传送到后端的 PHP-FPM 模块， 取消默认的 PHP 配置块的注释，并修改为下面的内容
location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
#报错：File not found
location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    /home/webuser/wwwroot$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}

3.9、重启 Nginx
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx -s reload

四、编译安装Mysql
4.1、安装依赖
sudo yum install gcc gcc-c++ pcre pcre-devel openssl openssl-devel
sudo yum install zlib zlib-devel cmake ncurses ncurses-devel bison bison-devel
#如下的几个依赖在CentOS7中需要安装,CentOS6不需要
sudo yum install perl perl-devel autoconf

4.2、安装boost
#如果安装的MySQL5.7及以上的版本，在编译安装之前需要安装boost,因为高版本mysql需要boots库的安装才可以正常运行。否则会报CMake Error at cmake/boost.cmake:81错误
#MySQL5.7.22要求boost的版本是1.59，更高版本的不适用MySQL5.7.22
cd /usr/local
wget http://www.sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
tar zxvf boost_1_59_0.tar.gz
mv boost_1_59_0 boost
#在预编译安装MySQL时要加上-DWITH_BOOST=/usr/local/boost

4.3、下载安装cmake 3.11.3
wget -c https://cmake.org/files/v3.11/cmake-3.11.3.tar.gz
tar -xzvf cmake-3.11.3.tar.gz
cd cmake-3.11.3
./bootstrap
gmake && gmake install

4.4、编译安装MySQL
# 添加MySQL用户
useradd -s /sbin/nologin -M mysql

# 下载MySQL
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.22.tar.gz

# 解压MySQL
tar zxvf mysql-5.7.22.tar.gz

# 进到MySQL目录
cd mysql

# 预编译
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DWITH_BOOST=/usr/local/boost \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/tmp/mysql.sock \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DWITH_INNODB_MEMCACHED=1 \
-DWITH_DEBUG=OFF \
-DWITH_ZLIB=bundled \
-DENABLED_LOCAL_INFILE=1 \
-DENABLED_PROFILING=ON \
-DMYSQL_MAINTAINER_MODE=OFF \
-DMYSQL_TCP_PORT=3306

# 编译&安装
make && make install

4.5、配置
# 创建软连接
# cd /usr/local
# ln -s mysql-5.7.22 mysql

# 添加到环境变量
vim /etc/profile
export PATH=/usr/local/mysql/bin:$PATH
export PATH=/usr/local/mysql/bin:/usr/local/mysql/lib:$PATH
source /etc/profile

cd /usr/local/mysql
mkdir -p /usr/local/mysql/{data,tmp,logs,pids}
chown mysql.mysql /usr/local/mysql/data
chown mysql.mysql /usr/local/mysql/tmp
chown mysql.mysql /usr/local/mysql/logs
chown mysql.mysql /usr/local/mysql/pids

# 修改/etc/my.cnf文件，编辑配置文件如下
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/tmp/mysql.sock

[mysqld_safe]
log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/pids/mysqld.pid

[client]
default-character-set=utf8mb4

# 创建mysqld.log 和 mysqld.pid文件
touch /usr/local/mysql/logs/mysqld.log
touch /usr/local/mysql/pids/mysqld.pid

chown mysql.mysql -R /usr/local/mysql/logs/
chown mysql.mysql -R /usr/local/mysql/pids/

# 加入守护进程
cd /usr/local/mysql
cp support-files/mysql.server /etc/init.d/mysqld
chmod a+x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on


# 初始化数据库， –initialize 表示默认生成一个安全的密码，–initialize-insecure 表示不生成密码
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

# 启动MySQL
service mysqld start

# 登录MySQL，修改密码
mysql -u root -p
set password for root@localhost = password('root');

# 修改MySQL数据存放目录

chown mysql.mysql /data/mysql
