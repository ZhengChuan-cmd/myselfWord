# 2.sql优化

### 1.基础查询

```
1.SQL执行顺序
	select,from,where,group by,having,order by,limit/fetch
	执行顺序：from->where->group by->having->select->order by->limit
2.基本子句用法
	select:去重 distinct,常量和表达式，列别名 AS(可省略)
	where:条件运算符=、<>	、>、<、between、in、like（%和_）、is null。逻辑运算符：AND、OR、NOT
	order by:asc（默认）、desc。可按多列、表达式、别名排序
	limit:limit offset，count 或 limit count OFFSET offset(MySQL/PosgreSQL);SQL Server用TOP,Oracle 用rownum或fetch first
3.聚合函数与分组
	聚合函数：count、sum、avg、max、min
	group by:分组字段可以不在selectz中（但通常包含）
	having:对分组后的结果进行筛选，例如:having count(*)>1
4.常见陷阱与易错点
	where:不能使用聚合函数，必须having
	count(*)包含 null 行，count(column)不包含Null
	Null参与任何比较结果为unknown,判断必须用is null 或 is not null
	like 匹配时注意通配符位置：‘abc%’可用索引，‘%abc’不能
	分页排序时若排序字段不唯一，需加第二排序键保证结果稳定
5.复习建议
	动手写：在本地或在线 SQL 练习平台（如 LeetCode、牛客网）完成 10+ 道简单/中等难度的查询题。
	理解执行顺序：能解释下面错误的原因：
	sql
	SELECT dept_id, AVG(salary) AS avg_sal
	FROM employee
	WHERE avg_sal > 5000  -- 错误：WHERE 中不能使用聚合别名
	GROUP BY dept_id;
	区分 IN 和 EXISTS：基础查询一般不深究性能差异，但要知道 IN 适合子查询结果集小，EXISTS 适合外表大且子查询关联时。
```

### 2.多表连接

```
1.连接类型
	内连接 inner join  仅仅返回两个表中匹配的行
	左外连接 left join 左表全部行，右表无匹配时补NULL
	右外连接 right join 右表全部行，左表误匹配补NULL
	全外连接 full join 左右表全部行，对方无匹配补NULL
	交叉连接 cross join 笛卡尔积，行数=左表行数*后表行数
	自连接 from t1 a join t1 b 同一张表连接自身，必须使用别名

2.连接条件 ON VS where
	ON：指定表间匹配逻辑。多余inner join ，ON 和 where 效果相同；对于outer join ，on在连接时生效，不匹配时的行仍保留
	where：在连接完成后对结果集整体过滤，若在left join 的where中加入右表过滤条件，会过滤掉右表为null的行，将左连接变成内连接
	原则：外连接时，右表的过滤条件应放 on;对左表的过滤放where（或全放where也可，因为左表无NULL 问题）

3.表别名
	简化书写，子连接时必须使用别名以区分同一张表的不同实例
	
4.执行计划中的连接顺序
	数据库优化器会决定驱动表和连接顺序。可使用explain查询type(eq_ref、ref、all等）和Extra
	一般原则：用小结果集驱动大结果集，被驱动表的连接列应有索引
	
5.子连接的应用场景
	查询层级关系（员工上级、父类父子）
	查询重复值（同一表内比较字段）
	查询满足特定行间关系的数据（如工资高于经理的员工）

问题:
1. 如何优化多表连接的性能？
	确保连接列有索引（特别是被驱动表的连接列）。
	小表驱动大表（优化器通常自动选择，但可用 STRAIGHT_JOIN 强制顺序）。
	只 SELECT 需要的列，避免 SELECT *。
	对 WHERE 条件使用索引。
	考虑是否可以用 EXISTS 替代 DISTINCT JOIN。

2. EXPLAIN 输出中的 type 字段哪些连接类型性能较好？
答：性能从高到低：system > const > eq_ref > ref > range > index > ALL。ALL 表示全表扫描，应避免；eq_ref 表示使用主键/唯一索引连接，性能很好。
```

### 3.子查询

```
1.子查询的分类
	标量子查询 返回单个值（一行一列） where salary > (select avg（salary） from employees)
	行子查询 返回一行多列 where （dept,title） = (select dept,title from ... limit 1)
	表子查询 返回多行多列 from （select ...） as alias
	列子查询 返回一列多行 where id in (select user_id from orders)

2.子查询位置
	select 子句：必须是标量子查询
	from 子句：表子查询（派生表），必须带别名
	where/having:可使用标量、行、列子查询，配合= 、IN、EXSITS、ANY/SOME 、ALL等运算符
	join条件：可将子查询作为虚拟表参与连接
	
3.相关子查询vs非相关子查询
	非相关子查询：独立于外层查询，只执行一次，结果供外层使用
	相关子查询：子查询的执行依赖于外层查询的当前行，每行可执行一下（性能较差）
	
4.in 与 EXISTS 的区别
	IN:适合子查询结果集较小；IN列表会先被物化
	EXISTS:适合外表大、子查询关联条件较强；通常改为半连接优化
	NOT IN陷阱：如果子查询结果包含NULL,NOT IN 会返回空结果.应该改为not exists 或处理 NULL

5.ANY/SOME/ALL
	>ANY(...)等价于大于子查询中的最小值
	>ALL(...)等价于大于子查询中的最大值
```

### 4.CTE（共用表表达式）

```
1.基本语法
	with cte_name(column_list) AS(
		select ...
	)
	select * from cte_name;

2.CTE的特点
	临时命名结果集：仅在单个sql语句执行期间存在
	可被多次引用：在同一条语句中可多次使用同一个ETC
	可递归：with recursice 支持自引用，用于树形/图遍历
	提高可读性：将复杂查询分解为有意义的块，替代嵌套子查询
	
3.递归CTE结构
	with recursive cte_name AS(
		select ...
		union ALL
		select ... cte_name where ...
	)
	select * from cte_name
	递归终止条件：必须在递归查询中用where限定，否则死循环
	限制递归深度：MySQL通过cte_max_recursion_depth,PostgreSQL默认可无限
	
4.CTE vs 子查询

```

### 5.窗口函数

```sql
1.基本语法
	<函数> over (
		[partition by 列1 ，列2，...]  --分区（分组）
    	[order by 列3，列4 ...]		--排序
        [rows/range between .. and ...]--窗口框架
    )

2.三大组成部分
	partition by :将数据分成独立的分区，函数在各个分区内计算
	order by:定义每个分区内的排序顺序，影响排序名类函数以及框架的默认行为
	窗口框架；控制计算的行范围（仅对聚合类函数有意义，如 sum、avg）.常见框架：
		rows between unbounded and current row (默认到当前行)
		rows between 1 perceding and 1 following(前后各一行)
		range 按值范围（较少用）

3.窗口函数分类
	排名函数：row_number()、rank()、dens_rank()、ntile(n)
		用途：生成序号、排名、分桶
	偏移函数：LAG(expr，offest,default)、LEAD(expr,offset,default)
		用途：访问同一分区内前后行的值
	首尾函数：first_value(expr)、last_value(expr)
		用途：分区内第一行/最后一行值
	聚合函数：sum()、avg()、count()、max()、min()
		用途：与聚合相同、但保留每行，可叠加框架
		
4.排名函数区别
	row_number()：连续不重合序号（1，2，3，4...），同值随机给不同序号
	rank():同值同排名，但是会跳过后续排名（1，2，3，4...)
	dense_rank():同值同排名，不跳过（1，2，3，4...）
	ntile(n):将行均匀分配到n个桶中，返回桶编号
	
5.窗口函数 vs group by 聚合
	行数变化     	每组输出一行			 输出所有原始行
	可与明细列混用	  仅能包含分组列或聚合	可任意列+窗口结果
	典型场景		汇总报告			  累计、排名、环比、移动平均

6.框架默认行为
	只有 order by 时，默认框架 range between unbounded perceding and current row （在mysql中等效于 rows）
	无 order by 时，框架为整个分区（range between unbounded perceding and unbounded following）
	
7.问题：
	简述窗口函数与普通聚合函数的区别。
	答：普通聚合函数配合 GROUP BY 会将多行压缩成一行，丢失明细数据。窗口函数在每一行上计算聚合结果，同时保留所有原始行。例如计算每个部门的平均工资，AVG(salary) OVER (PARTITION BY dept_id) 会在每个员工旁边显示部门平均工资，而不像 GROUP BY 那样只输出部门汇总行。

	什么时候使用 LAG 和 LEAD？
	答：需要对比当前行与前一行或后一行的数据时。例如计算环比增长率：(sales - LAG(sales,1) OVER (ORDER BY month)) / LAG(sales,1) OVER (ORDER BY month)；或查询每个用户上一次登录时间。

	窗口函数中的 ROWS 和 RANGE 有什么区别？
	答：ROWS 按物理行偏移，如 ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING 指前1行、当前行、后1行。RANGE 按逻辑值范围，如 RANGE BETWEEN 1 PRECEDING AND 1 FOLLOWING 指当前行的值 ±1 范围内的所有行（值相同时可能多行）。RANGE 需要 ORDER BY 列值具有明确等差意义。

8.复习建议
	手写三种排名函数的区别：用简单数据集验证 ROW_NUMBER、RANK、DENSE_RANK。
	掌握框架写法：尤其 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 用于累计求和。	
	练习常见模式：分组 Top N、累计汇总、同比环比、去重保留最新记录。
	推荐平台：LeetCode 窗口函数专题（例如 “Department Top Three Salaries”、“Consecutive Numbers”）。
```

### 6.索引与性能优化

```
1.索引类型与数据结构
	B+Tree索引(最常用，InnoDB默认)：所有数据都存储在叶子节点，叶子节点用双向链表链接，支持范围查询和顺序扫描
	Hash索引：等值查询快，不支持范围查询，Memory引擎支持
	全文索引：MATCH AGAINST,用于文本搜索
	聚簇索引：表数据按照主键顺序存放，叶子节点存储完成行数据（InooDB）
	非聚簇索引（二级索引）：叶子节点存储主键值，需回表

2.索引使用原则
	最左前缀原则：符合索引(a,b,c)能支持a、a,b、a,b,c的查询条件时；跳过a则无法使用索引
	索引下推（ICP）:存储引擎层用索引列提前过滤，减少回表次数
	覆盖索引：索引包含查询所需全部字段，无需回表（Extra：Using index）
	索引条件下推：在索引遍历过程中直接判断where 条件

3.索引失效场景
	对索引列使用函数或者表达式：where year(date) = 2023 ->改成 date between '2023-01-01' and '2023-12-31'
	隐式类型转换：where phone = 123 (phone 为 varchar)->字符串转数字，索引失效
	使用 like '%abc' 或 like '_abc' (前导通配符)
	OR 连接的查询条件，若有一侧不是索引列
	！= 或 not in(特定情况下失效)
	复合索引不遵守最左前缀
	优化器认为全表扫描更快（如 小表）
 
 4.执行计划（explain） 关键字段
 	type			访问类型：count > eq_ref > ref > range > index > all
 	possible_keys	可能使用索引
 	key				实际使用索引
 	key_lin			使用索引的长度（可推断使用了符合索引的那些列）
 	rows			预估扫描的行数
 	Extra			using index(覆盖索引)，using where（存储索引层+服务器过滤）、using indexing coditing(索引下					  推)、using temporary(用了临时表)、using filesort(文件排序，需要优化)
 	
 5.常见优化手段
 	避免使用 select *，只取必要列（方便索引覆盖）
 	分页优化：limit 10000000,10 -> 使用 id>last_id limit 10或延迟关联
 	关联查询：小表驱动大表，被驱动表关联列建立索引
 	count(*)优化：mysql 中 count(*)走非聚簇索引(如果存在)比主键更高效
 	避免复杂表达式和函数操作索引
 	合理设计索引：高频查询条件、区分度数高的列靠左，避免重复索引
 
 6.索引维护与监控
 	重复索引和冗余索引（如（a）和（a，b）中（a）荣冗余）
 	使用 show index from table 查看索引
 	删除未使用的索引（通过 sys.schema_unused_indexes 或慢查询日志分析）
 	
 7.复习建议
 	理解 B+Tree 结构：能解释为什么适合范围查询。
	熟练使用 EXPLAIN：分析典型 SQL 的执行计划，关注 key、rows、Extra。
	背诵失效场景：考试或面试中快速列举 5 条以上。
	练习常见模式：分页优化、多表连接优化、子查询优化。
	推荐资源：MySQL 官方手册、极客时间《MySQL 实战 45 讲》、LeetCode 数据库优化类题目。
```

### 7.事务与隔离级别

```sql
1.事务的ACID特性
	原子性：事务中的操作要么全部成功，要不全部失败
		实现手段：undo log （回滚日志）
	一致性：事务前后数据库完整性约束不被破坏
		实现手段：由原子性、隔离性等公共保证
	隔离性：并发事务之间互不干扰
		实现手段：锁机制 + MVCC
	持久性：事务提交后数据永久保存
		实现手段：redo log（重做日志）

2.事务的控制语句
	begin / start transaction; --开启事务
	commit;					   --提交
	rollback;				   --回滚
	savepoint sp;			   --设置保存点
	rollback to savepoint sp;  --回滚到保存点

3.并发事务带来的问题
	脏读：读到其他事务未提交的数据（可能被回滚）
	不可重复读：同一事务内两次读取同一条记录，结果不同（被其他事务修改并提交）
	幻读：同一事务内两次查询范围数据，结果行数不同（其他事务插入或删除了行）

4.sql标准定义的三个隔离级别
	读未提交（read cuncommitted）:不加锁，直接读取最新版本
	读已提交（read committed）:普通select 使用快照读（每条语句独立快照）；锁定读使用行锁
	可重复读（repeatable read）:事务第一次select建立快照；锁定读加行锁+间隙锁
	串行化（serializable):所有select隐式加锁（读锁），强行串行执行
	mysql innoDB 默认隔离级别式可重复读，并通过 间隙锁 解决了幻读问题（但仅限 select ,,, for update）等当前读；快照读			仍可能看到旧快照，不是严格意义的幻读

5.问题：
	隔离级别中，哪些级别会产生幻读？InnoDB 如何解决？
	答：SQL 标准中，可重复读级别允许幻读。但 InnoDB 在可重复读级别下，通过 间隙锁（Gap Lock） 防止幻读。例如 SELECT * FROM t WHERE id > 10 FOR UPDATE 会锁定 id > 10 的间隙，其他事务无法插入新行，从而避免当前读的幻读。对于快照读（普通 SELECT），依然可能看到不同快照，但通常业务场景可接受。
	
6.复习建议：
	理解 ACID 以及 undo/redo log 的作用。
	背诵三种并发问题 及隔离级别的关系表格。
	区分不同隔离级别下 MVCC 的行为：ReadView 生成时机。
	熟悉 MySQL 特有实现：间隙锁、可重复读下幻读的解决方式。
	动手实验：开启两个终端，验证脏读、不可重复读、幻读现象。
	结合锁知识：了解行锁、间隙锁、Next-Key Lock 与隔离级别的联系。
```

### 8.MVCC（多版本并发控制）

```
原理：
	InnDB为每行记录增加两个隐藏列：DB_TRX_ID（最后修改改行的事务ID）、DB_ROLL_PTR（回滚指针，指向undo log中的旧版本）
	
ReadView:
	事务开始时（或首次selects时）生成快照，包含活跃事务ID列表

可见性规划:
	当前事务只能看到DB_TRX_ID小于当前事务ID且不在活跃列表中的版本

不同隔离级别下的ReadView生成时机：
	Read committed:每条select语句都重新生成 readView
	repeateable read:事务内每一条select生成readview,后续复用（实现可重复读）

问题：
	简述 MVCC 如何实现可重复读。
	答：在可重复读隔离级别下，事务开始时（或第一次 SELECT）创建一个 ReadView，记录当前活跃事务 ID。之后该事务内的所有快照读（普通 SELECT）都复用这个 ReadView，根据可见性规则读取数据版本：只显示版本号小于当前事务 ID 且不在活跃列表中的已提交事务的修改。这样即使其他事务提交了修改，当前事务依然看到第一次查询时的快照，实现可重复读。
```

### 9.锁机制

```
行锁：lock in share mode（共享锁）/for update（排他锁）
表锁：lock tables
意向锁：InnDB 自动添加，用于表锁和行锁的兼容性判断
间隙锁（Gap Lock）：锁定一个范围，防止幻读（仅在可重复读以及以上级别使用）
Next-Key Lock = 行锁 + 间隙锁

问题：
	什么是死锁？如何避免？
	答：两个或以上事务相互持有对方需要的锁，导致无限等待。避免方法：1）按固定顺序访问表和行；2）使用 try lock 或超时；3）尽可能缩短事务；4）使用低隔离级别；5）InnoDB 会自动检测死锁并回滚其中一个事务。
	
	什么是“当前读”和“快照读”？
	答：快照读：普通的 SELECT（不加锁），基于 undo 读取历史版本，不阻塞其他事务。
	当前读：SELECT ... FOR UPDATE、SELECT ... LOCK IN SHARE MODE、UPDATE、DELETE、INSERT，读取最新提交版本，并加锁。
```

### 10数据库设计

```
1.三大范式
	1NF:属性不可再分（原子性）
	2NF:满足1NF+没有部分依赖（非主属性完全依赖主键）
	3NF：满足2NF+没有传递依赖
	BCNF（3NF的加强版）：所有非平凡函数的左侧都包含候选键（通常3NF已满足）

2.反范式设计
	定义：适当增加冗余字段，减少关联查询，提交可读性
	典型场景：
		订单表冗余商品名称（避免每次join商品表）
		统计字段（如 comment_count)冗余到主表
	代价：更新维护成本增加（需用触发器或应用层保持一致性）

3.主键，外键与索引设计
	主键：
		推荐使用代理主键（自增id或UUID），避免业务主键变更
	外键：
		保证数据完整性，但会带来性能开销（插入/更新需检查）。大型系统中常由应用层维护引用关系
	索引：针对高频查询条件创建索引，但是索引过多会拖慢写入

4.字段类型选择
	整数				 INT/BIGINT			 避免：VARCHAR存数字
	短字符串（固定长度）	CHAR				避免：VARCHAR会有额外开销
	可变长度字符串		  VARCHAR			  避免：text 影响性能
	小数				 DECIMAL（财务）	   避免：FLOAT/DOUBLE 有精度问题
	日期时间			DATETIME/TIMESTAMP	 避免：存字符串
	布尔值				TINYINT(1)/BOOLEAN	  避免：CHAR(1)
	
进阶优化：
	1. 如何对订单表进行分库分表设计？
	答：按用户 ID 哈希分库，按订单日期或用户 ID 取模分表。同时要考虑跨分片查询（如按时间范围查询所有订单）的解决方案（Elasticsearch 或冗余表）。

	2. 什么情况下适合使用 JSON 字段？
	答：存储不固定模式的属性（如用户自定义扩展字段）、避免频繁更改表结构。但不要滥用，JSON 字段无法有效索引（除虚拟列），查询性能较差。
	
复习建议：
	熟练范式分解：能识别 2NF、3NF 并拆分表。
	区分规范与反规范：知道何时牺牲范式换性能。
	画 ER 图：常用实体关系（1:1, 1:N, N:N）的数据库实现。
	设计练习：设计博客、电商、学生选课等系统表结构。
	理解分库分表：基本概念及适用场景。
```

### 11.DDL 和 DML

```
DDL(Data Definition Language,数据定义语句)：
	作用：定义、修改或删除数据库对象（如表、索引、视图、约束等）的结构
	常用命令：
		create:创建表、索引、视图、数据库等
		alter:修改表结构（增加/删除列，修改类型、添加约束等）
		DROP:删除表、索引、视图等
		TRUNCATE:清空表中所有的数据（保留结构，类似delete但更快，属于DDL 的一种
		RENAME:重命名对象
	特点：
		执行后自动提交（隐式commit），无法回滚（truncate 在部分数据库可回滚，如PostgreSQL，但MySql中不可回滚）
		操作的数据库的元数据（表结构），而非数据本身

DML(Data Manipulation Language,数据库操作语句)
	作用：对表中的数据进行增，删，改，查操作（注意：严格定义的DML通常只包含增删改，select常被归于DQL,但广义上也算DML）
	常用命令：
		select：查询数据（有时单独称为DQL）
		insert:插入数据
		update:更新数据
		delete:删除数据
	特点：
		可以显式控制事务：begin/commit/rollback
		未提交的DML操作可以通过rollback撤销
		操作的是数据内容，不改变表结构

问题：
1.TRUNCATE 和 DELETE 的区别？
	答：
	TRUNCATE 是 DDL，DELETE 是 DML。
	TRUNCATE 不能带 WHERE，DELETE 可以。
	TRUNCATE 重置自增列计数器（MySQL），DELETE 不重置。
	TRUNCATE 通常不可回滚（MySQL），DELETE 在事务内可回滚。
	TRUNCATE 速度更快（不记录逐行日志，只记录页释放）。

2. DDL 为什么通常不能回滚？
	答：因为 DDL 操作会导致数据库元数据变更，且许多数据库的设计中 DDL 会隐式提交当前事务，并无法被包含在用户事务中。但某些数据库（如 PostgreSQL）支持部分 DDL 回滚（如 CREATE TABLE 在事务中可回滚）。

3. 如何在同一个事务中同时使用 DDL 和 DML？
	答：不同的数据库支持程度不同。MySQL 中 DDL 会隐式提交事务，因此不能与 DML 混合在同一个事务中（除非使用特定存储引擎如 InnoDB 且版本高于 8.0？实际上 MySQL 8.0 中 DDL 依然自动提交）。Oracle、PostgreSQL 支持在事务中执行 DDL（原子性）。面试时可以说“大部分数据库会将 DDL 视为自动提交，但 PostgreSQL 等支持事务性 DDL”。

4. ALTER 语句会对表加什么锁？
	答：在 MySQL InnoDB 中，许多 ALTER 操作（如添加列）需要元数据锁（MDL），可能阻塞并发的 DML。但可以通过 ALGORITHM=INPLACE 或 ALGORITHM=INSTANT（MySQL 8.0+ 部分操作）减少锁影响。
```

