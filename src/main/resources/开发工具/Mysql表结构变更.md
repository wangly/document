###修改字段名称：
>alter table table_name change column old_column_name new_column_name column_define;

结合after、before，first等关键字，可指定列的顺序。

###修改字段类型：
>alter table table_name modify column column_name new_column_define;

除了不能重命名列，它的功能和change相同。

###同时修改多个字段类型：
>alter table table_name modify column column_name new_column_define, modify column column_name1 new_column_define1...

###新增字段：
>alter table table_name add column column_define;

###设置或删除列的默认值
>alter table table_name altercolumn column_name setdefault default_value;

>alter table table_name altercolumn column_name dropdefault;

###修改表修改自增点
>alter table table_name auto_increment = 1; 

###修改表名
>alter table table_name rename new_table_name;

在实践中，注意，线上数据库操作需要到ops数据库平台操作，自助表变更，且限制扫描记录数不可超过1000行。这种情况下，有些操作必须请DBA协助。

###添加PRIMARY KEY（主键索引）
>mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )

###添加UNIQUE(唯一索引) 
>mysql>ALTER TABLE `table_name` ADD UNIQUE uni_name(`column`) 

###添加INDEX(普通索引) 
>mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` )

###添加FULLTEXT(全文索引) 
>mysql>ALTER TABLE `table_name` ADD FULLTEXT (`column`) 

###添加多列索引
>mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )

###删除索引
>mysql>ALTER TABLE `table_name` drop INDEX index_name