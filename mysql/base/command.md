### mysql连接

```bash
mysql -u root -p

mysql -uroot -p123456
```

### 查看mysql版本

```bash
mysql --version
mysql -V

mysql  Ver 8.0.27 for macos11 on arm64 (MySQL Community Server - GPL)
```

### 创建数据库

```bash
create database test;
use test;
```

### 查询当前使用的数据库

```bash
select database();
```

### 终止一条语句

```bash
\c
```

### 退出mysql

```bash
exit;
quit;
\q
```
