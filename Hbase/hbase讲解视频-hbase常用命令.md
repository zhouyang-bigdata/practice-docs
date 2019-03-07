# hbase常用shell命令





```
list
```

描述：

```sh
describe 'key_test_table'
scan 'key_test_table'
count 'key_test_table'
```

创建表：

```sh
create 'key_test_table',{NAME=>'f1',VERSION=>1}
```

建立一个有3个column family的表

```shell
create 't1', {NAME => 'f1', VERSIONS => 1},  {NAME => 'f2', VERSIONS => 1}, {NAME => 'f3', VERSIONS => 1}
```



创建表，带压缩：

```sh
create 'key_test_table', 'f1', {NUMREGIONS => 100, SPLITALGO => 'HexStringSplit'}
```

创建表，带压缩，开启REPLICATION复制功能

```sh
create 'key_test_table', 'f1', {NAME=>'f1', SPLITALGO => 'HexStringSplit', REPLICATION_SCOPE =>1}, SPLITS => ['S0','S1','S2', 'S3', 'S4','S5','S6','S7','S8','S9']
```

创建表，有10个分区：

```sh
create 'key_test_table', 'f1', SPLITS => ['S0','S1','S2', 'S3', 'S4','S5','S6','S7','S8','S9']
```

创建表，有100个分区

```sh
create 'key_test_table', 'f1', SPLITS => ['S00','S01', 'S02', 'S03', 'S04','S05','S06','S07','S08','S09','S10','S11', 'S12', 'S13', 'S14','S15','S16','S17','S18','S19','S20','S21','S22', 'S23', 'S24','S25','S26','S27','S28','S29','S30','S31', 'S32', 'S33', 'S34','S35','S36','S37','S38','S39','S40','S41', 'S42', 'S43', 'S44','S45','S46','S47','S48','S49','S50','S51','S52', 'S53', 'S54','S55','S56','S57','S58','S59','S60','S61', 'S62', 'S63', 'S64','S65','S66','S67','S68','S69','S70','S71', 'S72', 'S73', 'S74','S75','S76','S77','S78','S79','S80','S81','S82', 'S83', 'S84','S85','S86','S87','S88','S89','S90','S91', 'S92', 'S93', 'S94','S95','S96','S97','S98','S99']
```



删除表：

```sh
disable 'key_test_table'
drop 'key_test_table'
```

删除表中所有数据：

```sh
truncate 'key_test_table'
```

修改表

```sh
disable 'key_test_table'
alter 'key_test_table',{NUMREGIONS => 100, SPLITALGO => 'HexStringSplit',REPLICATION_SCOPE =>1}
enable 'key_test_table'
```



分配权限：

```sh
grant 'root','RWXCA','key_test_table'
```

查看权限

```shell
user_permission 'key_test_table'
```



添加数据(rowkey:S04_002904_1045_3)

```shell
put 'key_test_table','S0_002904_1045_1','f1:col1','value01'
```

```shell
put 'key_test_table','S0_002904_1045_1','f1:col2','value02'
```

落在别的分区：

```shell
put 'key_test_table','S1_002904_1045_1','f1:col2','value02'
```



##### 例如：查询表t1，rowkey001中的f1下的col1的值**

```sh
get 'key_test_table','S04_002904_1045_3', 'f1:col1'
```

```sh
scan 'key_test_table',{LIMIT=>5}
```

分区：

```sh
NUMREGIONS => 3
```

