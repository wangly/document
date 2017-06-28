##一、非常规更新
批量更新
>update lock_fix_contract set city = case city
when '北京' then '北京市'
when '武汉' then '武汉市'
when '青岛' then '青岛市'
else city
end
where id > 276

注意：若非全量更新，一定要记得带上else，否则，非匹配的记录，该字段将被清空。
对已存在记录，替换而不新增
方案一：
>insert into contract_employee (id,name,mis,tel) values (1,"123","123","1234567")
ON DUPLICATE KEY UPDATE name = '123', mis = '123', tel='1234567'

缺陷：字段多时，编码麻烦。
 
方案二：
>replace into contract_employee  (id,name,mis,tel) values (2,"123456","123","1234567")

注意：
此处不可再使用"select last_insert_id()"查询更新记录id，否则将覆盖原id，改写为0；并且，replace实现是删除记录后，重新添加，如果更新数据信息项补全，将导致该项被清空。例：
replace into contract_employee (id, name, mis, tel) values (2, '124325', '2134', null)
此时，该记录中原有的tel将被设置为NULL；
 
方案三：若已存在，则不更新
>insert into  contract_employee  (id,name,mis,tel)
select #{id},#{name},#{mis},#{tel} from contract_employee
where not exists (select id from contract_employee where id = #{id)

 在实际更新门锁扣款记录中，我们的最佳实践是：
>insert into lock_deduct_record (lock_sign_id, fact_total_deduction, deduct_time, deduction_status,
gmt_create, gmt_modify)
values (#{lockSignId,jdbcType=BIGINT}, #{factTotalDeduction,jdbcType=INTEGER}, #{deductTime,jdbcType=TIMESTAMP}, #{deductionStatus,jdbcType=INTEGER},
now(), now())
ON DUPLICATE KEY UPDATE
<if test="factTotalDeduction != null">
  fact_total_deduction = #{factTotalDeduction,jdbcType=INTEGER},
</if>
<if test="deductTime != null">
  deduct_time = #{deductTime,jdbcType=TIMESTAMP},
</if>
<if test="deductionStatus != null">
  deduction_status = #{deductionStatus,jdbcType=INTEGER},
</if>
gmt_modify = now(),
lock_sign_id = #{lockSignId,jdbcType=BIGINT}

该方案的前提是，更新或插入的依据必须主键。

##二、常用函数
查询时间间隔：datediff、timediff
date_format：时间戳转日期
unix_timestamp：日期转时间
substring函数
mysql中的substring的beginIndex不同于java中String的substring;
java中的beginIndex从0开始
"DDGJ-A6-00001234-001".substring(0)
Result："DDGJ-A6-00001234-001"
"DDGJ-A6-00001234-001".substring(1)
Result："DGJ-A6-00001234-001"
mysql中的beiginIndex从1开始：
select substring('DDGJ-A6-00001234-001', 1) from lock_fix_contract limit 1
Result：DDGJ-A6-00001234-001
select substring('DDGJ-A6-00001234-001', 0) from lock_fix_contract limit 1
Result：null
##三、使用正则表达式
参考：http://blog.csdn.net/vvhesj/article/details/22299413
常见问题：Column count doesn't match value count at row 1
原因：存储的数据与数据库表的字段类型定义不相匹配.检查字段是否匹配，类型是否正确, 是否越界, 有无把一种类型的数据存储到另一种数据类型中.