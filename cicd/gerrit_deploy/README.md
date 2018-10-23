# gerrit 部署记录

>Gerrit 是一个基于 web 的代码评审工具, 它基于 git 版本控制系统。Gerrit 旨在提供一个轻量级框架, 用于在代码入库之前对每个提交进行审阅。‎‎更改将上载到 Gerrit, 但实际上并不成为项目的一部分, 直到它们被审阅和接受。它是标准开源过程的一个简单工具来支持提交补丁程序, 然后由项目成员在应用到代码库之前进行评审。‎Gerrit 首先是一个临时区域, 在提交的代码成为代码库的一部分之前, 可以对其修改进行检查。

## 工作原理
![](images/intro-quick-central-repo.png)

这是使用gerrit之前的架构，Gerrit取代了这个中央存储库并添加了一个额外的概念：一个待定更改存储。

![](images/intro-quick-central-gerrit.png)
使用Gerrit，当开发人员进行更改时，会将其发送到此待处理更改的商店，其他开发人员可以查看，讨论和批准更改。经过足够的审核人员批准后，更改将成为代码库的官方部分。

## 环境准备
- MySQL数据库-使用已有
```
mysql>create database gerritdb CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql>grant all on gerritdb.* to 'gerrituser'@'%' identified by 'gerritpass';
```
- 新建gerrit用户
```
useradd gerrit
passwd gerrit
```
- 配置Java环境
```
tar zxf jdk-8u181-linux-x64.tar.gz -C /usr/local/

vim /etc/profile

export JAVA_HOME=/usr/local/jdk1.8.0_181
export JRE_HOME=/usr/local/jdk1.8.0_181/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

source /etc/profile
```
- 安装git
`
 yum -y install git
`
- 下载gerrit
  http://gerrit-releases.storage.googleapis.com/index.html
## 安装部署
```
su - gerrit  #切换用户
mkdir gerrit_site
java jar gerrit-2.15.5.war init -d ~/gerrit_site   #安装
```
安装过程可能会遇到数据库表无法创建，可用管理员用户登录数据库执行`set global explicit_defaults_for_timestamp=1;`操作，如果还不行，可手动执行sql创建。

报错：
```
Exception in thread "main" com.google.gwtorm.server.OrmException: Cannot apply SQL
CREATE TABLE account_group_members_audit (
added_by INT DEFAULT 0 NOT NULL,
removed_by INT,
removed_on TIMESTAMP NULL DEFAULT NULL,
account_id INT DEFAULT 0 NOT NULL,
group_id INT DEFAULT 0 NOT NULL,
added_on TIMESTAMP NOT NULL
,PRIMARY KEY(account_id,group_id,added_on)
)
    at com.google.gwtorm.jdbc.JdbcExecutor.execute(JdbcExecutor.java:44)
    at com.google.gwtorm.jdbc.JdbcSchema.createRelations(JdbcSchema.java:134)
    at com.google.gwtorm.jdbc.JdbcSchema.updateSchema(JdbcSchema.java:104)
    at com.google.gerrit.server.schema.SchemaCreator.create(SchemaCreator.java:81)
    at com.google.gerrit.server.schema.SchemaUpdater.update(SchemaUpdater.java:108)

```
登录MySQL操作：
``` 
$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 76
Server version: 5.7.20-0ubuntu0.16.04.1 (Ubuntu)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set global explicit_defaults_for_timestamp=1;
Query OK, 0 rows affected (0.00 sec)

mysql> exit;
Bye

```
- 配置Nginx代理
```
server {
        listen       10083;
        location / {
          auth_basic              "Gerrit Code Review";

          auth_basic_user_file    /etc/nginx/conf.d/passwords; #用户认证文件路径 
          proxy_set_header        X-Forwarded-For $remote_addr;

          proxy_set_header        Host $host;
          proxy_pass http://10.150.1.99:8080; #gerrit地址
        }
}

```
- 创建admin用户
在Nginx服务器创建登录用户
由于使用的认证类型是http，所以以后创建用户都需要手动在此创建。
```
 htpasswd -c /etc/nginx/conf.d/passwords gerrit
```
## 配置修改

> vim review_site/etc/gerrit.config
```
[gerrit]
	basePath = git
	serverId = 1e6c64ad-57fe-4dcb-8c7b-1b8c36c2da48
	canonicalWebUrl = http://xxx.xxx.xxx.xxx:10083/  #nginx代理地址
[database]
	type = mysql
	hostname = xxx.xxx.xxx.xxx
	port = 3306
	database = gerritdb
	username = gerrit
[index]
	type = LUCENE
[auth]
	type = HTTP  #认证方式
[receive]
	enableSignedPush = false
[sendemail]      #发件邮箱设置，在安装过程已设置，如果没有可在此配置，密码存在secure.config文件
	smtpServer = smtp.sina.com
	smtpServerPort = 25
	smtpUser = send_not
        from = xxx@sina.com
[container]
	user = gerrit
	javaHome = /usr/local/jdk1.8.0_181/jre
[sshd]
	listenAddress = *:29418
[httpd]
	listenUrl = http://*:8080/
[cache]
	directory = cache
[plugins]
    allowRemoteAdmin = true

```

- 重启gerrit和Nginx‎
```
review_site/bin/gerrit.sh restart

nginx -s reload
```

## 登录验证
![](images/login.png)
- 添加邮箱
 ID为1000000的是admin用户
![](images/mail1.png)
![](images/mail2.png)

## 安装插件
上面一键安装后，默认没有安装上任何插件的，如果用到插件，需要手动安装
- replication插件
> gerrit replication插件可以实现gerrit与gitlib同步

**安装：**
```
unzip gerrit-2.15.5.war
‎cp WEB-INF/plugins/replication.jar ~/temp/
ssh -p 29418 gerrit@127.0.0.1 gerrit plugin install -n replication.jar - <~/temp/replication.jar
ssh -p 29418 gerrit@127.0.0.1 gerrit plugin ls
```
**配置ssh config 示例**
```
cd ~/.ssh/
vim config
Host gitlab.***.cn
        User gitlabowner
        IdentityFile ~/.ssh/id_rsa #gitlab owner id_rsa
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null
```
**替换gitlab lubase(project owner) ssh key 示例**
```
cd ~/.ssh/
rm id_rsa
rm id_rsa.pub
vim id_rsa
(粘贴owner的id_rsa)
vim id_rsa.pub
(粘贴owner的id_rsa.pub)
 id_rsa
 id_rsa.pub
```
**加入gitlab pubkey到kown_hosts 示例**
```
sh -c "ssh-keyscan -t rsa gitlab.***.cn >> ~/.ssh/known_hosts"
sh -c "ssh-keygen -H -f ~/.ssh/known_hosts"
```
**配置replication.config 示例**
> vim gerrit_site/etc/replication.config
```
[remote "gitlab.***.cn"]
        url = git@gitlab.***.cn:mobile/${name}.git
        push = +refs/heads/*:refs/heads/*
        push = +refs/tags/*:refs/tags/*
        push = +refs/changes/*:refs/changes/*
        timtout = 30
        threads = 3
```
**启动replication**
```
bin/gerrit.sh restart
ssh -p 29418 gerrit@127.0.0.1 gerrit plugin reload replication
ssh -p 29418 gerrit@127.0.0.1 replication start ***
```
## 参考
> [1]: http://www.cnblogs.com/kevingrace/p/5624122.html "cnblogs"
> 
> [2]: https://www.bbsmax.com/A/mo5kYQWzwR/
> 
> https://blog.csdn.net/tq08g2z/article/details/78627653
> 
> https://gerrit-documentation.storage.googleapis.com/Documentation/2.15.3/intro-how-gerrit-works.html