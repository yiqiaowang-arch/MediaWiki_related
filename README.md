# 公网MediaWiki站点搭建全流程攻略·小白向（含MediaWiki和Parsoid）

**关键词**：MediaWiki-1.34.2、Ubuntu 18.04、腾讯云、Wiki、VisualEditor、Parsoid

**包含**：打开VisualEditor时，Parsoid 404的一个解决办法，详见Parsoid安装部分。

大家好，我是johnbriton。最近我在为学院的一个竞赛团队搭建一个团队wiki，用来储存并分享我们前期调研所收集到的各种信息。作为一种知识管理系统，wiki的内链做的相当方便，而且方便设置权限，便于多人协同工作，适合团队使用。

由于本人的专业与计算机完全不相关，所以在搭建站点的时候踩了很多坑，而在专业人士看来，这些“坑”甚至没有必要解释。经过一段时间的使用，我发现Wiki是一个很有效的知识分享工具，而这个工具不应当只存在于计算机“专业人士”的视野范围内。写这篇文章的目的，也是希望Wiki这种好的工具能够帮到更多需要知识管理的团队。

文章比较长，由于是面向零基础小白，所以基本算是从“开天辟地”讲起了；而门外汉写的文章，里面也肯定有很多知识性的错误和不恰当的操作。希望能帮到大家，也请各位大佬多指教！

## 正文目录
**00 服务器和域名选购**

**01 基于Ubuntu的基本环境配置**

  i 软件

  ii 基本服务的安装
  
  iii 基于Clash设置系统黛礼

**02 安装和配置MediaWiki**

i 配置Apache

   *网络与防火墙*
  
   *文件格式检索顺序*
  
ii 配置MySQL

iii 配置PHP（可选）[^4]

iiii 配置虚拟主机Virtual Host（推荐）

iiiii 安装MediaWiki

  *下载并解压安装包*
  
  *安装过程*
  
  *数据库配置*
  
  *扩展选择*
  
**03 Parsoid和VisualEditor的安装（MediaWiki 1.35以下）**

  VisualEditor安装

  Parsoid安装

  其他扩展的安装

  手机友好界面MobileFrontend

  短URL的设置

**参考链接**

  基本环境配置

  黛礼设置

  安装教程

  Parsoid 404问题
  
  *以下为正文*
    
## 00 服务器和域名选购

我选用了[腾讯云](cloud.tencent.com)作为我的服务器供应商。如果你是学生，就使用腾讯云的学生认证，可以给你提供一个单核2G内存的服务器，一年大概120块钱左右。

本篇文章主题是公网wiki站点的建设，所以为了方便访问，同时也需要购买一个域名。建议也在腾讯云买域名，这样方便把域名解析（连接）到服务器的ip地址上。域名的价格和它后缀的文字有关。一般而言，众所周知的.com就比其他的文字要贵一些，这也算是市场经济规律。我买的域名是.top，因为刚好赶上腾讯云的优惠，五年的域名使用权只需要120多。

另外，**强烈建议尽早购买域名！** 域名不是买来就能用的，而是需要经过实名认证和网信局备案的。这里有个坑：腾讯云的域名备案系统要求腾讯云账号注册48小时后才能备案，而登陆进去之后，系统会提示域名购买72小时之后才能备案。备案文件（主要是身份证照片和自拍）提交之后，腾讯云会打电话来确认信息，通过之后才能提交网信局备案。这个流程我到现在还没走完，所以我的wiki目前还只能用IP地址访问。

回到服务器的话题。当你购买服务器时，一般会有如下几个系统可选（截至2020.8.22）：

- Windows Server 2012 R2数据中心版 64位中文版
- Ubuntu Server 16.04.1 LTS 64位
- CentOS 7.2 64位

首先排除Windows，觉得Linux显得更厉害一些。其次，Linux的不同发行版（种类）之间，包管理器（下载服务时敲的代码）也稍有不同。Ubuntu据说是使用最为广泛的一种Linux操作系统（注：网上关于Debian的教程，Ubuntu也可以用），所以我就选了Ubuntu。尽管后来发现网上的教程更多还是针对CentOS，但Ubuntu里各种服务的下载还是相对更方便。

购买成功之后，你会获得一个初始密码，可以用于登录你的服务器。服务器不需要审核，可以随时使用。

## 01 基于Ubuntu的基本环境配置

### i 软件

强烈建议在你自己的电脑里下载WinSCP软件，输入[用户名]:[你的ip地址]后就可以登录，密码和服务器的密码一样。下载之后可以直接在系统之间传文件。

vncserver是一个远程桌面软件，安装费时费力，而且占用服务器内存相当严重，[教程在此](https://cloud.tencent.com/developer/article/1609578)。大多数操作尽量还是在命令行里解决，这样占用内存相当少。但如果涉及到文字编辑，图形界面还是很有必要的。还是建议使用腾讯云自带的VNC登录方式，不要自己搭vncserver。 

有些服务可能会出现无法安装，请自行研究镜像等解决方案。

### ii 基本服务的安装

首先介绍下命令行的基本使用方式。

```bash
cd /a/bb/.c #将当前所在位置切换为a文件夹下的bb文件夹下的.c文件夹。注意：.c文件夹是隐藏的。
cd .. #返回上级文件夹
ls #查看当前目录下的所有文件和文件夹，不包括隐藏文件夹
ls -a #查看当前目录下的所有文件和文件夹，包括隐藏文件夹
sudo #可以理解为“以管理员身份运行”。
su #切换为root管理员权限
su ubuntu #切换回普通用户权限，此处用户名为ubuntu
```

以上是基本的命令行操作方式。

```bash
sudo apt-get install apache2 mysql-server php php-mysql libapache2-mod-php php-xml php-mbstring
```

之后重启apache2[^1]。

```bash
sudo service apache2 reload
```

可选的软件包，为了以防万一还是全部安装上：

```bash
sudo apt-get install php-apcu php-intl imagemagick inkscape php-gd php-cli php-curl git
```

[^1]: 在Ubuntu或Debian中，这个服务被称为Apache。而在CentOS中，它被称为httpd，两个系统的操作并不完全相同。如果你使用的系统不是Ubuntu，此教程中的一些命令行代码可能不适合你。

### iii 基于Clash设置系统黛礼

在[此处](https://github.com/Dreamacro/clash/releases)选择clash-linux-amd64版本下载Linux版clash。

如果不熟悉Linux命令行，可以在本地解压后，将解压出来的clash-linux-amd64文件，用前述的WinSCP软件，拷贝至/home/ubuntu（你的账户名称，此处为ubuntu）/clash文件夹下面。之后定位至这个文件夹：

```bash
cd /home/ubuntu/clash
```

并修改该clash文件的权限，使得你可以执行它，并执行一次：

```bash
chmod +x clash-linux-amd64
./clash-linux-amd64
```

这时它将提示两个信息：

```
INFO[0000] Can't find config, create a initial config file
INFO[0000] Can't find MMDB, start download
```

因为你现在没有代理，所以这个mmdb文件无法下载，需要你[在这里](https://github.com/wp-statistics/GeoLite2-Country)手动下载。如果下来下来的是压缩文件，请先手动解压。

之后联系你的代理服务供应商，让他为你提供一个含有可用代理的config.yaml，里面应当包含和科学上网有关的各种服务器信息，以及一些代理策略。

打开你的`config.yaml`，检查代理策略里面**不走代理，直接连接**的策略（注意最后的`[DIRECT]`），一般是这样的内容（如果没有策略组，就手动加上）：

```yaml
proxy-groups: # 策略组，如果文件里已经有，就不需要重复添加
	- { name: '(设置的策略名称)', type: select, proxies: [DIRECT] } # 不走代理，直接连接
```

因为之后要涉及Ubuntu命令行中的下载，如果腾讯云的下载源走了代理，就会下不下来；但有些内容又必须要代理才能下载，所以需要手动设置代理策略，以绕开腾讯云的链接。

输入规则：

```yaml
rules: # 如果文件里已经有rules，就不需要重复添加
    - 'DOMAIN-KEYWORD,tencentyun,（设置的策略名称）' # 只要域名中含有tencentyun这个关键词，就不走代理，直接连接
```

之后保存，并和Country.mmdb一同通过WinSCP软件上传至服务器端的`/home/（用户名）/clash`文件夹下。先删除clash刚刚建立的，空的`config.yaml`文件，再将这两个文件都拷贝至`home/（用户名）/.config/clash`文件夹下：

```bash
rm -rf /home/（用户名）/.config/clash # 清空这个文件夹
cp /home/（用户名）/clash/config.yaml /home/（用户名）/.config/clash
cp /home/（用户名）/clash/Country.mmdb /home/（用户名）/.config/clash # 注意Country的大小写
```

如果你是在图形界面操作，那你很有可能看不到.config文件夹，这是因为它是个隐藏文件夹。即使是在命令行里，用ls命令也看不到，但这不代表文件夹不存在。你可以用-a去显示全部的文件夹：

```bash
ls -a home/（用户名）
```

之后再删去原来的那个文件夹里的Country.mmdb文件：

```bash
rm /home/（用户名）/clash/Country.mmdb
```

请注意，现在已经有了两个clash文件夹，分别是：

```bash
/home/（用户名）/clash
/home/（用户名）/.config/clash
```

**这两个文件夹里的config.yaml文件要保持完全一致。**

此时请使用腾讯云的VNC方式登录。输入账号密码之后，进入图形界面，在应用中找到terminal（终端），打开后定位至第一个clash文件夹，并运行clash

```bash
su （用户名） # 此命令用于退出root权限，以普通用户的身份运行clash，否则会找不到我们刚才设好的那几个文件
cd /home/（用户名）/clash
./clash-linux/amd64
```

此时clash应该就能正常运行了。这时我们**最小化（不要关掉）**这个terminal，打开浏览器，输入`http://clash.razord.top`，在浏览器里完成clash本身的设置。首次打开时可能会要求你输入ip和端口，请在你刚刚上传到服务器的`config.yaml`中找到这两行：

```yaml
external-controller: '127.0.0.1:xxxx'
secret: 'xxxxxx' # 没有就不填
```

然后在Settings里将策略改为Rule，这样才能个别地对网站进行代理，同时绕过腾讯云镜像。

随后在桌面左下角打开应用桌面，搜索settings，点击齿轮，在左边找到proxy标签栏。选择代理方式为manual（手动），在http代理和socks5代理中分别输入127.0.0.1和对应的端口。端口信息还是从config.yaml中获取，找到这两行：

```yaml
port: **** #这是http代理端口
socks-port: **** #这是socks5代理端口
```

以上设置完毕后，打开浏览器输入wikipedia.org，如果能正常访问，说明代理已经设置成功了。如果要关闭代理，首先打开刚才最小化的客户端，按ctrl+c关闭；其次打开设置，将刚才设置的manual改为disabled，**两者都需要关闭，否则无法上网。**如果想继续在terminal里操作，请新建一个terminal，并且不要关闭之前运行着clash的那个。关于订阅自动更新、clash应用图标设置等高级内容，请参阅[Clash For Linux 安装及使用](https://www.jianshu.com/p/2906066d2e0a)以及[clash for linux 安装及配置自动订阅](https://www.jianshu.com/p/1f711db26f34)。

需要注意的是，**图形界面设置的全局代理在终端中并不能直接使用，仍需手动配置环境变量，或修改命令的.conf文件（不推荐）。**可以用下面的方式设置终端内的临时代理[^2]。

```bash
export http_proxy=http://127.0.0.1:**** # **** = 刚才你设置的http代理端口
sudo apt-get update
```

这种方法能够对同一个terminal内的所有命令配置代理，但有时apt并不能成功。本教程中共有两个命令需要走以上设置好的代理，分别是：

```
add-apt-repository
apt
```

亲测有效的代理配置方式各不相同，请参见**基于Parsoid的扩展:VisualEditor**一节。

**当你配置完代理，并确认其可用之后，请立刻在运行代理的terminal窗口中按`Ctrl+C`终止代理程序，关闭terminal窗口，并在Setting中将代理模式设置为`disabled`。**

[^2]: 不推荐一直使用代理，因为这样可能会被腾讯云查封服务器。

## 02 安装和配置MediaWiki

### i 配置Apache

#### 网络与防火墙

如果你刚才（基于Ubuntu的基本环境配置-基本服务安装）已经成功安装了Apache，则你应该可以在防火墙的应用名单中看到Apache的身影：

```bash
sudo ufw app list
# 以下为系统输出的内容
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

 然后查看`Apache Full`的档案，你应当可以看到它允许`80`和`443`端口的数据：

```bash
sudo ufw app info "Apache Full"
# 以下为系统输出的内容
Profile: Apache Full
Title: Web Server (HTTP,HTTPS)
Description: Apache v2 is the next generation of the omnipresent Apache web server.

Ports:
  80,443/tcp
```

然后在防火墙中放行进入`Apache Full`的流量：

```bash
sudo ufw allow in "Apache Full"
sudo systemctl restart apache2
```

接下来，请试着找到服务器的公网IP。对于腾讯云服务器，其公网IP可以在控制台中找到。如果控制台中没有，可能是你的页面缩放问题，请将浏览器窗口最大化后刷新。

还有一种查看IP地址的方法：

```bash
sudo apt install curl
curl http://icanhazip.com
```

此方法显示出的IP地址即为电脑的公网IP。

另一个通用的方法是：

```bash
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

你将看到2-3个IP地址，但你的电脑只能使用其中的一个，所以请你依次尝试一下。

这时，如果你直接在浏览器里输入你的电脑IP，应当会出现这样的页面：<img src="https://assets.digitalocean.com/articles/how-to-install-lamp-ubuntu-18/small_apache_default_1804.png" alt="Apache2 Ubuntu的默认界面"  />

如果你成功看到了这个页面，说明你的Apache2已经正确配置了。如果看不到，首先去`/var/www/html`目录下面看看，是否有`index.html`这个文件。如果没有，说明你的apache没能成功安装。如果有，就请再检查一下防火墙的设置。

#### 文件格式检索顺序

刚才我们看到的页面是由html文件生成的静态网页，而MediaWiki是一种由PHP语言编写成的动态网页。Apache在读取网站的运行文件时，总会先找到一个叫做index的文件，再从index中读取它想要运行的其他内容。

因此，我们希望MediaWiki所使用的php文件能够先于html文件被找到并访问。为此，我们需要到Apache的文件夹中改变其配置文件中不同格式文件的读取顺序。输入：

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

之后你将看到这样的显示内容：

```html
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```

解释一下刚才的命令中vim的含义：vim是linux系统的一个文件编辑器，可以在命令行环境下使用。打开文件后，你会发现你无法修改，这时按`I`键进入`INCERT`模式后，便可以移动光标进行修改。

将内容中的index.php移动至最前方，仅次于`DirectoryIndex`的位置，如下所示：

```html
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

修改完成后，按下esc键退出`INCERT`模式，并输入`:wq`（半角冒号+wq）保存并退出。之后重启Apache2：

```bash
sudo systemctl restart apache2
```

你也可以用`systemctl`命令检查Apache2的状态：

```
sudo systemctl status apache2
```

如果Apache2运行正常，则会显示这样的内容：

```bash
● apache2.service - LSB: Apache2 web server
   Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Tue 2018-04-23 14:28:43 EDT; 45s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 13581 ExecStop=/etc/init.d/apache2 stop (code=exited, status=0/SUCCESS)
  Process: 13605 ExecStart=/etc/init.d/apache2 start (code=exited, status=0/SUCCESS)
    Tasks: 6 (limit: 512)
   CGroup: /system.slice/apache2.service
           ├─13623 /usr/sbin/apache2 -k start
           ├─13626 /usr/sbin/apache2 -k start
           ├─13627 /usr/sbin/apache2 -k start
           ├─13628 /usr/sbin/apache2 -k start
           ├─13629 /usr/sbin/apache2 -k start
           └─13630 /usr/sbin/apache2 -k start
```

### ii 配置MySQL

首先，创建一个新的MySQL用户（new_mysql_user）：

```bash
sudo mysql -u root -p
```

此时将进入mysql的命令行界面，并输入[^3]：

```mysql
mysql> CREATE USER 'new_mysql_user'@'localhost' IDENTIFIED BY '（在这里输入密码）';
mysql> quit;
```

然后创建一个新的mysql数据库my_wiki：

```bash
mysql -u root
```

```mysql
mysql> CREATE DATABASE my_wiki;
mysql> use my_wiki;
Database changed
```

然后授予你刚刚创建的mysql用户访问my_wiki的权限：

```mysql
mysql> GRANT ALL ON my_wiki.* TO 'new_mysql_user'@'localhost';
Query OK, 0 rows affected (0.01 sec)
mysql>quit;
```

[^3]:mysql命令行都以半角分号`;`结尾。

### iii 配置PHP（可选）[^4]

来自MediaWiki的官方教程：

> 编辑你的PHP配置文件，`php.ini`。在Ubuntu Trusty和Debian Jessie上，这个文件位于`/etc/php5/apache2/php.ini`。
>
> 在 Ubuntu Xenial and Debian Stretch (PHP 7)上, 它位于`/etc/php/7.0/apache2/php.ini`。
>
> 在Raspbian（bluster）上，它位于 `/etc/php/7.3/apache2/php.ini`。

本人认为这个路径和php版本有关，所以请打开`/etc/php`文件夹，确认你的php版本：

```bash
cd /etc/php
ls
```

本人电脑上的php版本为7.2。所以打开/7.2/apache2/php.ini：

```
sudo vim /7.2/apache2/php.ini
```

注意右下角的top，当你向下滚动光标的时候，它会提示你现在处在文件的哪个位置。你也可以按`/`键进入查找模式。

因为一些php脚本可能相当占用内存，所以向下滚动到约21%处，找到

```ini
memory_limit = 8M
```

将其修改为至少128M。

并向下滚动到约41%处，找到

```ini
upload_max_filesize = 2M
```

修改为至少20M，或根据你想上传的文件大小自行调整。

根据本人实测，即使修改了`upload_max_filesize`，在wiki的“上传页面”进行上传的时候也会显示“最大文件限制 = 8M”，这是因为：

```ini
post_max_size = 8M
```

请自行修改至合适大小。

[^4]:这些步骤是可选的，也可以在安装之后进行。 没有这些改变MediaWiki也可以运行。

### iiii 配置虚拟主机Virtual Host（推荐）

当你使用Apache2时，虚拟主机能使你用一个服务器同时支持多个网站，同时不会干扰到神秘的默认设置（本人自己的理解）。以下以**your-domain**作为网站名称的代指，实际应用中请将所有的your_domain更换为自己想要的网站名字。

首先，我们抛弃掉apache2默认界面所在的/var/www/html文件夹，另立门户：

```bash
sudo mkdir /var/www/your_domain
```

其次，修改文件夹的相应权限，使得你不用超级管理员也可以访问：

```bash
sudo chown -R $USER:$USER /var/www/your_domain
sudo chmod -R 755 /var/www/your_domain
```

然后在里面也创建一个测试用的index.html：

```bash
vim /var/www/your_domain/index.html
```

按`I`键在里面输入这些内容：

```html
<html>
    <head>
        <title>Welcome to Your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
```

然后按esc键，输入`:wq`保存并离开。

之后，为了让Apache2正确识别到你的内容，你需要专门为your_domain创建一个配置文件：

```bash
sudo vim /etc/apache2/sites-available/your_domain.conf
```

并在里面输入这些内容：

```html
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
# 在ServerName这里输入你的域名。
    ServerName www.your_domain.com
# 如果你想多个网站名称都取得相同的网站，可以加在ServerAlias后加上其他网站别名。建议把你的公网ip也放进去。
# 别名间以空格隔开。
    ServerAlias your_domain.com 154.xxx.xxx.xxx 
# 在ServerAdmin后加上网站管理员的电邮地址，方便别人有问题是可以联络网站管理员。
    ServerAdmin nide@youxiang.com
    DocumentRoot /var/www/your_domain
    <Directory /home/www/your_domain>
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

然后按esc键，输入`:wq`保存并离开。

之后让Apache2记住这个配置文件，并忽视默认的那个配置文件：

```bash
sudo a2ensite your_domain.conf
sudo a2dissite 000-default.conf
```

之后检查一下配置文件的格式是否正确：

```bash
sudo apache2ctl configtest
```

不出意外的话，应该会显示：

```
Syntax OK
```

然后重启apache2：

```bash
sudo systemctl restart apache2
```

然后在浏览器栏中输入`http://your-domain.com`或者你刚才设置的那些别名。不出意外的话，应该可以看到：![working](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9hc3NldHMuZGlnaXRhbG9jZWFuLmNvbS9hcnRpY2xlcy9hcGFjaGVfdmlydHVhbF9ob3N0c191YnVudHUvdmhvc3RfeW91cl9kb21haW4ucG5n?x-oss-process=image/format,png)

这样，你的虚拟主机就设置完成了。

在进行下一步操作之前，我们需要先测试你的apache是否支持MediaWiki所使用的php文件。

创建一个`info.php`：

```bash
sudo vim /var/www/your_domain/info.php
```

并输入以下内容：

```php
<?php
phpinfo();
?>
```

然后保存并退出。

然后在浏览器栏中输入`http://your_domain.com/info.php`。如果不出意外的话，你可以看到php的信息页面：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828005810771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvaG5icml0b24=,size_16,color_FFFFFF,t_70#pic_center)


如果成功打开，说明你的PHP已经能成功启动了。此时就可以删掉这个临时的php文件，以免和你的其他文件起冲突：

```bash
sudo rm /var/www/your_domain/info.php
```

### iiiii 安装MediaWiki

#### 下载并解压安装包

官方压缩包下载地址：`https://www.mediawiki.org/wiki/Download`。有以下两种方式可选，推荐使用第一种。

- 本地下载后上传至服务器端，需要本地安装WinSCP软件。

  - 下载并解压之后，将mediawiki-*文件夹内部的全部文件[^5]，上传至/var/www/your_domain文件夹中。

- 在服务器端下载

  - 配置wget黛礼：

    - 向服务器中设置以下环境变量：

    - ```bash
      export http_proxy=http://127.0.0.1:8087
      ```

    - 将压缩包下载到/tmp文件夹中

    - ```bash
      cd /tmp/
      wget https://releases.wikimedia.org/mediawiki/1.34/mediawiki-1.34.2.tar.gz
      ```

  - 然后解压至你的虚拟主机目录中[^5]

    - ```bash
      tar -xvzf /tmp/mediawiki-*.tar.gz
      sudo cp -r mediawiki-*./ /var/www/your_domain
      ```

    [^5]: *表示你当前的版本号

在正是安装之前，你需要输入以下代码，确定所需要的PHP扩展已经打开：

```bash
sudo phpenmod mbstring
sudo phpenmod xml
sudo systemctl restart apache2.service
```

然后打开浏览器，输入`http://localhost/index.php`（根据安装条件的不同，也可以输入`http://154.xxx.xxx.xxx`或者`http://www.your_domain.com`，并看情况在结尾加入`/index.php`）安装MediaWiki。

#### 安装过程

安装界面比较友好，在此不再赘述全部流程，只提醒几个需要注意的点：

##### 数据库配置

需要注意的是，在数据库配置的部分，需要你回忆起之前设置的`new_mysql_user`账户和它的密码。

同时，你必须打开“使用超级用户账户”选项。超级用户的名称为`debian-sys-maint`，密码应为空（如果你没有设置过的话）。

##### 扩展选择

MediaWiki预置了一些扩展[^6]，建议安装以下几种：[^7]

1. CategoryTree，动态导航分类结构。
2. Cite，参考引用的必备插件。
3. Gadgets，MediaWiki的小工具。
4. MultimediaViewer，媒体阅览器。
5. Nuke，让管理员批量删除页面。
6. ParserFunctions，用来加载维基百科的页面模板。
7. PdfHandler，在图像模式中查看PDF文件。
8. ReplaceText，让管理员执行全局字符查找和替换。
9. TitleBlacklist，允许管理员通过黑名单和白名单禁止页面和用户帐户的创建。
10. WikiEditor，wiki的经典编辑器，需要使用Wiki语法才能进行编辑。
11. VisualEditor（MediaWiki 1.35版本以上）：可视化编辑神器。

另外，还有一个非常有必要安装，但安装过程非常麻烦的插件：VisualEditor，能使用户像编辑word一样编辑wiki页面，大大降低编辑页面的成本，显著提升用户的体验和参与感。

[^6]:即使现在不安装这些扩展，之后也可以在官网另行下载安装。
[^7]:https://zhuanlan.zhihu.com/p/64283626。关于扩展的更多内容，可以参见这篇文章。

至此，MediaWiki应该已经全部安装完成了。当你结束安装时，浏览器将下载一个名为`LocalSettings.php`的文件。这个文件必须被移动至放置你所有MediaWiki文件的文件夹中，即`/var/www/your_domain`。一般来说，默认下载路径为`/downloads`，所以你可以使用这样的命令：

```bash
sudo mv /Downloads/LocalSettings.php /var/www/your_domain
```

然后将浏览器导航到你的公网ip（或域名），以查看你的新Wiki。

更多高级选项，请参见[手册:在Debian或Ubuntu上运行MediaWiki](https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu/zh)的最末尾，包括文件上传、更改Logo、更多MediaWiki扩展和URL美化。

## 03 Parsoid和VisualEditor的安装（MediaWiki 1.35以下）

如上所述，VisualEditor是一个非常好用的图形化编辑器，极大降低了Wiki的学习成本，强烈建议安装。其基本的工作原理为：利用Parsoid服务，将VisualEditor中带有排版的富文本，转换为wiki可以识别的Wiki语法。因此，**要想安装并使用VisualEditor，必须同时安装Parsoid。**需要注意的是，Parsoid服务在MediaWiki 1.35版本及以上默认存在。本教程基于MediaWiki 1.34.2版本，因此需要手动安装Parsoid。

### VisualEditor安装

下载链接：`https://www.mediawiki.org/wiki/Special:ExtensionDistributor/VisualEditor`

下载并解压之后，将整个VisualEditor文件夹拷贝至`/var/www/your_domain/extensions`目录下。

之后，打开`/var/www/your_domain`的`LocalSettings.php`，并在最下方输入：

```php
wfLoadExtension( 'VisualEditor' );
// Enable by default for everybody
$wgDefaultUserOptions['visualeditor-enable'] = 1;

// Optional: Set VisualEditor as the default for anonymous users
// otherwise they will have to switch to VE
$wgDefaultUserOptions['visualeditor-editor'] = "visualeditor";

// Don't allow users to disable it
$wgHiddenPrefs[] = 'visualeditor-enable';

// OPTIONAL: Enable VisualEditor's experimental code features
$wgDefaultUserOptions['visualeditor-enable-experimental'] = 1;

$wgVirtualRestConfig['modules']['parsoid'] = [
        // URL to the Parsoid instance - use port 8142 if you use the Debian package - the parameter 'URL' was first used but is now deprecated (string)
    	//在url中输入你的ip地址:8142，8142为Ubuntu系统中Parsoid的端口。
 'url' => 'xxx.xxx.xxx.xxx:8142',
 // Parsoid "domain" (string, optional) - MediaWiki >= 1.26
	//在domain中输入你的ip地址。
 'domain' => '154.8.203.228',
        //  // Parsoid "prefix" (string, optional) - deprecated since MediaWiki 1.26, use 'domain'
    //在prefix中输入你的ip地址。
 'prefix' => '154.8.203.228',
        //  // Forward cookies in the case of private wikis (string or false, optional)
 'forwardCookies' => false,
        //  // request timeout in seconds (integer or null, optional)
 'timeout' => null,
        //  // Parsoid HTTP proxy (string or null, optional)
 'HTTPProxy' => null,
        //  // whether to parse URL as if they were meant for RESTBase (boolean or null, optional)
 'restbaseCompat' => null,
 ];
```

其他高级操作（如安装RESTBase）请参考[扩展:VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor)。

### Parsoid安装

再次声明：MediaWiki 1.35版本以上不需要手动安装Parsoid。本文基于MediaWiki 1.34.2。

导入repository gpg key（资料库密钥）：

```bash
sudo apt install dirmngr
sudo apt-key advanced --keyserver keys.gnupg.net --recv-keys AF380A3036A03444
```

如果第二条命令无法工作（`gpgkeys: key AF380A3036A03444 canot be retrieved`），则尝试下面这个：

```bash
sudo apt-key advanced --keyserver pgp.mit.edu --recv-keys AF380A3036A03444
```

或者这个：

```bash
sudo apt-key advanced --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys AF380A3036A03444
```

然后添加Wikimedia资料库。此时需要用到刚才配置好了的黛礼。

首先运行clash：

```bash
cd /home/ubuntu/clash
./clash-linux-amd64
```

然后打开浏览器栏，输入`http://clash.razord.top`，确认黛礼方式为“rules”，并且黛礼规则中`tencentyun`关键词为`[DIRECT]`。

回到终端，输入：

```bash
export http_proxy=http://127.0.0.1:**** # **** = 刚才你设置的http黛礼端口
```

开启黛礼后不要关闭终端，继续输入：

```bash
sudo -E apt-add-repository "deb https://releases.wikimedia.org/debian jessie-mediawiki main"
```

其中，`-E`能够让`apt-add-repository`记住你刚才设置的黛礼。

然后执行安装：

```bash
sudo apt install apt-transport-https
sudo apt update
```

apt服务也需要黛礼，而它的黛礼方式与`apt-add-repository`有所不同。你需要在同一行代码内单独设置一次黛礼：

```bash
sudo apt -o "Acquire::http_proxy=http://127.0.0.1:****" install parsoid # **** = 刚才你设置的http黛礼端口
```

安装成功后，你需要打开`/etc/mediawiki/parsoid/config.yaml`，并滚动至约50%的位置，配置Parsoid：

```bash
sudo vim /etc/mediawiki/parsoid/config.yaml
```

并在`uri`的引号内填写mediawiki文件中`api.php`的路径：

```yaml
uri: 'http://xxx.xxx.xxx.xxx/api.php' # 在前面填写ip
```

然后在`domain`的引号内填写你的IP地址：

```yaml
domain: 'xxx.xxx.xxx.xxx'  # optional
#填写ip
```

**注意！这里domain的内容一定要和前面VisualEditor中domain保持一致！**

**注意！请滚动至文件末尾，将`#strictSSL: true`前面的`#`删掉，并将值改为改为`false`。**

之后重启Parsoid服务，并查看其状态：

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

打开浏览器，输入`http://xxx.xxx.xxx.xxx:8142`。如果Parsoid安装成功，你应该可以看到这样两句话：

> ### Welcome to the [Parsoid](https://www.mediawiki.org/wiki/Parsoid) web service.
>
> See [the API documentation on mediawiki.org](https://www.mediawiki.org/wiki/Parsoid/API).

这时打开你的wiki，并尝试“编辑”功能。不出意外的话，你应该可以正常使用VisualEditor了。

## 其他扩展的安装

### 手机友好界面MobileFrontend

鉴于上面已经介绍过解压和移动的操作，在这里不再重复。

[下载](https://www.mediawiki.org/wiki/Special:ExtensionDistributor/MobileFrontend)并将解压后的MobileFrontend文件夹放置于mediawiki文件夹下的extensions文件夹中。

MinervaNeue是Mediawiki专门为移动设备设计的特殊页面主题。在mediawiki文件夹下的skin文件夹中添加MinervaNeue文件夹（注意大小写），然后[下载压缩包](https://www.mediawiki.org/wiki/Special:SkinDistributor/MinervaNeue)并将解压后的所有文件拷贝于其中。

回到LocalSettings.php，在最末尾添加：

```php
wfLoadExtension( 'MobileFrontend' ); # 启用MobileFrontend扩展
wfLoadSkin( 'MinervaNeue' ); # 启用MinervaNeue主题
$wgMFDefaultSkinClass = "SkinMinerva"; # 设置移动设备的默认主题为MinervaNeue
```

安装即告完成。

### 短URL的设置

此前我们在使用wiki的时候，浏览器地址栏里总会显示`index.php`，看起来很长，而且不太美观：![设置之前的浏览器栏，URL特别长](https://img-blog.csdnimg.cn/20200829035221977.png#pic_center)
MediaWiki对此亦有设置的方法，可以美化URL的显示，隐藏后面一长串的内容，在[此处](https://www.mediawiki.org/wiki/Manual:Short_URL/Apache)查看具体的教程。教程中提供了一个[网站](https://shorturls.redwerks.org/)，能够根据你现有的URL和你想要设置的URL，自动生成配置文件的内容。根据网站的提示，尽量在域名的后面加一个`/wiki`，而不是直接使用域名，如：`wikipedia.org/wiki/blablabla`，而不是`wikipedia.org/blablabla`。

需要注意的是，尽量不要使用`.htaccess`的方法。请在此处点选`have root access`（因此以下操作请用`sudo`或者root权限进行）：![](https://img-blog.csdnimg.cn/20200829035316680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvaG5icml0b24=,size_16,color_FFFFFF,t_70#pic_center)
或者请在填写你现有的URL之后，待页面更新后，下拉至网页底端。展开Detected Settings，选择`I have root access on my server, plese output apache config.`，如图：![](https://img-blog.csdnimg.cn/20200829035340535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvaG5icml0b24=,size_16,color_FFFFFF,t_70#pic_center)
然后将`Apache Config`框内的内容，复制到`/etc/apache2/sites-enabled/your_domain.conf`里。注意，这些内容一定要放在最后一行`</VirtualHost>`的前面。![](https://img-blog.csdnimg.cn/20200829035405782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvaG5icml0b24=,size_16,color_FFFFFF,t_70#pic_center)
然后在`/var/www/your_domain/LocalSettings.php`里找到相应的字段，添加网页中所说的命令，注意不能简单“替换”：![](https://img-blog.csdnimg.cn/20200829035423535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvaG5icml0b24=,size_16,color_FFFFFF,t_70#pic_center)
最后不要忘记重启Apache2服务：

```bash
systemctl restart apache2
```


## 参考链接

### 基本环境配置

[手册:在Debian或Ubuntu上运行MediaWiki - MediaWiki](https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu/zh)关于安装服务和MySQL配置的部分（不包括Apache配置的部分，因为这个教程面向的是希望在本地使用wiki的读者）

[How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 18.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04)关于Apache Virtual Host配置的部分（不包括your_domain.yaml文件内容的部分，因为servername的设置不对）

[ubuntu apache2配置详解(含虚拟主机配置方法)](https://www.cnblogs.com/ylan2009/archive/2012/02/25/2368028.html)中关于配置sites-available里your_domain.yaml文件内容中`ServerName`，`ServerAlias`和`<Directory>`的部分。

### 黛礼设置：

[add-apt-repository设置黛礼]([https://tyyzqmf.github.io/2018/10/24/add-apt-repository%E8%AE%BE%E7%BD%AE%E4%BB%A3%E7%90%86/](https://tyyzqmf.github.io/2018/10/24/add-apt-repository设置黛礼/))

[Debian管理员手册](https://www.debian.org/doc/manuals/debian-handbook/sect.apt-get.zh-cn.html)中关于`apt -o "Acquire::""`的用法

[Ubuntu的apt-get黛礼设置](https://blog.csdn.net/lonelysky/article/details/81059339)中`-o`+命令行临时带入黛礼的用法，注意`apt-get`和`apt`不是一个语法。

### 安装教程

[手册:在Debian或Ubuntu上运行MediaWiki](https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu/zh)MediaWiki安装教程

[Extension:VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor)VisualEditor安装教程

[Parsoid](https://www.mediawiki.org/wiki/Parsoid)Parsoid安装教程

### Parsoid 404问题

[MediaWiki 安装 VisualEditor，编辑器和 parsoid 无法连接求助](https://www.v2ex.com/t/453206)最后关于404的问题，即将ParsoidConfig.js（老版本的parsoid配置文件）的ParsoidConfig.prototype.strictAcceptCheck设置成false，原帖见：[httpResponse 406 from Parsoid](https://www.mediawiki.org/wiki/Topic:Ua42lnptxq4056ki)

