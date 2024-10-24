# MySQL中SQL的执行顺序 

在日常的开发工作中，我们经常会自己手写一些sql语句，但是对于这些sql语句是怎么执行的，执行的顺序又是怎么样的呢？想必各位大佬对此也是了解的，所以对sql语言的执行顺序有一定的了解的话，会更好的理解一些sql语句，从而更好的写sql语句，也有助于SQL的调优。就比如说，先使用子查询对数据进行过滤后在进行join操作还是直接使用join操作之后，再进行数据过滤。

# 一、SQL执行顺序 

下面是一条我们经常会用到也经常会手写的一条sql语句。

``` sql

select distinct
    查询列表（要查的字段）,
    max(), avg().... 聚合函数
from
    左边的表们s
连接类型(left|inner) join 
    右边的表们s
on 
    连接条件
where
    筛选条件
group by
    分组的列表(按什么字段分组)
having
    having_condition
order by
    排序的字段
limit
	pageSize

```

下面是sql语句的执行顺序

| 语法格式          | 语法含义   | 执行顺序 |
| ----------------- | ---------- | -------- |
| select            | 查询语句   | 8        |
| distinct          | 去重       | 9        |
| sum(), avg()...   | 聚合函数   | 6        |
| from     tableA   | 主表       | 1        |
| join       tableB | 连接       | 3        |
| on                | 连接条件   | 2        |
| where             | 过滤条件   | 4        |
| group by          | 分组       | 5        |
| having            | having条件 | 7        |
| order by          | 排序       | 10       |
| limit             | 分页       | 11       |

 **查询语句都是从from开始执行的，在执行过程中，每个步骤都会为下一个步骤生成一个虚拟表，这个虚拟表将作为下一个执行步骤的输入**。 

1. 首先对from子句中的前两个表执行一个笛卡尔乘积，此时生成虚拟表 vt1（选择相对小的表做基础表）。 

2. 接下来便是应用on筛选器，on 中的逻辑表达式将应用到 vt1 中的各个行，筛选出满足on逻辑表达式的行，生成虚拟表 vt2 。

3. 如果是outer join 那么这一步就将添加外部行，left outer jion 就把左表在第二步中过滤的添加进来，如果是right outer join 那么就将右表在第二步中过滤掉的行添加进来，这样生成虚拟表 vt3 。

4. 如果 from 子句中的表数目多余两个表，那么就将vt3和第三个表连接从而计算笛卡尔乘积，生成虚拟表，该过程就是一个重复1-3的步骤，最终得到一个新的虚拟表 vt3。 

5. 应用where筛选器，对上一步生产的虚拟表引用where筛选器，生成虚拟表vt4。

   注意where与on的区别：先执行on，后执行where；on是建立关联关系在生成临时表时候执行，where是在临时表生成后对数据进行筛选的。

6. group by 子句将中的唯一的值组合成为一组，得到虚拟表vt5。如果应用了group by，那么后面的所有步骤都只能得到的vt5的列或者是聚合函数（count、sum、avg等）。原因在于最终的结果集中只为每个组包含一行。这一点请牢记。 
7. 应用avg或者sum选项，为vt5生成超组，生成vt6. 
8. 应用having筛选器，生成vt7。having筛选器是第一个也是为唯一一个应用到已分组数据的筛选器。 
9. 处理select子句。将vt7中的在select中出现的列筛选出来。生成vt8. 
10. 应用distinct子句，对vt8进行去重，生成vt9。
11. 应用order by子句。按照order_by_condition排序vt9，此时返回的一个游标，而不是虚拟表。
12. 应用limit选项。生成vt10返回结果给请求者即用户。 

# 二、MySQL的执行顺序 

1、SELECT语句定义 
一个完整的SELECT语句包含可选的几个子句。SELECT语句的定义如下： 

``` sql
<SELECT clause> 
[<FROM clause>] 
[<WHERE clause>] 
[<GROUP BY clause>] 
[<HAVING clause>] 
[<ORDER BY clause>] 
[<LIMIT clause>] 
SELECT子句是必选的，其它子句如WHERE子句、GROUP BY子句等是可选的。 
一个SELECT语句中，子句的顺序是固定的。例如GROUP BY子句不会位于WHERE子句的前面。 
```

2、SELECT语句执行顺序 
SELECT语句中子句的执行顺序与SELECT语句中子句的输入顺序是不一样的，所以并不是从SELECT子句开始执行的，而是按照下面的顺序执行： 

开始->FROM子句->WHERE子句->GROUP BY子句->HAVING子句->SELECT子句->ORDER BY子句->LIMIT子句->最终结果 

每个子句执行后都会产生一个中间结果，供接下来的子句使用，如果不存在某个子句，就跳过 
对比了一下，MySQL和sql执行顺序基本是一样的, 标准顺序的 SQL 语句为: 

```sql
select empName, max(salary) as maxSalary 

from t_emp 

where empName is not null 

group by empName 

having max(salary) > 2000 

order by maxSalary
```

 在上面的示例中 SQL 语句的执行顺序如下: 

 (1). 首先执行 FROM 子句, 从 t_emp 表组装数据源的数据 

 (2). 执行 WHERE 子句, 筛选 t_emp 表中所有数据不为 NULL 的数据 

 (3). 执行 GROUP BY 子句, 把 t_emp 表按 empName 列进行分组

​		(注：这一步开始才可以使用select中的别名，他返回一个游标，而不是一个表，所以在where中不可以使用select中的别名，而having却可以使用)

 (4). 计算 max() 聚集函数, 按  maxSalary 求出总薪酬中最大的一些数值 

 (5). 执行 HAVING 子句, 筛选员工的总薪酬大于 2000的. 

 (7). 执行 ORDER BY 子句, 把最后的结果按 maxSalary 进行排序. 

参考：https://blog.csdn.net/u014044812/article/details/51004754?spm=1001.2014.3001.5506