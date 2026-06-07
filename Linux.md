# 1.Linux

## 1.文件与目录操作

```shell
ls  ：列出目录内容
	常用参数: -1（长格式）、-a（显示隐藏）、-h（人类可读）、-t（按照时间排序） 

cd	：切换目录
	常用参数: ~（家目录）、-（上次目录）

pwd	：显示当前路径

mkdir	：创建目录
	常用参数：-p（递归创建父目录）

rmdir	:删除空目录
	常用参数：-

rm	:删除文件或目录
	常用参数：-r（递归）、-f（强制）、-i（交互）

cp	:复制文件/目录
	常用参数：-r（递归复制目录）、-i（交互）、-p（保留属性）、-a（归档）

mv	:移动/重命名
	常用参数：-i、-u（更新）

touch:创建空文件或更新时间戳
	常用参数：-

ln	:创建链接文件
	常用参数：-s（软链接）
```

## 2.查看文件内容

```shell
cat	:全文输出（小文件）
	常用参数：-n（显示行号）

more:分页查看（向下）
	常用参数：空格翻页，回车下一行

less:可上下翻页，功能更强
	常用参数：/搜索，q退出，G到最后

head:查看文件头部
	使用参数：-n 20（前20行）

tail:查看尾部，常用于日志监控
	使用参数：-n 20、-f（实时跟踪）

grep:按模式过滤文本
	使用参数：-i（忽略大小写）、-v（反向）、-n（显示行号）、-r（递归）
```

## 3.文本处理

```shell
sed	:流编辑，批处理替换
	常用实例：sed 's/old/new/g' file （替换）
			 sed -i ‘2d' file（删除第2行）

awk	:列提取、格式化报表
	常用实例：awk '{prink $1}'（打印第1列）
			awk -F',' '{print $2}'（按逗号分隔）

sort:排序
	常用实例：sort -k2 -n（按第2列数字排序）
			sort -r（逆序）

uniq:去除（需要先排序）
	常用实例：uniq -c（统计重复次数）
			uniq -d（只显示重复行）

cut	:按分隔符获取
	常用实例：cut -d':' -f1 /etc/passwd

wc	:统计
	常用实例：wc -1（行数）、-w（单词数）、-c（字节数）

tr	:字符替换或删除
	常用实例：tr 'a-z' 'A-Z'（大小写转换）
			tr -d '\n'（删除行）
```

## 4.权限管理

```shell
chmod:修改权限（数字/符号模式）
	常用参数：chmod 755 file
			chmod u+x file

chown:修改所有者与组
	常用参数：chown user:group file

chgrp:修改组（较少单独用）
	常用参数：chgrp group file

umask:设置默认权限密码
	常用参数：umask 022
权限数字含义：4=t,2=w,1=x.例如755->(rwx),组:5(r-x),其他:5(r-x)
```

## 5.进程管理

```shel
ps	:查看进程快照
	常用参数：ps aux（BSD风格）、ps -ef（System V风格）

top	:动态实时监控
	常用参数：-p PID（监控指定进程）、交互按p（CPU）m（内存）排序

htop:进阶监控
	常用参数：更友好界面（需安装）

kill:发送信号终止进程
	常用参数：kill -9 PID（强制终止），kill -15 PID（优雅终止）

pkill:按进程名终止
	常用参数：pkill java

jobs:查看后台任务
	常用参数：jobs -1

fg/bg:作业控制
	常用参数：fg %1（将后台任务调到前台）、bg %1（让后台暂停任务继续运行）

nohup:后台运行，忽略挂断信号
	常用参数：nohup command &
```

## 6.网络相关

```shell
netstat:查看端口占用
	常用参数：-tunlp（tcp/udp/数字/监听/程序名）

ss	:替代netstat
	常用参数：-tunlp（更快的netstat）

lsof:查看端口/文件打开情况
	常用参数：-i:8080（查看占用8080端口的进程）

ping:测试连通性
	常用参数：-c 4（限制次数）

telnet:测试端口是否开放
	常用参数：telnet host port 

curl:HTTP 请求测试
	常用参数：curl -i http://xxx（查看响应头）
			curl -0 url（下载文件）

wget:下载文件
	常用参数：wget url

tcpdump:网络抓包（需root）
	常用参数：tcpdump -i eth0 port 80
```

## 7.磁盘与内存

```shell
df	:查看磁盘分区使用情况
	常用参数：-h（人类可读）、-i（查看inode）
	
du	:查看目录或文件占用空间
	常用参数：-sh *（统计当前目录各子项大小）
			-sh （总大小）

free:查看内存使用
	常用参数：-h、-m（MB单位）

vmstat:系统修整性能（CPU、内存、IO）
	常用参数：vmstat 1 5（每秒一次，共5次）
	
iostat:磁盘I/O统计
	常用参数：-x 1
	
fdisk:磁盘分区操作
	常用参数：-1（列出分区）
```

## 8.压缩与打包

```shell
tar:最常用打包工具
	常用参数：tar -czvf archive.tar.gz dir/（打包+压缩gzip）
			tar -xzvf archive.tar.gz（解压）

gzip/gunzip:单文件压缩
	常用参数：gzip file（压缩为.gz）
	
zip/unzip:windows兼容格式
	常用参数：zip -r archive.zip dir/、unzip archive.zip
```

## 9.查找与定位

```shell
find:强大文件搜索
	常用参数：find / -name "*.log" -type f （按名称）
			find . -mtime -7 （7天内修改）
			find . size  +1G （大于1G）

locate:快速查找（非实时）
	常用参数：locate java （基于数据库，需updatedb）
	
which:查找命令路径
	常用参数：which java 

whereis:查找二进制、源码、手册
	常用参数：whereis java
```

## 10.系统信息与日志

```shell
unname -a:查看内核版本、系统架构

cat /etc/os-release:查看发行版本信息

uptime:查看系统负载及运行时间

demsg:内核环缓冲区（硬件、驱动信息）

journalctl:systemed 日志（-u service 查看服务日志）

tail -f /var/log/messages 或 /var/log/syslog:实时系统日志
```

## 11.shell脚本常用

```shell
$0,$1,$@,$# :脚本参数
$? :上一条命令的退出码（0成功）
&&，||：逻辑与/或（短路）
>,>>,2>,2>$1:重定向输出或错误
|:管道（前命令输出作为后命令输入）
xargs:将前命令输出作为参数传给后命令，如 find ...|xargs rm
tee:输出的同时保存到文件，如 command | tee log.txt
```

## 12.题目

```shell
1. 查找所有 Java 进程并杀死
ps aux | grep java | grep -v grep | awk '{print $2}' | xargs kill -9

2. 统计访问日志中 IP 出现次数，并排序取前10
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

3. 查找当前目录下 7 天前修改的 .log 文件并删除
find . -name "*.log" -type f -mtime +7 -exec rm {} \;

4. 实时监控日志文件中包含 "ERROR" 的行
tail -f app.log | grep ERROR

5. 查看 8080 端口是否被监听
netstat -tunlp | grep 8080 或 ss -tunlp | grep 8080

6. 将当前目录下所有 .txt 文件中的 "foo" 替换为 "bar"
sed -i 's/foo/bar/g' *.txt
```

## 13.复习建议

```shell
动手敲：在虚拟机或 WSL 中逐个命令实践，特别是 grep、awk、sed 的组合。
理解 man：遇到陌生参数用 man command 查看帮助。
记住常用参数：-r（递归）、-f（强制）、-v（反向）、-i（忽略大小写）、-n（行号）。
场景化记忆：比如日志排查用 tail -f、grep；CPU 高用 top、ps、jstack；磁盘满用 df、du、find。
```

