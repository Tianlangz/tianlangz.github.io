---
author: Hugo Authors
title: 数据库-视图，高级SQL特性
date: 2022-04-21
description:  数据库
series:
  - 数据库

---
视图、索引、约束

- 视图
  - 简化复杂的SQL语句
  - 使用表的一部分而不是整个表
  - 保护数据
  - 更改数据格式和表示 

<!--more-->
# 视图的规则与限制
  - 与表一样，视图必须唯一命名
  - 视图是可以嵌套的，可以从其他的视图中检索数据来构造新的视图

# 新建查询
 新建查询的方法和新建一个表格的方法大体相似。
 在新建的视图中可以检索出多个表中的多个数据插入，让我们在更方便的调用其中的数据。
   - 计算字段
   - 筛除不想要的数据
   - 重新格式化检索出的数据
 利用新建的查询来查询我们想要的字段例如:
 我们新建一个查询:
 ```sql
 CREATE VIEW ProductCustomers AS
 SELECT cust_name, cust_contact, prod_id
 FROM Customers, Orders, OrderItems
 WHERE Customers.cust_id = Order.cust_id
   AND OrderItems.order_num = Orders.order_num;
 ```
 利用这个视图来简化我们，我们的查询语句:
 ```sql
 SELECT cust_name, cust_contact
 FROM ProductCustomers
 WHERE prod_id = 'RGAN01';
 ```

# 约束

 约束修改的话一定要，先删再改。

 ## 主键
   - 主键具有唯一性且不允许为`NULL`值。
   - 包含主键值的列从不修改或更新。
   - 主键值不能重用。如果从表中删除某一行，主键值不分配给其他行。

   两种方式定义主键:
   ```sql
   CREATE TABLE Vendors
   (
     vend_id         CHAR(10)  NOT NULL  PRIMARY KEY,
     vend_name       CHAR(50)  NOT NULL,
     vend_address    CHAR(50),
     vend_city       CHAR(50),
     vend_state      CHAR(5),
     vend_zip        CHAR(10),
     vend_country    CHAR(50)
   )
   ```
   上述语句就是将`vend_id`列设为了主键。
   这种语句我们还有另一种写法，可以不将主键直接写在列名后
   例如:
   ```sql
   CREATE TABLE Vendors
   (
     vend_id         CHAR(10)  NOT NULL,
     vend_name       CHAR(50)  NOT NULL,
     vend_address    CHAR(50),
     vend_city       CHAR(50),
     vend_state      CHAR(5),
     vend_zip        CHAR(10),
     vend_country    CHAR(50),
     CONSTRAINT PRIMARY KEY (vend_id)
   )
   ```
   这两种写法作用是一样的。当然我们不可能只在新建时将一个列定义成主键，我们在后期也可以修改。
   例如:
   ```sql
   ALTER TABLE Vendors
   ADD CONSTRAINT PRIMARY KEY (vend_id);
   ```

 ## 外键
   外键也是表中的一列，其值必须列在另一个表中的主键列。
   外键是保证引用完整性的极其重要的部分。
   创建外键和主键的方法大体相似。
   例如：
   ```sql
   CREATE TABLE Orders
   (
     order_num   INTEGER   NOT NULL    PRIMARY KEY,
     order_date  DATETIME  NOT NULL,
     cust_id     CHAR(10)  NOT NULL    REFERENCES Customers (cust_id)
   );
   ```
   第二种新建方式
   ```sql
   CREATE TABLE Orders
   (
     order_num   INTEGER   NOT NULL,
     order_date  DATETIME  NOT NULL,
     cust_id     CHAR(10)  NOT NULL,
     CONSTRAINT PRIMARY KEY (order_num),
     CONSTRAINT FOREIGN KEY (cust_id) REFERENCES Customers (cust_id)
   );
   ```
   修改和主键的方法是差不多的，因为都是利用`CONSTRAINT`关键字来修改我们想要修改的约束。
   ```sql
   ALTER TABLE Orders
   ADD CONSTRAINT FOREIGN KEY (cust_id) REFERENCES Customers (cust_id);
   ```
   外键有助防止意外删除，在定义外键后，DBMS不允许删除在另一个表中具有关联行的行。
   当然如果硬要删除我们也有办法，例如用`CASCADE`级联关键字来删除有两个表相关联的数。


 ## 唯一约束
   唯一约束是保证列中的数据保持唯一性它们类似于主键，但不是主键。
     - 表中可以包含多个唯一约束，但只能包含一个主键。
     - 唯一约束列可以包含`NULL`值。
     - 唯一约束列可更改可更新，只要保证其唯一性即可。
     - 唯一约束列的值可随意使用。
     - 与主键不一样，唯一性不能定义外键。
   它的设置方式和主键外键基本一样，就是将`UNIQUE`放到原来主键外键的地方就是将唯一约束加上去了。
   例如:
   ```sql
   CREATE TABLE Orders
   (
     order_num   INTEGER   NOT NULL,
     order_date  DATETIME  NOT NULL,
     cust_id     CHAR(10)  NOT NULL,
     CONSTRAINT PRIMARY KEY (order_num),
     CONSTRAINT FOREIGN KEY (cust_id) REFERENCES Customers (cust_id),
     CONSTRAINT UNIQUE (cust_id)
   );
   ```
   第二种方法和更改的方式我就不过多赘述了，和主键外键的方式是一样的。

 ## 检查约束
   这是一种除上述三种约束方式外的一种约束，只能允许特定的值进入列中。举几个常见的用途:
     - 检查最大最小值
     - 指定范围
     - 只允许特定的值
   例如:
   ```sql
   CREATE TABLE OrderItems
   (
     order_num     INTEGER     NOT NULL,
     order_item    INTEGER     NOT NULL,
     prod_id       CHAR(10)    NOT NULL,
     quantity      INTEGER     NOT NULL    CHECK ( quantity > 0 ),
     item_price    MONEY       NOT NULL
   );
   ```
   这个程序将利用这个约束将`quantity`这一列，输入的值都做一边筛查，保证其中的值都大于0。

   类似的，我们还可以将`gender`的列只包含 M 或 F ，可编写如下的修改语句:
   ```sql
   ADD CONSTRAINT CHECK (gender LIKE '[MF]');
   ```



# 索引
  索引是用来排序数据以加快搜索和排序操作的速度。
    - 索引改善检索操作的性能，但降低了数据插入、修改和删除的性能。再执行这些操作时，DBMS也会实时更新索引中的数据。
    - 索引数据可能需要占用大量的存储空间。
    - 并非所有数据都适合做索引。
    - 索引用于数据过滤和数据排序。
    - 可以在索引中定义多个列
  索引的创建:
  ```sql
  CREATE INDEX prod_name_ind
  ON Products (prod_name);
  ```
  索引必须唯一命名。这里的索引名`prod_name_ind`在关键字`CREATE INDEX`之后定义。在建立索引时我们还可以选择建立一个唯一索引还是一个聚簇索引，而这两种索引的建立方式是在`CREATE`后加`UNIQUE`唯一或`CLUSTER`聚簇来表示我们建立什么样的索引。`ON`用来指定被索引的表，而索引中包含的列(此例中仅有一列)在表名后的圆括号中给出。在括号中，我们还可以规定其是升序还是降序。

  ## 更改索引名
    更改索引名的方式:
   ```sql
   ALTER INDEX <prod_name_ind> RENAME TO <prod_name_inc>
   ```