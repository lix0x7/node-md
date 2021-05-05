# 基本SQL，以MySQL为例


- 基本检索
   - `select * from table where …;`
   - 检索不同值，关键字`DISTINCT`
      - `select distinct id from table where …;`
   - 限定检索结果行数，关键字`LIMIT`，`OFFSET`代表起始偏移量
      - `select * from table LIMIT 5 OFFSET 5;`
- 排序
   - 使用`ORDER BY`子句
      - `select * from table ORDER BY id, rank`
      - 可以在列名后使用DESC指定排序方向，每一个DESC只对其前面的列名有效
         - `select * from table ORDER BY id DESC, rank;`
- 筛选
   - `WHERE`子句
      - 除了常见的比较符还有IS NULL和BETWEEN
         - `select * from table where rank BETWEEN 2 AND 3;`
- 逻辑操作符
   - SQL的比较时用单等于号`=`即可
   - `AND`
   - `OR`
   - `IN`
      - `select name from table where name IN ('Lee', 'Mark');`

需要注意的是IN不能使用索引，所以效率很低

   - `NOT`
- 通配符 只能用于文本字段
   - `LIKE`
      - `%` 表示任意字符出现任意次，可以使用多次并且使用在任意位置
         - 例如找出所有Fish开头商品：

`select name from table where name LIKE 'Fish%';`

      - `_` 表示任意字符出现一次，可以多次使用
         - 例如 `select name from table where name LIKE '_is_'`

上一句会匹配fish而不会匹配fishy

- 计算字段
   - Concat() 使用Concat函数拼接字段
`select Concat(name, '(', country, ')') from table order by name`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570346-6b09ddfd-0190-4fe4-b1f2-dce54f73a748.png#align=left&display=inline&height=125&margin=%5Bobject%20Object%5D&originHeight=125&originWidth=465&size=0&status=done&style=none&width=465)
   - TRIM() 清除空白字符
很多数据库将字段宽度填充字符也输出，由此导致上文的空白字符问题，可以使用TRIM函数清除空白字符，TRIM为清除两端空白符，LTRIM和RTRIM分别清除左侧和右侧空白符
`select Concat(trim(name), '(', trim(country), ')') from table order by name`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570376-2542dce9-58c9-4c95-ba43-e47ad172ca26.png#align=left&display=inline&height=121&margin=%5Bobject%20Object%5D&originHeight=121&originWidth=195&size=0&status=done&style=none&width=195)
   - AS 别名
可以看到上述小节拼接出来的字符串是没有列名的，这种情况下只能在sql的控制台看到输出，而不能在程序语言输出中对其进行引用，所以需要设置别名
`select Concat(trim(name), '(', trim(country), ')') AS vend_title from table order by name`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570615-9c5e9c46-9b39-4df5-8178-de3127d31e9a.png#align=left&display=inline&height=156&margin=%5Bobject%20Object%5D&originHeight=156&originWidth=477&size=0&status=done&style=none&width=477)
   - `+-*/` 算数计算
```sql
select 
		id, product, item_price, 
		quantity*item_price AS expanded_price
from OrderItems
where order_num = 20008
```

   - 
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570632-7f9dc037-01d8-4774-a641-385f1e69bd9f.png#align=left&display=inline&height=129&margin=%5Bobject%20Object%5D&originHeight=129&originWidth=477&size=0&status=done&style=none&width=477)
- 使用函数处理数据
   - 文本类
      - UPPER()、LOWER()
      - LENGTH()
      - TRIM()
   - 日期时间类，此部分不同的数据库函数名很混乱，下述以MySQL为例
      - YEAR() 取date字段的年份
   - 计算类
      - ABS()、SQRT()、COS()、SIN()、TAN()、EXP()
- 汇总数据，聚集计算
   - AVG()
      - `NULL`值当做0看待
      - 只能对单列使用
   - COUNT()
      - 使用COUNT(*)是统计行数
      - 使用COUNT(COLUMN)是统计指定列具有特定值的行数。会包含重复的，如果不希望重复则先使用DISTINCT去重
   - MAX()、MIN()、SUM()
- 分组数据
   - GROUP BY
`select vend_id, count(*) as num_prods from Products GROUP BY vend_id;`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570421-770ea144-5d05-498c-837d-ac9c2c2b8210.png#align=left&display=inline&height=102&margin=%5Bobject%20Object%5D&originHeight=102&originWidth=179&size=0&status=done&style=none&width=179)
      - 作用是使得select中的聚集计算式count范围限定在每个group中
      - 所有的NULL会被归为一组
      - group by可以跟多列
      - 必须出现在WHERE后，ORDER BY之前。
   - HAVING
```sql
select cust_id, count(*) as orders
from Orders
group by cust_id
HAVING count(*) >= 2;
```

   - 
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570386-bc57d9f0-609b-4ba7-be89-172a493e653a.png#align=left&display=inline&height=59&margin=%5Bobject%20Object%5D&originHeight=59&originWidth=214&size=0&status=done&style=none&width=214)
      - 每一个HAVING作用对象是一个GROUP，可以将其看做group级别的WHERE子句；换句话说，WHERE是行级过滤，HAVING是组级过滤
      - 根据前文提到的GROUP BY子句必须出现在WHERE子句之后可知，WHERE是在分组前过滤数据，而HAVING是在分组后过滤数据
- 子查询
   - WHERE子句的子查询
```sql
select cust_id from Orders
where order_num in (
	select order_num from OrderItems where prod_id='RGAN01'
);
```

      - SELECT语句中，查询总是由内向外进行的。
      - 作为WHERE子句的子查询的只能是**单列**！
      - 子查询嵌套是没有限制的，但是为了性能考虑，还是不要嵌套太多
   - 计算字段的子查询
      - 查询：Customs表中每个顾客的订单总数，订单与顾客记录存储在Orders表中。
```sql
select customer_id, (
	select count(*) 
	from Orders 
	where Orders.customer_id = Customers.customer_id) as order_count
from Customers;
```

- 表别名 用于缩短表名，方便输入
```sql
select vend_name, prod_name, prod_price
from Vendors AS V, Products AS P
where V.vend_id = P.vend_id
```

- 联结 join
   - 当时用join关键字时，将where子句的where替换为on
   - 例如，有如下两张表：
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570428-65a0bceb-98e1-44ae-b68d-36717ba3f18e.png#align=left&display=inline&height=159&margin=%5Bobject%20Object%5D&originHeight=159&originWidth=196&size=0&status=done&style=none&width=196)
   - 笛卡尔积
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570407-7285136e-68b6-415c-b135-ce49efcccd1f.png#align=left&display=inline&height=99&margin=%5Bobject%20Object%5D&originHeight=99&originWidth=370&size=0&status=done&style=none&width=370)
      - 返回笛卡尔积的联结有时也称为`cross join`
```sql
select vend_name, prod_name, prod_price
from Vendors, Products
where Vendors.vend_id = Products.vend_id
```

         - 上述SQL语句语义如下：将Vendors和Products两张表进行笛卡尔积操作，再对得到的结果取满足where子句条件的列，取对应列，返回结果
   - 内联结 `inner join`
      - 在两个表的笛卡尔积基础上加入了等值判断
      - `Select …… from 表1 inner join 表 2 on 表1.A=表2.E`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570413-28b6def4-5a4b-4bec-8e32-d233a2127c03.png#align=left&display=inline&height=45&margin=%5Bobject%20Object%5D&originHeight=45&originWidth=370&size=0&status=done&style=none&width=370)
   - 自然连接 `natural join`
在内联结的基础上增加了以下限定条件：
      - 比较操作必须在同名列上进行
      - 产生的结果会消除同名列，将其合并为一列
      - `Select …… from 表1 natural join 表2`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570420-dcd1e9ae-39eb-4b87-9cce-4e79c727b006.png#align=left&display=inline&height=47&margin=%5Bobject%20Object%5D&originHeight=47&originWidth=303&size=0&status=done&style=none&width=303)
   - 外联结
      - 与内联结的区别在于，内联结不会对等值判断不存在列进行扩充，而外联结则一定会保证指定方向的表中的每一列都会出现在联结结果中（如果指定方向的表不满足联结条件，那么为了保证出现，则对其再另一侧表中补NULL行）
      - left join和right join只是根据方向不同而有所区别，下文以left join为例
      - `Select …… from 表1 left outer join 表2 on 表1.C=表2.C`
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577093570440-72998ec7-0053-4795-8aa5-870d940c722d.png#align=left&display=inline&height=64&margin=%5Bobject%20Object%5D&originHeight=64&originWidth=296&size=0&status=done&style=none&width=296)
- UNION
   - 组合多个查询的查询结果
   - 例如，查询所有身在IL、IN、MI州的顾客和Fun4All的数据：
```sql
select cust_name, cust_contact, cust_email
from Customers
where cust_state in ('IL', 'IN', 'MI')
UNION
select cust_name, cust_contact, cust_email
from Customers
where cust_name='Fun4All'
```

   - UNION的行为规则
      - 每个查询必须具有相同的列，他们的顺序可以不同
      - 列数据类型必须兼容
      - UNION会消除结果中重复的行，如果不希望消除，使用UNION ALL
      - 所有的UNION结果共用同一个ORDER BY子句，它会对UNION之后的结果进行统一排序
- 插入数据
   - 插入语句 `INSERT INTO`
```sql
INSERT INTO Customers 
(cust_id, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country, cust_contact, cust_email) 
VALUES
(' 1000000006', 'Toy Land', '123 Any Street', 'New York', 'NY', '11111', 'USA', NULL, NULL);
```

   - 
INSERT SELECT，将查找的数据插入到表中
```sql
INSERT INTO Customers 
(cust_id, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country, cust_contact, cust_email) 
select
cust_id, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country, cust_contact, cust_email)
from
CustomersNew;
```

   - 复制表
```sql
create table CustCopy as
select * from Customers;
```

- 更新数据
```sql
update Customers
set cust_email='new@mail.com', cust_contact='Sam Roberts'
where cust_id='1000000005';
```

- 删除数据
```sql
delete from Customers
where cust_id='1000000005';
```

- 创建表和操纵表
   - 创建 create
   - 修改表结构 alter table
   - 删除表 drop table
- 视图
   - 优点
      - 重用SQL语句
      - 保护底层表结构，可以只对某些用户开放视图权限
   - 创建、删除
      - create view
```sql
create view ProductCustomers as
select cust_name, cust_contact, prod_id
from Customers, Orders, OrderItems
where Customers.cust_id = Orders.cust_id and OrderItems.order_num = Orders.order_num;
```

      - drop view
   - 视图的核心在于保存了一个select语句的结果作为一张虚拟表，只要是select支持的，视图都可以支持
   - 不能在视图上创建索引或触发器。
- 存储过程
- 事务 `transaction`
   - 通过在一个`transaction`中设置检查点，可以方便的在发生异常时回退至检查点，从而保证数据库的完整性
   - 通过`ROLLBACK [operation]`撤销上一步该操作
   - 通过`ROLLBACK TO [savepoint_name]`回退值指定检查点
   - 通过`COMMIT [tansaction_name]`提交事务
- 约束
   - 主键
   - 外键
      - REFERENCES Customers(cust_id)
      - 保证引用完整性
      - 定义为外键的列的值必须为其他表中的主键
   - 唯一约束
   - 检查约束
      - `check(count>=0)`
      - 保证列中的值满足设定的条件
- 索引
   - Tips
      - 索引改善了查找性能，但是因为更新索引需要时间消耗，所以相应地降低了插入、修改、删除的性能
      - 索引数据需要占用额外的存储空间
      - 数据的值越多，索引效果一般越好。（例如，对姓名索引带来的性能改善要比对性别索引带来的改善要大）
      - 如果经常对某列数据进行排序或过滤，那这一列就适合添加索引
   - 创建
      - `create index [index_name] on [table_name](column_name)`
- 触发器



