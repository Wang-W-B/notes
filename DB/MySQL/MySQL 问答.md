### 自增 ID 问题
**问**：如果有一张表，里面有个字段为id的自增主键，当已经向表里面插入了10条数据之后，删除了id为8，9，10的数据，再把mysql重启，之后再插入一条数据，那么这条数据的id值应该是多少，是8，还是11？

**答**：如果表的类型为 MyISAM，那么是 11。如果表的类型为 InnoDB，则 id 为 8。

这是因为两种类型的存储引擎所存储的最大 ID 记录的方式不同，MyISAM 表将最大的 ID 记录到了数据文件里，重启 mysql 自增主键的最大 ID 值也不会丢失；

而 InnoDB 则是把最大的 ID 值记录到了内存中，所以重启 mysql 或者对表进行了 OPTIMIZE 操作后，最大 ID 值将会丢失。

顺便说一下 MYSQL 获取当前表的自增值的四种方法：

1. `SELECT MAX(id) FROM <table_name>` 针对特定表

2. `SELECT LAST_INSERT_ID()`  函数   针对任何表

3. `SELECT @@identity`    针对任何表

@@identity 是表示的是最近一次向具有 identity 属性(即自增列)的表插入数据时对应的自增列的值，是系统定义的全局变量。(一般系统定义的全局变量都是以@@开头，用户自定义变量以@开头。)使用 @@identity 的前提是在进行 insert 操作后，执行`select @@identity`的时候连接没有关闭，否则得到的将是 NULL(0) 值。

4. `SHOW TABLE STATUS LIKE ‘<table_name>’`  如果针对特定表，建议使用这一种方法。得出的结果里边对应表名记录中有个 Auto_increment 字段，里边有下一个自增 ID 的数值就是当前该表的最大自增 ID。


### 显示宽度
**问**：MySQL 中，设置了数值的宽度，是否会影响数值的取值范围。

**答**：不会。

显示宽度和数据类型的取值范围是无关的。显示宽度只是指明 MYSQL 最大可能显示的数字个数，数值的位数小于指定的宽度时会有空格填充。**如果插入了大于显示宽度的值，只要该值不超过该类型整数的取值范围，数值依然可以插入，而且能显示出来**。如果不指定显示宽度，则 MYSQL 为每一种类型指定默认的宽度值。

例如，创建一个 year 字段，并设置为 int(4) 类型。当向 year 字段插入数值 19999 后，使用 select 查询的时候，MYSQL 显示的将是完整带有 5 位数字的 19999，而不是 4 位数字的值。

**显示宽度只用于显示，并不能限制取值范围和占用空间，例如：INT(3) 会占用 4 个字节的存储空间，并且允许的最大值也不会是 999，而是 INT 整型所允许的最大值。**

### SELECT 结果排序问题
当有 ORDER BY 子句的时候，会按照 ORDER BY 子句排序后返回结果。

如果没有 ORDER BY 子句，MySQL 对 SELECT 语句的返回结果有潜规则：

* 对于 MyISAM 引擎来说，其返回顺序是其物理存储顺序；
* 对于 InnoDB 引擎来说，其返回顺序是按照主键排序的。

### 连接时 localhost 与 127.0.0.1 的区别
* `localhot(local)` 不使用 TCP/IP 连接，而使用 Unix socket。它不受网络防火墙和网卡相关的的限制。

* `127.0.0.1` 是通过网卡传输，使用 TCP/IP 连接，依赖网卡，并受到网络防火墙和网卡相关的限制。

一般设置程序时本地服务用`localhost`是最好的，`localhost`不会解析成 IP，也不会占用网卡、网络资源。

有时候用`localhost`可以，但用`127.0.0.1`就不可以的情况就是在于此。猜想`localhost`访问时，系统带的本机当前用户的权限去访问，而用 IP 的时候，等于本机是通过网络再去访问本机，可能涉及到网络用户的权限。


### 将选择的结果导出到文件中时提示 --secure-file-priv

当用`SELECT ... INTO OUTFILE`把查询结果写入到文件的时候提示以下信息：

```
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

出现这个问题的原因是因为启动 MySQL 的时候使用了`--secure-file-priv`这个参数，这个参数的主要目的就是限制`LOAD DATA INFILE`或者`SELECT INTO OUTFILE`之类文件的目录位置，我们可以使用：

```sql
SELECT @@global.secure_file_priv;
```
 查询到你当前设置的路径，默认应该是`/var/lib/mysql-files`。

如果要解决这个问题，我们可以通过下面 2 种方式：

1.	将你要导入或导出的文件位置指定到你设置的路径里(绝对路径)
2.	由于不能动态修改，我们可以修改`my.cnf`里关于这个选项的配置，然后重启即可。 
> 转摘：[MySQL查询出错提示 --secure-file-priv解决方法](http://www.cnblogs.com/zhuangliu/p/6211688.html)

