# MySQL 语句学习 04：where 条件怎么写，才能真正把“找哪批数据”这件事学明白

createTime: 2026-04-15 15:04:18 +0800
updateTime: 2026-04-15 15:04:18 +0800
status: sorting
tags: [MySQL, SQL, where, and, or, in, between, like, is null, 条件查询, sorting]

## 一、这一篇为什么必须单独学 where

前面你已经见过很多次 `where` 了：
- `select ... where ...`
- `update ... where ...`
- `delete ... where ...`

但如果只把它当成“一个条件子句”，你还是会容易写乱。

因为 `where` 真正解决的不是语法问题，
而是这件事：

“这条 SQL 到底在面对哪一批数据？”

你可以这么理解：
- 没有 `where`，你面对的通常是整张表
- 有了 `where`，你面对的是整张表中的一部分行

所以 `where` 的本质，不是给 SQL 增加一点修饰，
而是在定义“作用范围”。

这也是为什么：
- 在 `select` 里，`where` 决定你看哪些行
- 在 `update` 里，`where` 决定你改哪些行
- 在 `delete` 里，`where` 决定你删哪些行

你先记住这一句：

“学 where，不是在背条件写法，而是在学会精确圈定目标数据。”

## 二、先把 where 的核心动作想明白

假设还是这张 `user` 表：

```sql
create table user (
  id bigint primary key auto_increment,
  name varchar(50) not null,
  age int,
  city varchar(50),
  balance decimal(10, 2),
  create_time datetime default current_timestamp
);
```

假设当前有这些数据：

```text
id | name  | age | city      | balance | create_time
1  | Alice | 18  | Hangzhou  | 1000.00 | 2026-04-15 10:00:00
2  | Bob   | 20  | Shanghai  |  500.00 | 2026-04-15 10:10:00
3  | Carol | 22  | Hangzhou  | 2000.00 | 2026-04-15 10:20:00
4  | David | 17  | Beijing   |  300.00 | 2026-04-15 10:30:00
5  | Eva   | 19  | Hangzhou  |  800.00 | 2026-04-15 10:40:00
6  | Frank | NULL| Shanghai  | 1200.00 | 2026-04-15 10:50:00
```

你现在不要急着看各种写法，
先只看一个最核心问题：

如果我说：
- 找杭州用户
- 找年龄大于等于 18 的用户
- 找余额在 500 到 1000 之间的用户

其实我都在做同一件事：

“从整张表里，筛出符合某个条件的那批行。”

这就是 `where` 的唯一主线。

## 三、where 最基础的 3 类条件

### 1. 等值条件：某列必须等于某个值

```sql
select * from user where id = 1;
select * from user where city = 'Hangzhou';
```

意思分别是：
- 找 `id` 等于 1 的行
- 找 `city` 等于 Hangzhou 的行

这是最基础、也最常见的一类条件。

最小业务体感：
- 按主键查一个人
- 按城市筛一批人
- 按状态筛订单

你可以把它理解成：

“这一列，必须精确满足这个值。”

### 2. 比较条件：大于、小于、区间判断

```sql
select * from user where age >= 18;
select * from user where balance > 1000;
select * from user where age < 20;
```

意思是：
- 找成年用户
- 找余额大于 1000 的用户
- 找年龄小于 20 的用户

这是在做“范围筛选”。

最小心智模型：
- `=`：精确找
- `>` / `<` / `>=` / `<=`：按大小范围找

### 3. 不等于条件：排除某些值

```sql
select * from user where city != 'Hangzhou';
```

或者有些地方也会写：

```sql
select * from user where city <> 'Hangzhou';
```

意思都是：
- 找城市不是 Hangzhou 的用户

初学时你先知道：
- `!=` 和 `<>` 都常见
- 它们表达的是“不等于”

但业务上要小心：

“不是某个值”通常会筛出比较大的一批数据，
所以你要时刻有范围意识。

## 四、多个条件一起写：and 和 or 才是 where 真正开始容易乱的地方

当条件只有一个时，大家通常不慌。
真正开始乱，往往是从多个条件同时出现开始。

### 1. and：必须同时满足

```sql
select * from user
where city = 'Hangzhou' and age >= 18;
```

意思是：
- 城市必须是 Hangzhou
- 年龄还必须大于等于 18
- 两个条件都成立，才会被选中

按我们的样例数据，大概会得到：
- Alice
- Carol
- Eva

你可以把 `and` 理解成：

“收紧范围。”

条件越多，而且都用 `and`，
通常筛出来的数据越少。

### 2. or：满足其中一个就行

```sql
select * from user
where city = 'Hangzhou' or city = 'Shanghai';
```

意思是：
- 在杭州，或者在上海
- 只要满足其中一个条件，就会被选中

你可以把 `or` 理解成：

“放宽范围。”

因为它允许更多行通过筛选。

### 3. 为什么 and / or 特别容易写错

因为它们不是在改语法风格，
而是在改“筛选集合”的范围。

你一定要有这种感觉：
- `and` 更像交集
- `or` 更像并集

最小体感例子：

```sql
select * from user
where city = 'Hangzhou' and age < 19;
```

这句是：
- 在杭州
- 且年龄小于 19

结果可能只有 Alice。

而：

```sql
select * from user
where city = 'Hangzhou' or age < 19;
```

这句是：
- 在杭州，或者年龄小于 19

那范围就明显更大，
因为 David 虽然不在杭州，但年龄小于 19，也会被选中。

## 五、为什么一旦 and 和 or 混用，你就要特别小心

比如：

```sql
select * from user
where city = 'Hangzhou' or city = 'Shanghai' and age >= 20;
```

很多初学者看到这句会下意识以为它表示：
- 在杭州或上海
- 并且年龄大于等于 20

但实际理解如果不加括号，很容易和你脑子里想的不一样。

所以初学阶段，你先建立一个最稳的原则：

“只要 and 和 or 混用，就主动加括号。”

比如你真正想表达：

“城市是杭州或上海，并且年龄至少 20 岁”

那就明确写成：

```sql
select * from user
where (city = 'Hangzhou' or city = 'Shanghai')
  and age >= 20;
```

如果你想表达的是另一种意思：

“在杭州，或者（在上海且年龄至少 20）”

那就写成：

```sql
select * from user
where city = 'Hangzhou'
   or (city = 'Shanghai' and age >= 20);
```

你会发现：

真正难的不是 `and` / `or` 关键字本身，
而是你脑子里想要的是哪一批数据。

所以这一节的核心训练永远不是背优先级，
而是把自然语言条件翻译清楚，再用括号写明白。

## 六、in：当你要匹配多个候选值时，比一串 or 更顺

比如你想找杭州、上海、北京的用户。

当然你可以写：

```sql
select * from user
where city = 'Hangzhou'
   or city = 'Shanghai'
   or city = 'Beijing';
```

但更常见、更清楚的写法是：

```sql
select * from user
where city in ('Hangzhou', 'Shanghai', 'Beijing');
```

`in` 的意思可以先理解成：

“这一列的值，是否属于这个候选集合。”

最小体感：
- `city in (...)`：城市在这个名单里
- `id in (...)`：id 在这些编号里

比如：

```sql
select * from user where id in (1, 3, 5);
```

意思是只找 id 为 1、3、5 的用户。

所以：
- 少量多个等值候选，用 `in` 很顺
- 它本质上是在表达“多个可能值中的任意一个”

## 七、between：当你要表达一个连续区间时

比如你想找年龄在 18 到 20 之间的用户：

```sql
select * from user
where age between 18 and 20;
```

可以先把它理解成：

“age 在 18 到 20 这个区间内。”

对初学阶段来说，你先建立这个体感就够了：
- 它适合表达连续区间
- 比同时写两个条件更像一句完整的话

比如余额区间：

```sql
select * from user
where balance between 500 and 1000;
```

这就比下面这种写法更紧凑：

```sql
select * from user
where balance >= 500 and balance <= 1000;
```

两者表达的业务意思很接近。

所以你可以把 `between` 看成：
- 一个“区间版”的 where 条件写法

## 八、like：当你不是精确匹配，而是想按模式去找

有时候你不是要找“刚好等于某个值”，
而是要找“长得像某种模式”的数据。

比如名字里包含字母 `a`：

```sql
select * from user
where name like '%a%';
```

你现在先不用把细节学得很满，
先抓住最常见的直觉：

- `%` 可以先理解成“任意一串字符”

所以：

```sql
where name like 'A%'
```

表示：
- 名字以 A 开头

```sql
where name like '%a%'
```

表示：
- 名字里包含 a

```sql
where name like '%e'
```

表示：
- 名字以 e 结尾

这类条件在模糊搜索、关键词查找里很常见。

但你也要记住一点：

`like` 往往已经不是“精确定位”，
而是在做模式匹配。

所以它和 `=` 的使用场景很不一样。

## 九、is null：判断空值时，不要想当然地写成 = null

这是一个很容易踩的坑。

比如我们样例里，Frank 的 `age` 是 `NULL`。

如果你想找年龄为空的用户，应该写：

```sql
select * from user where age is null;
```

如果想找年龄不为空的用户：

```sql
select * from user where age is not null;
```

初学阶段你先记死一条规则：

“判断空值，用 is null / is not null，不要写 = null。”

你可以先把它当成 SQL 在处理空值时的一条特殊规则。

## 十、把这些 where 条件放到一个总表里理解

你暂时可以这样记：

- `=`：精确等于某值
- `!=` / `<>`：不等于某值
- `>` `<` `>=` `<=`：比较大小
- `and`：同时满足
- `or`：满足其一
- `in (...)`：在候选集合里
- `between a and b`：在连续区间里
- `like`：按模式匹配
- `is null`：判断空值

如果你能把它们都翻译成自然语言，
你的 where 就已经开始稳了。

## 十一、真正学会 where，不是背这些写法，而是会“翻译需求”

比如下面这些业务话，你要慢慢练习把它翻成 where：

### 1. 找杭州的成年人

业务话：
- 在杭州
- 并且年龄大于等于 18

SQL：

```sql
select * from user
where city = 'Hangzhou' and age >= 18;
```

### 2. 找上海或北京的用户

业务话：
- 城市是上海或北京

SQL：

```sql
select * from user
where city in ('Shanghai', 'Beijing');
```

### 3. 找余额在 500 到 1500 之间的用户

SQL：

```sql
select * from user
where balance between 500 and 1500;
```

### 4. 找名字以 A 开头的人

SQL：

```sql
select * from user
where name like 'A%';
```

### 5. 找年龄还没填的用户

SQL：

```sql
select * from user
where age is null;
```

所以你会发现：

where 学得好不好，本质上不是看你背了多少关键词，
而是看你能不能把一句业务需求，拆成准确条件。

## 十二、为什么 where 会直接影响性能，但你现在先建立直觉就够了

前面在 `select` 那一篇我们提过：
- `where id = 1`
- `where city = 'Hangzhou'`

它们虽然都能查到数据，
但数据库找数据的代价可能完全不同。

你现在先不要急着深挖执行计划，
先建立两个直觉：

1. `where` 不只是决定“查不查得到”，还决定“数据库怎么找”。
2. 越能精确缩小范围的条件，通常越值得重点关注。

后面你再把它和索引、explain 连起来，就会更完整。

## 十三、初学 where 最容易犯的 5 类错

### 1. 把业务话想模糊了

比如你本来想找：
- 杭州或上海
- 并且年龄至少 20

结果你自己脑子里没想清，SQL 当然也容易写歪。

### 2. and / or 混用时不加括号

这是特别常见的坑。

初学阶段最稳的做法：
- 混用就加括号
- 用括号把你真正想要的分组关系写清楚

### 3. 把 like 当成 = 来用

`=` 是精确匹配，
`like` 是模式匹配，
它们不是一回事。

### 4. 判断空值写成 = null

记住：
- `is null`
- `is not null`

### 5. 写 update / delete 时，忘了 where 的范围含义

你一旦明白 `where` 是“圈目标行”，
就不会只把它当作一个附加条件了。

## 十四、给你一个防慌模板：先说人话，再翻 SQL

以后你每次写 `where`，可以先在脑子里走这 3 步：

### 第一步：先说人话

比如：
- 我要找杭州、且年龄至少 18 的用户

### 第二步：拆成最小条件

- city = 'Hangzhou'
- age >= 18

### 第三步：再决定用什么连接

- 两个都要满足，所以用 `and`

最终写成：

```sql
where city = 'Hangzhou' and age >= 18
```

如果你总是先用人话想范围，
再翻译成 where，稳定性会高很多。

## 十五、这一篇你至少要记住的 6 句话

1. `where` 的本质是圈定“哪一批数据”。
2. `=` 是精确匹配，比较符号是范围匹配。 
3. `and` 是收紧范围，`or` 是放宽范围。 
4. `and` / `or` 混用时，初学阶段主动加括号。 
5. 多个候选值常用 `in`，连续区间常用 `between`。 
6. 判断空值要用 `is null` / `is not null`。

## 十六、下一步怎么学更顺

学到这里，你已经可以写出一批很基础但很有用的查询条件了。

下一步很适合继续补两条线：

1. `where` 放进真实查询中继续练
- `select + where + order by + limit`
- 面试里常见的数据筛选题

2. `where` 放进数据变更中继续练
- `update ... where ...`
- `delete ... where ...`
- 为什么这时 where 会更敏感

如果你愿意，下一篇我可以继续带你学：

《MySQL 语句学习 05：select + where + order by + limit，怎么拼成一条真正像业务查询的 SQL》
