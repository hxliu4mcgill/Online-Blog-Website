# Projects001 个人在线博客

更新记录：

<table>
<tr>
<td>日期<br/></td><td>更新内容<br/></td></tr>
<tr>
<td>2024-11-04<br/></td><td>使用NGinx代替了Apache2，实现了LNMP<br/></td></tr>
</table>

# 背景

该项目基于 LAMP 和 LNMP 架构，旨在实现一个基础的网页功能，为之后的网页应用开发做准备。

LAMP 架构:

LAMP 架构是指一种常用的 Web 应用程序开发和部署架构，由四个主要组件组成，分别是 Linux 操作系统、Apache Web 服务器、MySQL 数据库以及 PHP 编程语言，它们的首字母缩写组成了 LAMP。

LNMP 架构:

LNMP 架构和上述 LAMP 架构的区别是讲 Apache 替代为了 NGinx。据说 NGinx 是现在比较主流的服务器框架，轻量化，效率高。

# 基于 LAMP 架构的网页

## 安装 Apache2

```bash
sudo apt-get install apache2 -y
```

安装好后，您可以通过访问实验室 IP 地址  [http://](http://119.29.200.201/)公网 IP 查看到 “it works” 界面，说明 apache2 安装成功。

## 安装 php 组件

```bash
sudo apt-get install php7.4 -y （这里的版本需要和服务器的镜像版本匹配）
sudo apt-get install libapache2-mod-php7.4


sudo apt install php php-mysql libapache2-mod-php
sudo apt install php php-mysql libapache2-mod-php php-xml php-gd php-curl
```

如果只有 apache，apache 不能解析 php 文件。

## 安装 MySQL

安装 MySQL 过程中，控制台会提示您输入 MySQL 的密码，您需要输入两次密码，并记住您输入的密码，后续步骤需要用到：

`sudo apt-get install mysql-server -y`

## 安装 php MySQL 相关组件：

> `sudo apt-get install php7.4-mysql`

## 安装 phpAdmin

使用 `apt-get` 安装 `phpmyadmin`，安装过程中，您需要根据提示选择 apache2 ，再输入 `root密码` 和 `数据库密码`：
`sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin`

## 重启 mysql 和 Apache

```bash
sudo service mysql restart
sudo systemctl restart apache2.service
```

## 为 wordpress 设置 MySQL 数据库

1. 登录到 MySQL：

```bash
sudo mysql -u root -p
```

1. 创建数据库和用户：

```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 下载 WordPress

下载并解压 WordPress：

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvf latest.tar.gz
```

## 配置 WordPress

将 WordPress 文件复制到 Web 服务器的根目录：

```bash
sudo cp -R wordpress/* /var/www/html/
```

然后设置文件权限：

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

## 配置 WordPress

复制 `wp-config-sample.php` 文件并重命名：

```bash
cd /var/www/html/
sudo cp wp-config-sample.php wp-config.php
```

编辑 `wp-config.php` 文件，设置数据库信息：

```bash
sudo nano wp-config.php
```

找到以下行并进行相应修改：

```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpressuser');
define('DB_PASSWORD', 'your_password');
```

1. 完成安装

在浏览器中打开你的服务器 IP 地址或域名，访问 WordPress 安装页面，例如 `http://your_server_ip/`。按照屏幕上的说明完成安装。

1. 安装完成

按照提示设置网站标题、用户名和密码，完成后即可登录 WordPress 后台。

# 基于 LNMP 架构的网页

核心步骤就是安装 Nginx 替代 apache。

## 配置 Nginx

创建一个新的 Nginx 配置文件：

`sudo nano /etc/nginx/sites-available/wordpress`

添加以下配置（根据你的服务器名称和路径进行调整）：

```bash
server {
    listen 80;
    server_name your_server_ip;  # 或你的域名
    root /var/www/html;
    index index.php index.html index.htm;
    location / {
        try_files 
$uri $
uri/ /index.php?$args;
    }
    location ~ \.php
$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;  # 根据你的 PHP 版本调整
        fastcgi_param SCRIPT_FILENAME $
document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht {
        deny all;
    }
}
```

## 启用配置

启用新的 Nginx 配置并测试配置是否正确：

`sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/ ``sudo nginx -t`

如果没有错误，重新启动 Nginx：

`sudo systemctl restart nginx`

遇到问题：

重启 nginx 失败：

发现 apache 已经占用了 80 端口，所以 nginx 冲突了。需要做的是关闭 apache

```bash
sudo systemctl stop apache2
```

502Bad Gateway：

发现没有安装 php-fpm

sudo apt -y install php-fpm

# 使用自己的网页

wordpress 尽管方便，但毕竟都是使用已经设置好的插件，不够灵活。于是想尝试使用 ai 写网页并上线到自己的服务器。
