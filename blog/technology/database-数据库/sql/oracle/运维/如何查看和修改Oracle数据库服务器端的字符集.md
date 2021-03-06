如何查看和修改Oracle数据库服务器端的字符集

分类: database
日期: 2014-05-20

原文地址: 

http://blog.chinaunix.net/uid-29632145-id-4263294.html

------

****[如何查看和修改Oracle数据库服务器端的字符集]() *2014-05-20 22:44:50*

分类： Oracle

**Oracle数据库**查看和修改服务器端的**字符集**的方法是本文主要要介绍的内容，接下来救让我们一起来了解一下这部分内容。
**A、oracle server 端字符集查询**
select userenv(‘language’) from dual
其中NLS_CHARACTERSET 为server端字符集
NLS_LANGUAGE 为 server端字符显示形式
**B、查询oracle client端的字符集**
$echo $NLS_LANG
如果发现你select 出来的数据是乱码，请把client端的字符集配置成与linux操作系统相同的字符集。如果还是有乱码，则有可能是数据库中的数据存在问题，或者是oracle服务端的配置存在问题。
**C、server端字符集修改**
将数据库启动到RESTRICTED模式下做字符集更改：
SQL> conn /as sysdba  
Connected.  
SQL> shutdown immediate;  
Database closed.  
Database dismounted.  
ORACLE instance shut down.  
SQL> startup mount  
ORACLE instance started.  
Total System Global Area 236000356 bytes  
Fixed Size                   451684 bytes  
Variable Size             201326592 bytes  
Database Buffers           33554432 bytes  
Redo Buffers                 667648 bytes  
Database mounted.  
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION;  
System altered.  
SQL> ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;  
System altered.  
SQL> ALTER SYSTEM SET AQ_TM_PROCESSES=0;  
System altered.  
SQL> alter database open;  
Database altered.  
SQL> ALTER DATABASE CHARACTER SET ZHS16GBK;  
ALTER DATABASE CHARACTER SET ZHS16GBK  
ERROR at line 1:  
ORA-12712: new character set must be a superset of old character set 
提示我们的字符集：新字符集必须为旧字符集的超集，这时我们可以跳过超集的检查做更改：
SQL> ALTER DATABASE character set INTERNAL_USE ZHS16GBK;  
Database altered.  
SQL> select * from v$nls_parameters;  
略  
19 rows selected. 
重启检查是否更改完成：
SQL> shutdown immediate;  
Database closed.  
Database dismounted.  
ORACLE instance shut down.  
SQL> startup  
ORACLE instance started.  
Total System Global Area 236000356 bytes  
Fixed Size                   451684 bytes  
Variable Size             201326592 bytes  
Database Buffers           33554432 bytes  
Redo Buffers                 667648 bytes  
Database mounted.  
Database opened.  
SQL> select * from v$nls_parameters;  
略  
19 rows selected. 
我们看到这个过程和之前ALTER DATABASE CHARACTER SET操作的内部过程是完全相同的，也就是说INTERNAL_USE提供的帮助就是使Oracle数据库绕过了子集与超集的校验.
这一方法在某些方面是有用处的，比如测试；应用于产品环境大家应该格外小心，除了你以外，没有人会为此带来的后果负责。
结语(我们不妨再说一次):
对于DBA来说，有一个很重要的原则就是:不要把你的数据库置于危险的境地！
这就要求我们，在进行任何可能对数据库结构发生改变的操作之前，先做有效的备份，很多DBA没有备份的操作中得到了惨痛的教训。
**D、client端字符集修改**
在 /home/oracle与 /root用户目录下的.bash_profile中
添加或修改 export NLS_LANG="AMERICAN_AMERICA.UTF8" 语句
关闭当前ssh窗口。
注意：NLS_LANG变量一定要配置正确否则会引起sqlplus 失效。
关于Oracle数据库查看和修改服务器端的字符集的方法就介绍到这里了，希望能够对您有所收获！