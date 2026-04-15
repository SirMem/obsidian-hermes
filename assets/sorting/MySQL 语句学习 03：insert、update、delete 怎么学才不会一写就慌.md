# MySQL 语句学习 03：insert、update、delete 怎么学才不会一写就慌

createTime: 2026-04-15 14:57:40 +0800
updateTime: 2026-04-15 14:57:40 +0800
status: sorting
tags: [MySQL, SQL, insert, update, delete, DML, 数据变更, sorting]

## 一、这一篇为什么要学 insert、update、delete

前一篇我们学的是 `select`，它解决的是：
- 我怎么把数据查出来
- 我怎么筛、排、分页

但真正做业务时，只会查还不够。

你还要经常面对另外三类动作：
- `insert`：新增一条数据
- `update`：修改已有数据
- `delete`：删除不再需要的数据

如果说 `select` 是“看数据”，
那 `insert / update / delete` 更像是“改数据”。

而很多初学者一写这三类语句就慌，不是因为语法真的很难，
而是因为它们有一个共同特点：

“它们会直接改表里的数据，所以你天然会怕写错。”

所以这一篇的重点不是死记语法，
而是先建立一个稳定框架：

- 这条语句到底在改什么
- 它会影响哪些行
- 我怎么避免一不小心改太多

你先记一句总纲：

“写变更语句时，最重要的不是把语法敲出来，而是先想清楚：我要动哪几行数据。”

## 二、先把三类语句放到一张图里理解

你可以把表想成一个名单。

- `insert`：往名单里加新的人
- `update`：把名单里已有人的信息改掉
- `delete`：把名单里某些人从名单中移除

对应到 SQL：

```sql
insert into user (...列...) values (...值...);
update user set ... where ...;
delete from user where ...;
```

它们最大的差别，不只是关键字不同，
而是“作用对象”不同：

- `insert` 主要关心：我要新增什么内容
- `update` 主要关心：我要修改哪些行、改成什么
- `delete` 主要关心：我要删除哪些行

所以你会发现：

- `insert` 的核心是“给值”
- `update` 和 `delete` 的核心是“先用 where 找对行，再动手”

这也是为什么后两者更容易让人紧张。
因为一旦 `where` 没写对，影响范围就可能失控。

## 三、我们继续沿用同一张 user 表

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

假设当前表里有这些数据：

```text
id | name  | age | city      | balance | create_time
1  | Alice | 18  | Hangzhou  | 1000.00 | 2026-04-15 10:00:00
2  | Bob   | 20  | Shanghai  |  500.00 | 2026-04-15 10:10:00
3  | Carol | 22  | Hangzhou  | 2000.00 | 2026-04-15 10:20:00
```

后面的例子都围绕这张表来。

## 四、insert：往表里新增一行数据

### 1. 最基础写法

```sql
insert into user (name, age, city, balance)
values ('David', 19, 'Beijing', 300.00);
```

你可以这样理解：
- `insert into user`：往 `user` 这张表里插入数据
- `(name, age, city, balance)`：我要给哪些列赋值
- `values (...)`：这些列分别对应什么值

执行后，大概会新增一行：

```text
4 | David | 19 | Beijing | 300.00 | 2026-04-15 xx:xx:xx
```

这里 `id` 没写，是因为它通常会自增；
`create_time` 没写，是因为它有默认值。

### 2. insert 时最容易乱的点是什么

不是语法本身，
而是“列顺序和对应值是否一一匹配”。

比如：

```sql
insert into user (name, age, city, balance)
values ('David', 19, 'Beijing', 300.00);
```

这里的对应关系是：
- name → David
- age → 19
- city → Beijing
- balance → 300.00

所以初学时一定养成习惯：

“写 insert 时，尽量显式写列名，不要偷懒只写 values。”

因为这样：
- 可读性更强
- 不容易因为列顺序理解错而写乱
- 以后表结构变化时更稳

### 3. 一次插入多行

```sql
insert into user (name, age, city, balance)
values
  ('Eva', 21, 'Hangzhou', 800.00),
  ('Frank', 23, 'Shanghai', 1200.00);
```

意思是一次新增两行。

这在导入测试数据、批量创建初始数据时很常见。

### 4. insert 的最小检查思路

每次写完先问自己三件事：
- 我插入的是哪张表
- 我给了哪些列
- 值和列是不是一一对应

## 五、update：修改已有数据，真正的风险点在 where

### 1. 最基础写法

```sql
update user
set balance = 1500.00
where id = 1;
```

意思是：
- 在 `user` 表里
- 找到 `id = 1` 这一行
- 把它的 `balance` 改成 `1500.00`

如果你把它翻译成自然语言，就是：

“把 Alice 的余额改成 1500。”

### 2. update 的骨架要这样记

```sql
update 表名
set 列 = 新值
where 条件;
```

其中：
- `update`：说明我要改数据
- `set`：说明我要把哪些列改成什么
- `where`：说明我要改哪几行

你会发现，`update` 真正决定危险程度的，不是 `set`，而是 `where`。

### 3. 修改多个列

```sql
update user
set city = 'Suzhou', balance = 1800.00
where id = 3;
```

意思是：
- 找到 `id = 3` 的用户
- 同时修改城市和余额

### 4. 没有 where 会发生什么

```sql
update user
set balance = 0;
```

这条语句语法上可能完全正确，
但它的意思是：

“把整张 user 表里所有行的 balance 都改成 0。”

这就是为什么很多人第一次写 `update` 会慌。
因为你怕的不是不会写，
你怕的是“写对语法，却改错范围”。

所以请先建立一个强习惯：

“写 update 时，脑子里先问：我的 where 到底圈中了哪些行？”

### 5. update 前最稳的做法

如果条件稍微复杂一点，
先用同样的 `where` 写一遍 `select`：

```sql
select * from user where id = 1;
```

确认选中的确实是你想改的那一行，
再写：

```sql
update user set balance = 1500.00 where id = 1;
```

这是非常重要的实战习惯。

你可以把它记成一句话：

“先 select 验证范围，再 update 真正修改。”

## 六、delete：删除数据时，更要先想清楚范围

### 1. 最基础写法

```sql
delete from user
where id = 2;
```

意思是：
- 从 `user` 表里
- 删除 `id = 2` 这一行数据

也就是把 Bob 这条记录删掉。

### 2. delete 的骨架

```sql
delete from 表名
where 条件;
```

和 `update` 很像，
它的核心也在 `where`。

### 3. 没有 where 的 delete 有多危险

```sql
delete from user;
```

这句的意思不是“删一条”，
而是：

“把 user 表里的所有行都删掉。”

所以初学阶段你一定要把 `delete` 和 `where` 绑在一起理解：

“delete 不可怕，可怕的是没有明确范围的 delete。”

### 4. delete 前同样可以先 select

比如你准备删除所有杭州用户：

先看范围：

```sql
select * from user where city = 'Hangzhou';
```

确认没问题后再删：

```sql
delete from user where city = 'Hangzhou';
```

同样是那句话：

“先 select，看准；再 delete，动手。”

## 七、为什么 update / delete 一定要和 where 绑在一起学

因为从业务角度看，
这两类语句本质上都不是“改语法”，
而是“对一批行执行动作”。

- `update`：对符合条件的行做修改
- `delete`：对符合条件的行做删除

所以 `where` 在这里不是附属件，
而是决定影响范围的核心控制器。

你可以把它理解成一个筛子：
- 先用 `where` 筛出目标行
- 再对这些行执行 update 或 delete

这和上一节的 `select ... where ...` 其实是同一条主线。

只是：
- `select` 是“筛出来看看”
- `update/delete` 是“筛出来再动它”

## 八、初学阶段最该建立的 4 个防慌习惯

### 1. 永远先想‘我要动哪几行’

不要一上来就敲关键字。

你先用自然语言说清楚：
- 我要新增一条什么数据
- 我要修改哪一行
- 我要删除哪些行

如果自然语言都说不清，SQL 往往也容易写乱。

### 2. 写 update/delete 时，默认先写 where

很多人是先写：

```sql
update user set ...
```

然后再想条件。

更稳的思路是：
- 先知道目标范围
- 再去补 `set` 或 `delete`

也就是心智顺序上把 `where` 放到前面。

### 3. 条件不确定时，先写 select 验证

这是你从“会写 SQL”走向“敢在线上谨慎写 SQL”的关键一步。

模板就是：

```sql
select * from user where ...;
update user set ... where ...;
```

或者：

```sql
select * from user where ...;
delete from user where ...;
```

### 4. 小心‘语法正确，但业务错误’

比如：

```sql
update user set balance = 0 where city = 'Hangzhou';
```

它完全可能语法正确，数据库也会顺利执行。
但业务上也许你本来只是想改某一个人，结果改了整座城市的人。

所以你以后判断一条 SQL，不要只问：
- 这句能不能执行

还要问：
- 这句会影响几行
- 这是不是我真正想影响的范围

## 九、给你一个最小实战串联

假设现在有个需求：

“新增一个用户 Gina；然后把 Gina 的余额改成 900；最后如果录错了，再把 Gina 删除。”

你可以这样写：

### 第一步：新增

```sql
insert into user (name, age, city, balance)
values ('Gina', 20, 'Nanjing', 600.00);
```

### 第二步：先查一下 Gina

```sql
select * from user where name = 'Gina';
```

### 第三步：修改 Gina 的余额

```sql
update user
set balance = 900.00
where name = 'Gina';
```

### 第四步：如果确认这条数据不需要了，再删除

```sql
delete from user
where name = 'Gina';
```

这里你会明显感觉到：

- `insert` 更像“加进去”
- `update` 更像“找到它，再改它”
- `delete` 更像“找到它，再删它”

所以真正让人稳下来的，不是背更多语法，
而是建立“先定位对象，再执行动作”的思维。

## 十、你现在先不用急着深挖的点

这一篇先不展开这些问题：
- 主键冲突怎么办
- 批量 update / delete 的性能问题
- 事务里做 update / delete 会有什么锁影响
- `truncate` 和 `delete` 的区别
- `insert into ... select ...` 这种进阶写法

这些后面都可以继续学。

你当前最重要的是先把基础动作打稳：
- 新增：知道往哪里加、加什么
- 修改：知道改什么，更知道改谁
- 删除：知道删什么，更知道删谁

## 十一、这一篇你至少要记住的 5 句话

1. `insert` 是新增，核心是“列和值要对齐”。
2. `update` 是修改，核心不是 `set`，而是 `where`。 
3. `delete` 是删除，核心也不是关键字，而是影响范围。 
4. 条件稍复杂时，先用 `select` 验证范围，再真的改。 
5. SQL 不只是语法题，更是“我到底会影响哪批数据”的范围题。

## 十二、下一步怎么学更顺

学完这一篇，下一步很适合继续补这两个方向：

1. `where` 条件再进一层
- `and` / `or`
- `in`
- `between`
- `like`
- `is null`

2. update / delete 为什么会让人紧张
- 不只是语法问题
- 还和事务、锁、误操作风险有关

如果你愿意，下一篇我可以继续给你讲：

《MySQL 语句学习 04：where 条件怎么写，才能真正把“找哪批数据”这件事学明白》
