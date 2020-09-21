### 数据库基础

1. 能够**唯一表示一行数据的若干列**叫做**主键**，可以是几列，但是其组合必须**唯一表示**某一行；



### 数据库的使用--以MYSQL为例

1. **所有命令的最后都带分号“；”，如果没有使用BINARY则不区分大小写；**
2. 如果要使用数据库crashcourse，使用命令**USE crashcourse**。这里的USE叫做关键字，这个crashcourse是一个数据库，**里面可以有很多列表。**
3. 如果想查看当前有哪些数据库，可以使用**SHOW DATABASES**,这里面的mysql和information_schema是系统内部使用的数据库；
4. 当你进入了一个数据库的时候，可以使用**SHOW TABLES**查看当前这个数据库有**哪些表**；
5. 在4里我们查看了有哪些表，可以使用**SHOW COLUMNS FROM xxx**，这个xxx指的是4中看到的某一个表，就可以查看这个表；这个命令等价于**DESCRIBE xxx**；



### 排序检索数据

1. 如果只想从数据库中抽出若干列，要使用**SELECT x1，x2，x3 FROM y**，这里的x1、x2、x3都是y这个表中的列，用**逗号**分隔。

2. 如果要查看某一列中**不同的数据**有哪些，可以使用**SELECT DISTINCT x FROM y**；

3. 如果要**限制只显示几行**可以使用**SELECT x FROM y LIMIT 5**，表示只显示**前5行**；如果使用**SELECT x FROM y LIMIT a，b**表示查看从**a行开始的后面b行**；

4. 在mysql中可以使用类似于类的成员函数的方式来指定查看的对象，例如**SELECT y.x FROM t.y**表示查看**t数据库内的y表的x列**，这里的x和y分别都限定了；

5. **如果要对查询的数据进行排序**，可以使用**SELECT x1，x2，x3 FROM y ORDER BY x1，x2**，表示显示y表中的x1、x2、x3列，**先根据x1的值升序排序，然后根据x2的值升序排序**；如果要逆序排序，则使用**SELECT x1，x2，x3 FROM y ORDER BY x1 DESC，x2**，注意这里的DESC**只对x1有效**，如果x2也要逆序，要分别加DESC；

6. 注意，这里的执行的顺序由严格规定：

   ①SELECT x FROM y；

   ②ORDER BY x；

   ③LIMIT 1；



### 过滤

1. 如果需要查找某一列里面的特定数据，可以使用**SELECT x FROM y WHERE x = a**，表示在y表中的x列里面查找**值等于a**的行；
2. WHERE的操作符包括**=、！=（跟<>作用相同，不等于）、<、<=、>、>=、BETWEEN**，这里的BETWEEN表示在两个值之间，可以这么用**SELECT x FROM y WHERE x BETWEEN a AND b**，表示查找在a和b之间的值；
3. 列表里面的**空值NULL**并不等于0或者空格，如果需要查找，可以使用**SELECT x FROM y WHERE x IS NULL**；



### 更高级的过滤

1. 在WHERE中如果组合多条语句，与python中一样，使用AND、OR、IN和NOT，例如

   **WHERE （x OR y） AND z**，这里面是支持利用括号来提高优先度的，如果不使用，默认**AND优先处理**；

2. 可以使用IN来完成类似于OR的功能，例如

   **WHERE a = x OR a = y**等价于**WHERE a IN （x，y）**，表示查找a的值等于x或者y的行；此外，使用NOT IN可以取反，表示不再x和y这个范围内的行；



### 使用通配符

1. LIKE操作符

   1. 使用**百分号%**可以匹配**任意字符出现的任意次数**。例如

      **WHERE x LIKE ‘jet%’**匹配（这里的单引号表示**字符串**），可以匹配jetback 1000等；

      **WHERE x LIKE ‘%jet%’**，匹配字符串中间包含jet的子串，**但是效率很低**；

      **WHERE x LIKE ‘s%e’**，匹配的s..e，但是如果后面有空格或者别的就会失败；

   2. 使用**下划线_**但是只匹配一个字符。例如

      **WHERE x LIKE ‘_ A‘**可以匹配1 A或者2 A；



### 正则表达式

REGEXP表示regular expression

1. 如果要查找**值内含有字符串s**，可以使用

   **WHERE x REGEXP ’s‘**，匹配asb等，相当于LIKE ’%s%‘；

2. 使用点号’.‘匹配一个字符，相当于LIKE里面的下划线_；

3. **LIKE和REGEXP的区别**：

   LIKE匹配的是**整个值**，如果有一个值为asb，你只能用%s%来匹配；但是如果使用REGEXP可以简单地使用s来匹配，因为REGEXP是匹配**值内某部分**；同时REGEXP也可以限定某字符只出现在值的头部或者尾部；

4. 在REGEXP中使用OR的逻辑，例如

   **WHERE x REGEXP ’a|b‘**等价于**WHERE x REGEXP '[ab]'**；

   这里的中括号，表示匹配内部的**其中一个字符**，就相当于很多个或了。如果要匹配**除了中括号以外的字符，可以在内部加^，**即**WHERE x REGEXP ’\[^ab\]‘**。

5. 对于数字或者字母，可以使用横杠-来表示一个范围，例如：

   **WHERE x REGEXP ’[1-5]‘**表示1~5其中一个数字，类似的还有\[a-z\]\[A-Z\];

   1. 类似的，如果要匹配特殊字符，需要进行**转义**。sql的转义字符是两个反斜杠\\\\，**MYSQL自身解释一个，正则表达式解释另一个。**此外还有一些特别的**\\\\f（换页）、\\\\n（换行）、\\\\r（回车）、\\\\t（制表）、\\\\v（纵向制表）。**

   2. 对于一些特别的字符，可以使用**预定义的字符类**：

      \[：alnum：\]：字母和数字

      \[：alpha：\]：字母

      \[：blank：\]：空格和制表

      \[：digit：\]：数字

      \[：space：\]：任意空白字符，上面的那些\\\\n或者别的都是

      \[：upper：\]：大写字母

      \[：lower：\]：小写字母

      \[：xdigit：\]：十六进制数字

      \[：print：\]：任意可打印字符

6. 如果需要匹配多个字符，可以使用**匹配元字符**

   \*：0个或若干个

   +：1个或者若干个

   ?：0个或者1个

   {n}：n个

   {n,}：大于等于n个

   {n,m}：n到m个，m小于等于255

   例如：

   ①**WHERE x REGEXP '\\\\([0-9] sticks?\\\\)'**可以匹配到0 sticks或者0 stick，注意，那个？是指他的**前一个字符出现0或者1次**；

   ②**WHERE x REGEXP ’\[\[：digit：\]\]{4}‘**可以匹配4个数字1000或者2000等，注意，这里的digit有两层中括号，而且这个4也只是值前面的这个字符集出现四次；

7. **定位元字符**

   ^：文本的开始

   $：文本的末尾

   \[\[：<：\]\]：词的开始

   \[\[：>：\]\]：词的结尾

   例如：

   **WHERE x REGEXP '^\[0-9\\\\.\]'**匹配开头是0~9其中一个数字或者.的比如.5 a或者5 b，这里的^限定了这个字符必须在文本的头部；

   **WHERE x REGEXP ’[a-z]$‘**可以匹配 bba或者ccz这种结尾是a~z的值；



### 创建计算字段

1. 拼接字段Concat（），例如：

   ①**SELECT Concat（x1，’（‘，x2，’）‘） FROM y ORDER BY x1**表示从y中取出x1和x2这两列，然后以x1升序排序，最后再以x1（x2）这样的形式显示；

   ②此外，如果要去掉x1或者x2前面的空格可以用LTrim（），后面的空格RTrim（），全部空格Trim（），就会变成**SELECT Concat（RTrim（x1），’（‘，RTrim（x2），’）‘） FROM y ORDER BY x1**；

   ③因为①②中生成的新列没有名字，客户端无法引用他，因此需要给这一个列取一个名字，

   **SELECT Concat（x1，’（‘，x2，’）‘） AS test FROM y ORDER BY x1**，使用了AS，那么这一列就会叫做test列；

2. 执行**算术运算**，例如：

   ①支持加减乘除运算，可以使用括号改变运算顺序，

   **SELECT x1，x2，x3，x2\*x3 AS x4 **

   **FROM y **

   **ORDER BY x1**

   将会显示四列，然后第四列的名字叫做x4，其值为x2和x3的乘积；

   ②如何测试函数输出呢？可以使用SELECT NOW()直接返回当前时间，使用SELECT Trim（’ abc ‘）返回’abc‘，所以使用SELECT很方便；



### 使用数据处理函数

一般会支持**字符串处理函数、日期和时间处理函数、数值运算函数、DBMS特殊信息函数。**

1. 字符串处理函数

   **SELECT x1，Upper（x1） AS x2 **

   **FROM y **

   **ORDER BY x1**

   将显示两列，第二列名为x2，值为x1的大写。

   常用的函数有：

   Left（）：返回字符串左边的字符

   Length（）：返回字符串的长度

   Locate（）：查找一个子串

   Lower（）：转换为小写

   LTrim（）：去掉左边的空格

   Right（）：返回右边的字符

   RTrim（）：去掉右边的空格

   \*Soundex（）：返回串的SOUNDEX值

   SubString（）：返回子串的字符

   Upper（）：转换为大写

   其中，Soundex有点难理解，用来将字符串转换为某种发音。这里放出一个例子：

   **SELECT x1，x2 **

   **FROM y **

   **WHERE Soundex（x1） = Soundex（’A B‘）**

   这里会返回x1的发音类似于A B的行，比较神奇，用于我们数据输入错误了，只能依靠发音来查找的情况

2. 日期和时间处理函数，常用的函数有：

   **AddDate（）增加一个日期（天、周等）**

   AddTime（）增加一个时间（时、分等）

   CurDate（）返回当前日期

   CurTime（）返回当前时间

   Date（）返回日期时间的日期部分

   **DateDiff（）计算两个日期之间的差**

   Date_Add（）日期运算

   Date_Format（）返回一个格式化的日期或者时间串

   Day（）返回日期的天数部分

   DayOfWeek（）返回一个日期对应是星期几

   Hour（）返回时间的小时部分

   Minute（）返回时间的分钟部分

   Month（）返回日期的月份

   **Now（）返回当前日期和时间**

   Second（）返回时间的秒部分

   Time（）返回日期的时间部分

   Year（）返回日期的年部分

   **注意，日期的格式为yyyy-mm-dd**，例如2005-09-01，这是默认的格式。

   存储日期的数据类型为datetime，存储的值可以只有日期或者时间，也可以两者都有，但是匹配的时候需要注意一下：

   ①

   **SELECT x **

   **FROM y **

   **WHERE x = '2005-09-01'**这种只能匹配只有日期没有时间的值，如果值带有时间就凉了；

   ②

   **SELECT x **

   **FROM y **

   **WHERE Date（x） = '2005-09-01'**这种就是没有问题的了，因为我们只比较日期；当然，如果想比较时间就需要用到Time（）；

   ③想要查找9月份的订单

   **SELECT x **

   **FROM y **

   **WHERE Year（x） = 2005 AND Month（x） = 9**

   虽然也可以使用**WHERE BETWEEN '2005-09-01' AND '2005-09-30'**,但是使用函数就不需要知道这一个月到底有多少天；

3. 数值处理函数，主要分为代数、三角和几何运算，常用的有：

   Abs（）绝对值

   Cos（）余弦

   Exp（）指数

   Mod（）余数

   Pi（）圆周率

   Rand（）返回一个随机数

   Sin（）正弦

   Sqrt（）平方根

   Tan（）正切

   除了Rand（）和Pi（）不需要参数以外，cos（）sin（）tan（）需要一个角度参数，其他函数需要一个数字参数；





### 汇总数据

用于我们不需要实际的数据值，只需要数据汇总的结果。例如表中的行数、行组的和、表列的最大值最小值和平均值等。

| 函数      |       说明       |
| --------- | :--------------: |
| AVG（）   | 返回某列的平均值 |
| MAX（）   | 返回某列的最大值 |
| MIN（）   | 返回某列的最小值 |
| SUM（）   |   返回某列的和   |
| COUNT（） |  返回某列的行数  |

①AVG只能获得某列的均值，可以使用where进行约束，但是必须给出列名，即select AVG（uid）AS avg_uid ；此外，**AVG自动忽略UNLL**；

②COUNT函数统计某列的行数，使用COUNT（\*）的时候**不会忽略NULL**，具体指出列名就会**忽略NULL**；

③MIN、MAX、SUM类似，指定某一列名的时候会**忽略NULL**；此外，SUM（a\*b）会统计a、b这两列乘积的和；

**DISTINCT关键字**

1. 如果要统计某一列非重复元素可以使用SELECT AVG（**DISTINCT** a） AS avg_a FROM tb WHERE uid=3，这时候，参与统计的a列不会有重复的数据；
2. 可以组合使用多个函数：SELECT COUNT（\*） AS cnt，AVG（a）AS a_avg FROM tb；



### 分组数据

我们在汇总数据的时候加了一个约束WHERE uid=1003，但是如果我们需要分别统计uid=1004、uid=1005的汇总数据怎么办？答案是分组，分组的关键字是**GROUP BY**。例如：

SELECT uid，COUNT（\*） AS total FROM tb GROUP BY uid

这一句表示**以uid分组，对每一个分组运行一次COUNT**，返回结果就是不重复的uid，同时统计了各自有多少行。

GROUP BY的一些重要规定：

1. GROUP BY必须在WHERE之后，ORDER BY之前；
2. 如果分组列中有NULL，所有的NULL为一组；
3. 除聚集语句外，所有SELECT的列都必须在GROUP BY语句出现；

**关键字HAVING**

我们在分组里面还需要过滤数据能使用WHERE吗？答案是不行的，**因为WHERE过滤行，HAVING过滤分组**，我们可以使用HAVING，使用方法不变。例如：

SELECT uid，COUNT（\*） AS orders FROM od GROUP BY uid HAVING COUNT（\*）>=2

就会以uid为分组，然后返回行数大于等于2的分组了；

可以混用WHERE和HAVING吗？答案是肯定的，例如我们只统计物品单价大于10块钱的行就可以：

SELECT uid，COUNT（\*）AS cnt FROM tb WHERE price >=10 GROUP BY uid HAVING COUNT（\*）>=2

就可以返回指定数据了。

**ORDER BY 和 GROUP BY**的区别？

ORDER BY输出是排序的，但是GROUP BY只是分了组，组内和组间不一定有序；但是这两个关键字不冲突，很多时候都是混合使用的。例如：

SELECT uid，SUM（quantity \* price） AS total FROM tb GROUP BY uid HAVING SUM（quantity \* price）>=500；这一句输出购物总价超过500块的所有uid及其购物总价；如果我们需要按照总价升序排序呢？

SELECT uid，SUM（quantity \* price） AS total FROM tb GROUP BY uid HAVING SUM（quantity \* price）>=500 ORDER BY **total**；这里的ORDER BY可以使用别名了。

再次总计一下所有SELECT字句的顺序：

1. SELECT
2. FROM
3. WHERE
4. GROUP BY
5. HAVING
6. ORDER BY
7. LIMIT



### 使用子查询

如果我们需要查询的数据需要两个表分别进行查询，就需要两个SELECT语句，这个时候可以使用子查询，即嵌套SELECT。例如：

SELECT proid FROM pro WHERE item=3；返回proid=1，proid=2；

SELECT uid FROM user WHERE proid IN （1，2）；

可以嵌套得到：

SELECT uid FROM user WHERE proid IN （SELECT proid FROM pro WHERE item=3）

这就是子查询。MySQL默认从内到外执行SELECT语句。请注意，**子查询返回的列数应该等于WHERE中限定的列数，可以是一列或者是多列。**

当两个表具有相同的列，即外键的时候，要使用限定符进行区分，例如：

SECLECT uid，uid_state，（SELECT COUNT（\*） FROM tb2 WHERE tb1.uid=tb2.uid）AS num FROM tb1 ORDER BY uid；这一句首先统计tb2里面tb2.uid与tb1.uid相同的uid的行数，然后将其作为一列展示。



### 联结表

**SQL最强大的功能之一就是能在数据检索查询的执行中联结表（join）。**

外键：外键是某个表中的一列，这一列是另外一个表的主键，定义了两个表之间的关系；通过主键和外键的定义，可以保证两个表中表示对应关系的列保持一致，而不允许插入另一个表不存在的值。

可伸缩性：能够适应不断增加的工作量却不失败的的特性；

如果需要用一个SELECT查询两个表中的数据，就需要用到联结，可以联结多个表返回一个输出。

**创建联结**：使用WHERE和FROM就可以创建联结：

SELECT uid，price，item FROM tb1，tb2 WHERE tb1.uid=tb2.uid ORDER BY uid，tiem；

这里假设uid，price位于tb1，item位于tb2，我们分别从tb1和tb2抽取数据，过滤使用**完全限定列名**tb1.uid=tb2.uid。

**笛卡尔积**：由没有联结关系的表关系返回的结果为笛卡尔积。**检索出的行数是第一个表中的行数乘以第二个表中的行数。**例如上面的语句没有WHERE进行过滤，就会出现很多数据，但是这些数据都是无用的，所以不要忘记WHERE过滤。

**内部联结**：基于两个表某一列的相等测试的联结，就是内部联结。因此，上面的语句可以改为：

SELECT uid，price，item FROM tb1 **INNER JOIN** tb2 **ON** tb1.uid=tb2.uid；

注意，这里使用了INNER JOIN 和ON，这样的标记与WHERE一样，但是提示了这里是内联结。



### 创建高级联结

**自联结**：在**同一个表中执行两次SELECT**如何解决冲突的问题，请看：

SELECT p1.uid，p1.pro FROM tb1 AS p1，tb1 AS p2 WHERE p1.price=p2.price ；这就是自联结，**使用别名来解决同一个表内的两次SELECT可能引起的冲突；**

**自然联结：**在联结中可能有相同的列在不同的表导致返回相同列多次，为了解决这个问题，引入了自然联结。自然联结情况下，对其中一个表使用SELECT \*通配符，其他表明确指出的情况下完成。例如：

SELECT t1.\* t2.uid FROM tb1 AS t1，tb2 AS t2 WHERE t1.uid=t2.uid；

**外部联结**：如果两个表中有关联的列，但是有一些列是没有关联的。例如客户列表有的客户从未下单，所以订单列表没有这位客户的数据，但是我们仍然需要他的数据就要使用外部联结。例如：

SELECT tb1.uid，tb2.pro FROM tb1 **LEFT OUTER JOIN** tb2 ON tb1.uid=tb2.uid；

注意这里是两个关键字，一个是LEFT，表示左边的tb1包含那些无关联的行，如果是右边要使用RIGHT；OUTER JOIN就类似于INNER JOIN，表示外部联结，这里就不赘述了。

**使用带聚集函数的联结**，例如：

SELECT tb1.uid，COUNT（tb2.num）AS cnt FROM tb1 INNER JOIN tb2 ON tb1.uid=tb2.uid GROUP BY tb1.uid；这一句，**首先按照tb1.uid进行分组，然后对每一组分别统计每一组的行数；**

**TIPS：**使用联结需要注意限定条件，否则会返回笛卡尔积；



### 组合查询

将多个SELECT查询组合成一个结果进行返回，称为并或者复合查询。使用**关键字UNION**连接多个SELECT语句实现组合查询，例如：

SELECT uid，pro，price FROM tb1 WHERE price<=5 

UNION 

SELECT uid，pro，price FROM tb1 WHERE uid IN （1，2）；

这相当与使用OR，但是在不同的表上面进行查询，才能发挥UNION最大的威力。

**UNION的限制：①n个SELECT需要n-1个UINON进行连接；②SELECT的列必须相同，但是顺序不一定相同；③列数据类型必须至少可以隐式转化达到相同；**

1. 使用UNION默认会将**重复的行消除**，如果不想消除重复行，需要使用UNION ALL；
2. 可以对UNION后的数据排序，由于只有一个返回结果，所以**ORDER BY**必须放在**最后一个SELECT**之后；



### 全文本搜索

为了进行全文本搜索，**必须索引被搜索的列。**

1. MyISAM**支持**全文本搜索，但是InnoDB**不支持**全文本搜索；

2. 要支持全文本搜索，在建表的时候就需要表明全文本搜索的那一列，使用关键字**FULLTEXT**：

   CREATE productnotes

   （

   ​	note_id    int      NOT NULL    AUTO_INCREMENT,

   ​	prob_id    char（10）    NOT NULL,

   ​	note_date    datetime     NOT NULL,

   ​	not_text    text    NULL,

   ​	PRIMARY KEY（note_id），

   ​	**FULLTEXT（note_text）**,

   ）ENGINE=MyISAM；

   这里的FULLTEXT会对note_text这一个列建立一个索引，在对数据库的操作时，索引会随之更新。

3. 进行全文本搜索需要使用**Match（）指定被搜索的列，Against（）指定要使用的搜索表达式：**

   SELECT note_text

   FROM productnotes

   WHERE Match（note_text） Against（‘rabbit’）；

   这里的**Match必须使用建立了FULLTEXT索引的列上面，如果FULLTEXT由多个列组成，则应该按照相同顺序全部列出；**这句查询note_text中包含rabbit这个文本的所有结果；其实跟模糊搜索LIKE ‘%rabbit%’返回的结果一样；**如果不在BINARY模式下，搜索是不区分大小写的**；

4. 那么全文本搜索与LIKE的区别是：①全文本搜索会按优先级返回，例如第一个结果的匹配在第3个单词，第二个结果的匹配在第20个单词；而LIKE搜索则不一定是有序的。②全文本搜索使用了索引，速度比LIKE模糊搜索更快；

5. 全文本搜索如何进行排序和过滤？使用：

   SELECT note_text,Match（note_text） Against（‘rabbit’）AS rank

   FROM productnotes

   可以看到，rank列中，不包含rabbit的行为0，这个就是用于过滤的；包含rabbit的行根据rabbit所在的位置和出现次数有一个权重，权重值越大就越靠前（靠近开头，出现次数越多权重越大）；

**查询扩展**

如果我不仅要精确查找全文本，还需要查找不包含该文本的可能有用的数据，就需要查询扩展。

1. 在查询扩展中，mysql对**数据和索引进行两遍扫描来完成搜索；**首先，使用全文本搜索精确查找数据；然后从搜索结果中找一些可能有用的文本加入全文本搜索的过滤器中；最后，再次进行全文本搜索，这次的搜索范围比第一次大好多；使用方法：

   SELECT note_text

   FROM productnotes

   WHERE Match（note_text）Against（'rabbit' **WITH QUERY EXPANSION**）

2. **布尔（boolean）文本搜索**：这是一种可以设定包含某些文本就返回，包含某些文本就不返回，设定各个文本的优先级等功能的**非常缓慢的操作，但是不需要索引。**例如：

   SELECT note_text

   FROM productnotes

   WHERE Match（note_text）Against（**'rabbit -rope\*' IN BOOLEAN MODE**）；

   这一行表示匹配rabbit但是去除以rope开头的结果；

   | 布尔操作符 | 说明                 |
   | :--------: | -------------------- |
   |     +      | 必须包含             |
   |     -      | 必须不包含           |
   |     >      | 提高优先级           |
   |     <      | 降低优先级           |
   |     ()     | 组成子表达式         |
   |     ~      | 取消游戏级           |
   |     ""     | 指定字符串而不是单词 |
   |     *      | 通配符               |

   SELECT note_text

   FROM productnotes

   WHERE Match（note_text）Against（**'rabbit apple -rope\* +water +“pine apple”' IN BOOLEAN MODE**）；表示rabbit、apple中包含其中一个，必须不以rope开头，必须包含water，必须包含字符串pine apple；

**全文本搜索的一些规则：**

1. 仅在MyISAM中可用；
2. 如果数据词少于3个，自动忽略；
3. mysql内部有一个非用词列表，这些词在全文本搜索的时候自动忽略；
4. 出现频率超过百分之50的词自动加入**非用词列表**，因此表中只有3行以下的数据不会返回结果，布尔文本搜索没有这个百分之50限制；
5. 忽略单引号，don‘t等价于dont；



### 插入数据

使用关键字**INSERT**插入一行、一行中的一部分、多行或者某些查询结果。

1. **插入一行**：最简单的插入，语句如下：

   INSERT INTO productnotes VALUES（3，’kiki‘，20，NULL）；

   一般不会产生输出；顺序不能乱，没有值要用NULL填充，前提是这一列支持NULL值；如果设置了AUTO_INCREMENT的不需要填写；

2. 显示指出数据会更安全，而且还向后兼容，即使列增加了也可以正常工作；此外，还可以减少VALUES的数目，根据前面的列表而定，如果没有指定就要全部VALUES显式表明了；

   INSERT INTO productnotes（uid，name，price，state） VALUES（3，’kiki‘，20，NULL）；

3. 要省略某些列，这些列必须满足：①支持NULL值；②在表定义中给出默认值；

4. 由于INSERT有可能很耗时，导致阻塞了SELECT，可以使用关键字**LOW_PRIORITY**在INSERT和INTO之间降低优先级：INSERT **LOW_PRIORITY** INTO productnotes（uid，name，price，state） VALUES（3，’kiki‘，20，NULL）；

5. **插入多行：**最简单的就是分号间隔多个INSERT，如果两个数据对应的列相同，可以合并：

   INSERT INTO productnotes（uid，name，price，state） VALUES（3，’kiki‘，20，NULL），（4，’yoyo‘，21，NULL）；单个INSERT比多个INSERT更快；

6. **插入查询结果：**直接插入查询的结果，使用关键字SELECT：

   INSERT INTO productnotes（uid，name，price，state）

   **SELECT id，n，p，st FROM tb1**；

   将SELECT的查询结果插入到productnotes中；这里的SELECT就当做正常的查询就好了，可以使用WHERE等过滤；



### 更新和删除数据

使用关键字**UPDATE**来更新数据。

1. 注意，一定要使用WHERE或其他手段限定范围，否则将更新所有行；
2. 最简单的UPDATE：**UPDATE** customers **SET** email=’xx@qq.com‘ **WHERE** uid=1002；
3. 更新多个列，只需要用一个SET，用逗号分隔需要更新的列；
4. 如果发生错误，所有操作都会回滚；但是可以使用IGNORE来忽略发生的错误，例如UPDATE IGNORE customers ...；
5. 如果要删除这个值，可以更新为NULL，但是这一列必须支持NULL；

使用关键字**DELETE**来删除数据。

1. DELETE是用于删除一行的，如果要删除一列，请使用UPDATEA；
2. 删除一行数据：**DELETE FROM** customers WHERE uid=1001；
3. 如果要删除删除所有行，使用关键字**TRUNCATE TABLE；**实现原理是删除原来的表并创建一个表；

MySQL没有撤销，所以更新删除之前可以先select检查一下；



### 创建和操纵表

1. 使用**CREATE TABLE**来创建一个表，必须给出表的名称，以及表列的名字和定义；下面是一个例子：

   CREATE TABLE customers

   （

   ​	cust_id    	  int     					NOT NULL    AUTO_INCREMENT,

   ​	cust_name	char（50）		NOT NULL,

   ​	cust_address	char（50）	NULL，

   ​	PRIMARY KEY （cust_id）

   ）ENGINE=InnoDB；

2. 上面的PRIMARY KEY就是主键，可以在**表名后面**加上IF NOT EXISTS表示该表不存在的时候才创建，因为表名必须是唯一的；

3. 标明NOT NULL即插入必须有数据，如果标明NULL则表示可以缺省（**默认是NULL的**）；

4. **主键**：在这个表中，必须可以唯一标识某一行，可以是一列或者多列，多列的时候必须这几列唯一标识某一行，使用PRIMARY KEY（l1,l2），此外，主键必然是NOT NULL的，建表后可以改，但是不能使用接受NULL的那一列；

5. 使用**AUTO_INCREMENT**的列在每次INSERT会自动+1，保证安全。但是每个表**只允许一列**使用AUTO_INCREMENT，并且该列必须被索引；

   那么我们如何知道最后一个值是多少呢？答案是使用**函数last_insert_id()返回最后一次INSERT产生的AUTO_INCREMENT得到的值**，使用SELECT last_insert_id（）就可以用于后续操作了；

   1. **默认值**：插入某行时没有给定值的时候，如果有默认值也可以成功。使用关键字**DEFAULT**设置默认值，在建表的时候设置：**price	int	NOT NULL	DAFAULT	1，**这样使用；默认值不能是一个函数；
   2. 用ENGINE可以指定使用的引擎，一个数据库可以有多个不同引擎的表，**如果使用外键，则不允许跨引擎**；

6. **更新表**：

   1. **插入一列：**使用**ALTER TABLE tb1 ADD price	int；**

   2. **删除一列**：使用**ALTER TABLE tb1 DROP COLUMN price；**

   3. **定义外键**：如果我要把tb1中的uid作为外键，与tb2中的id关联，可以：

      ALTER TABLE tb2

      **ADD CONSTRAINT** fk_tb1_tb2

      **FOREIGN KEY** （uid） **REFERENCES** tb1（uid）；

   4. **删除整个表**：直接DROP TABLE customers；

   5. **重命名一个表**：直接**RENAME** TABLE customers **TO** cus；用逗号分隔可以修改多个表；





### 创建索引

给表创建索引一般分成两种，第一种是建表的时候建立，第二种是建表之后建立：

1. 建表时建立：

   ```sql
   CREATE TABLE projectfile (
   	id INT AUTO_INCREMENT COMMENT '附件id',
   	fileuploadercode VARCHAR(128) COMMENT '附件上传者code',
   	projectid INT COMMENT '项目id;此列受project表中的id列约束',
   	filename VARCHAR (512) COMMENT '附件名',
   	fileurl VARCHAR (512) COMMENT '附件下载地址',
   	filesize BIGINT COMMENT '附件大小，单位Byte',
   	-- 主键本身也是一种索引（注:也可以在上面的创建字段时使该字段主键自增）
       PRIMARY KEY (id),
   	-- 主外键约束（注:project表中的id字段约束了此表中的projectid字段）
   	FOREIGN KEY (projectid) REFERENCES project (id),
   	-- 给projectid字段创建了唯一索引(注:也可以在上面的创建字段时使用unique来创建唯一索引)
   	UNIQUE INDEX (projectid),
   	-- 给fileuploadercode字段创建普通索引
   	INDEX (fileuploadercode)
   	-- 建立联合索引
       INDEX (fileuploadercode,projectid)
   	-- 指定使用INNODB存储引擎(该引擎支持事务)、utf8字符编码
   ) ENGINE = INNODB DEFAULT CHARSET = utf8 COMMENT '项目附件表';
   ```

   解析：

   ①UNIQUE:可选。表示索引为唯一性索引。
   ②FULLTEXT:可选。表示索引为全文索引。
   ③SPATIAL:可选。表示索引为空间索引。
   ④INDEX和KEY:用于指定字段为索引，两者选择其中之一就可以了，作用是    一样的。
   ⑤索引名:可选。给创建的索引取一个新名称。
   ⑥字段名1:指定索引对应的字段的名称，该字段必须是前面定义好的字段。
   ⑦长度:可选。指索引的长度，必须是字符串类型才可以使用。
   ⑧ASC:可选。表示升序排列。
   ⑨DESC:可选。表示降序排列。

   这里最常用的就是主键PRIMARY KEY了，一般都需要设定；然后需要注意一下唯一索引UNIQUE INDEX，也是很好用的一个索引；第三个就是联合索引了，例如INDEX (fileuploadercode,projectid)就是一个联合索引；

2. 建表之后的索引操作

   1. 新增一个索引，一般使用ALTER或者CREATE

      ```sql
      -- 假设建表时fileuploadercode字段没创建索引(注:同一个字段可以创建多个索引，但一般情况下意义不大)
      -- 给projectfile表中的fileuploadercode创建索引
      ALTER TABLE projectfile ADD UNIQUE INDEX (fileuploadercode);	
      -- 将id列设置为主键
      ALTER TABLE index_demo ADD PRIMARY KEY(id) ;
      -- 将id列设置为自增
      ALTER TABLE index_demo MODIFY id INT auto_increment;  
      -- 新增一个索引在fileuploadercode上
      CREATE UNIQUE INDEX (fileuploadercode) ON projectfile;
      ```

   2. 查看当前表中的有什么索引

      ```sql
      show index from 表名；
      ```

   3. 删除一个索引，使用ALTER或者DROP

      ```sql
      drop index fileuploadercode1 on projectfile;
      alter table projectfile drop index s2123;
      ```

   4. 查看一个sql语句使用了什么索引，就是在select的前面加一个explain，这里就不再赘述了；



### 使用视图

视图是虚拟的表，**不包含任何数据**，与包含数据的表不同，视图**只包含使用时动态检索数据的查询。**

视图的作用：

1. SQL语句的重用（因为使用视图可以保存某一次的SELECT结果）；
2. 隐藏复杂的细节，但是可以方便地使用；
3. 使用表的组成部分而不是整个表；
4. 保护数据。给予用户对应的访问权限；
5. 更改数据的格式和表示；

视图的特点：

1. 命名必须唯一，不能与别的视图或者表重名；
2. 可创建的视图数量没有限制；
3. 创建视图需要权限；
4. 视图可以嵌套；
5. **视图不能索引，也不能关联触发器和默认值**；

视图的使用：

1. 创建视图：CREATE VIEW v1；

2. 删除视图：DROP VIEW v1；

3. 封装一个复杂的SQL语句：

   CREATE VIEW pcs AS

   SELECT uid，uname

   FROM customers，orders

   WHERE customers.uid=orders.uid

   AND orders.price >=5；

   然后视图pcs就代替了这一句SELECT，可以直接用了：

   SELECT uname

   FROM pcs

   WHERE uid=1001；

4. 视图的另一个常见用途就是**格式化检索得到的数据。**

5. 虽然视图不含有任何数据，但是**使用UPDATE、INSERT、DELETE会作用到基表**，造成持久性的修改；但是一般的视图不允许这些操作，例如GROUP BY、联结、子查询、并等；



### 使用存储过程

存储过程就是一系列SQL语句，那不是事务吗？

优点：

1. 代码重用；
2. 比单纯的SQL更高效；
3. 封装了复杂的实现；

如何使用存储过程？

1. 使用关键字**CALL**，例如：

   CALL pcs（@uidmin，@uidmax）；

   使用名为pcs的存储过程，计算并返回uidmin和uidmax；

如何创建存储过程？

**CREATE PROCEDURE pcs（）**

**BEGIN**

​	SELECT AVG（price）AS avg_p

​	FROM item；

**END**

1. BEGIN和END来限定pcs这个存储过程的过程体；

2. 在BEGIN和END之间可以是一句或者若干句SELECT语句；

3. 如果需要使用，直接CALL pcs就ok了；

4. 删除存储过程：DROP PROCEDURE pcs IF EXISTS；（没有带括号,IF EXISTS只有pcs存在的时候才删除）

5. **使用存储过程返回结果**：

   CREATE PROCEDURE pcs

   （

   ​	**OUT**	pl	DECIMAL（8，2），--可以存储2位小数的8位数

   ​	**OUT**	ph	DECIMAL（8，2），

   ​	**OUT**	pa	DECIMAL（8，2），

   ）

   BEGIN

   ​	SELECT	Min（price）	**INTO**	pl	FROM	pcs；

   ​	SELECT	Max（price）	**INTO**	ph	FROM	pcs；

   ​	SELECT	Avg（price）	**INTO**	pa	FROM	pcs；

   END

   这里的OUT表示这个存储过程的输出，INTO表示将数据传递给后面的值；注意，视图不支持一个变量返回多个行或者列。

   如果我们要调用上面这个存储过程，应该使用：**这里的@表示一个变量**

   CALL PROCEDURE  pcs

   （

   ​	@pricelow，

   ​	@pricehigh，

   ​	@priceavg	

   ）；

   可以使用SELECT @pricelow，@pricehigh来查看结果；

   **存储过程需要传入参数的时候？**

   CALL	PROCEDURE	ord

   （

   ​	IN	onumber INT，

   ​	OUT	ototal	DECIMAL（8，2）

   ）

   BEGIN

   ​	SELECT	Sum（quantity\*price）

   ​	FROM	items

   ​	WHERE	order_num=onumber

   ​	INTO	ototal；

   END

   这里的IN表示我们需要一个数据，OUT表示我们会输出数据；

   如何使用呢？使用：

   CALL ord（1001，@total）；

6. 在前面添加“--”，表示后面的是注释；

7. 在**存储过程内**，使用DECLARE	total	DECIMAL（8，2）声明一个临时变量；

8. **查看存储过程：**使用SHOW    CREATE    PROCEDURE    pcs;为了查看准确详细命令，使用：

   SHOW	PROCEDURE	STATUS可以查看我们编写了多少存储过程了；



### 使用游标

游标是一个存储在MySQL服务器上的数据集查询，不是一条SELECT语句，而是被该语句检索出来的结果集，可以根据需要使用游标来逐行操作查询结果。**游标只能用于存储过程**。

1. 创建游标：

   CREATE	PROCEDURE	pcs（）	

   BEGIN

   ​	**DECLARE**	ord	**CURSOR**

   ​	**FOR**

   ​	SELECT	order	FROM	tb1；

   END；

2. 打开游标使用：OPEN    ord；关闭游标：CLOSE    ord；关闭之后再次打开可以继续使用；

3. 检索其中一行：

   CREATE	PROCEDURE	pcs（）	

   BEGIN

   ​	DECLARE	num	INT；

   ​	**DECLARE**	ord	**CURSOR**

   ​	**FOR**

   ​	SELECT	order	FROM	tb1；

   ​	OPEN	ord；

   ​	--	下一次使用FETCH自动切换下一行

   ​	FETCH	orders	INTO	num；

   ​	CLOSE	ord；

   END；

4. 循环检索从第一行到最后一行：

   CREATE	PROCEDURE	pcs（）	

   BEGIN

   ​	DECLARE	num	INT；

   ​	DECLARE	ord	CURSOR

   ​	FOR

   ​	SELECT	order	FROM	tb1；

   ​	**DECLARE	CONTINUE	HANDLER	FOR	SQLSTATE	'02000'	SET	done=1**；

   ​	OPEN	ord；

   ​	**REPEAT**

   ​		--	下一次使用FETCH自动切换下一行

   ​		FETCH	orders	INTO	num；

   ​	**UNTIL	done	END	REPEAT**

   ​	CLOSE	ord；

   END；

   这个存储过程中，定义了一个**CONTINUE	HANDLER**，当FOR后面的条件满足的时候就会调用，这里的条件是SQLSTATE ‘02000’出现的时候，SET设置done为1；SQLSTATE ‘02000’是未找到时出现的错误代码，也就是到达末尾的时候就会将done置1；





### 使用触发器

触发器是发生UPDATE、INSERT、DELETE响应的时候自动执行的一条MySQL语句。

1. 创建触发器：必须满足四个条件①触发器名唯一；②触发器关联某表；③触发器要响应的活动；④触发器何时执行；例子：

   **CREATE	TRIGGER**	newin	**AFTER	INSERT	ON**	tb1

   **FOR	EACH	ROW**	SELECT	‘product’；

   这里创建了一个newin触发器，这个触发器作用的时机是AFTER INSERT，即INSERT语句之后；此外使用了FOR EACH ROW表示对每一个插入的行执行SELECT；

   **注意：**

   ①触发器仅支持表，不支持视图；

   ②触发器以每个表每次事件为单位，所以一个表最多有六个触发器（INSERT、UPDATE、DELETE的BEFORE和AFTER）；

2. 删除触发器：DROP    TRIGGER    newin；

3. 使用**INSERT触发器**：INSERT触发器中有一个NEW的虚拟表，例如：

   CREATE	TRIGGER	newin	AFTER	INSERT	ON	tb1

   FOR	EACH	ROW	SELECT	**NEW.uid**；

   这里的NEW是一个临时表，在我们执行INSERT之后，会自动返回当时的uid，如果是AUTO_INCREMENT就可以返回那时候的值；

4. 使用**DELETE触发器**：相对地，DELETE触发器包含一个OLD临时表，但是这个表示只读的，例如：

   CREATE TRIGGER	newout	BEFORE	DELETE	ON	tb1

   FOR	EACH	ROW

   BEGIN

   ​	INSERT	INTO	tb2（uid，price）

   ​	VALUES（**OLD.uid，OLD.price**）；

   END；

   这里在DELETE语句之前会在tb2插入一行，**被删除的那一行的相关信息**；

5. 使用**UPDATE触发器**：使用OLD访问**UPDATE语句之前**的虚拟表，使用NEW访问**UPDATE之后**的虚拟表；下面的一个例子是在更新之前保证更新的数据是大写：

   CREATE	TRIGGER	newupd	BEFORE	UPDATE	ON	tb1

   FOR	EACH	ROW	**SET**	NEW.item=Upper（NEW.item）；

   注意不要漏了SET关键字；



### 管理事务处理

事务管理使用COMMIT和ROLLBACK来保证成批的SQL语句要么执行要不回滚。

1. 使用**START	TRANSACTION**开始一个事务；
2. 使用**ROLLBACK**可以回退到START TRANSACTION之前的状态，也只能在事务里面使用；同时，**只能回退**UPDATE、INSERT、DELETE语句，**不能回退**SELECT、CREATE、DROP语句；
3. 如果没有使用事务管理，语句都是隐含COMMIT，即自动提交的；如果使用事务管理，只有COMMIT才会使得这个事务提交；
4. COMMIT和ROLLBACK执行后，**事务会自动关闭**；
5. **保留点**：在事务中的一个中间状态，使用**SAVEPOINT    sp**创建一个名为sp的保留点；如果要回退到sp这个点，可以使用**ROLLBACK    TO    sp**；事务完成后自动释放所有保留点，也可以调用RELEASE   SAVEPOINT释放所有保留点；
6. 使用**SET    autocommit=0**可以关闭自动提交，此时只有使用COMMIT才会提交；这个关键字是**以连接为单位的，而不是数据库**；



### 全球化和本地化

1. 使用**SHOW    CHARACTER   SET**查看mysql支持的字符集；

2. 使用**SHOW    COLLATION**查看mysql支持的校对；

3. 可以创建表的时候指定使用的字符集和校对：

   CREATE	TABLE	myt（...）**DEFAULT	CHARACTER	SET**	hebrew	**COLLATE**	hebrew_general_ci；在这个表中明确指定了字符集和校对；同时，还可以对表中某一列单独指定特定的字符集和校对：

   CREATE	TABLE	myt（

   ​	column1	INT，

   ​	column2	VARCHAR（10）	**CHARACTER	SET**	latin1	**COLLATE**	latin1_general_ci

   ）**DEFAULT	CHARACTER	SET**	hebrew	**COLLATE**	hebrew_general_ci；

4. 校对在使用ORDER BY的时候起到至关重要的作用，在使用ORDER BY的时候可以指定校对：

   SELECT \*	FROM 	tb1

   ORDER	BY	lastname，firstname	COLLATE	latin1_general_cs；

5. 串与字符集的转换可以使用Cast（）和Convert（）；



### 安全管理

安全管理就是**访问控制**，即不同的用户拥有的权限不同。

1. 使用**USE mysql；SELECT user FROM user；**可以查看当前数据库有什么用户；
2. **创建一个用户：**使用**CREATE  USER  ben  IDENTIFIED  BY  'p@$$w0rd'**将创建一个账户名为ben，密码为‘p@$$w0rd’的用户；使用**RENAME  ben TO  john；**可以重命名；
3. **删除一个用户：**使用**DROP  USER  ben；**删除ben这个用户；
4. **设置权限：**使用**GRANT  SELECT  ON  bd1.\*  TO  ben；**语句允许用户ben在数据库db1上执行SELECT语句，即拥有只读权利；
5. **撤销权限：**使用**REVOKE  SELECT  ON  bd1.\*  FROM  ben；**语句撤销用户ben在数据库db1上执行SELECT语句，即剥夺只读权利；撤销的权利必须本来存在，否则出错；
6. 使用GRANT ALL赋予整个服务器的权限，GRANT ON db1.tb1赋予某个表的权限；

| 权限 | 说明 |
| ---- | ---- |
|      |      |

7. **更改口令：**使用**SET PASSWORD FOR ben=Password（‘newpw’）；**设置ben的密码，如果是用户，可以自己使用**SET PASSWORD = Password（‘newpw’）；**



### 数据库维护

1. 使用BACKUP TABLE或者SELECT INTO OUTFILE转储所有数据到外部文件；使用RESTORE TABLE来复原；

   但是备份之前必须先FLUSH TABLES 将所有数据（包括索引）写入到磁盘；

2. 使用**ANALYZE TABLE tb1；**检查表键是否正常；此外，还有CHECK TABLE tb1；针对许多问题进行检查；CHANGED检查最后一次检查之后改动过的表；EXTENDED进行最彻底的检查；FAST只检查没关闭的表；

3. 如果遇到出错的表，可以使用**REPAIR TABLE tb1；**来修复，但是不应该经常使用；此外，如果从表中删除一大块数据，需要使用**OPTIMIZE TABLE**来收回空间；

4. **诊断启动问题：**在命令行中执行mysql --version可以查看版本然后退出；此外，还有--help显示帮助；--verbose与help配合使用，显示全文本信息；--safe-mode装载减去那些最佳配置的服务器；

5. **查看日志文件：**

   ①错误日志：启动和关闭等任意关键错误的细节；

   ②查询日志：记录所有的mysql活动；

   ③二进制日志：记录有可能更新数据的所有语句；

   ④**缓慢查询日志：**记录那些查询时间超过一定时间的查询；



### 改善性能

- 设计数据库时：数据库表、字段的设计，存储引擎
- 利用好MySQL自身提供的功能，如索引等
- 横向扩展：MySQL集群、负载均衡、读写分离
- SQL语句的优化（收效甚微）







## 实战45讲

1. MySQL的整体框架：

   大体上说，MySQL可以分为server层和存储引擎层。

   **Server层**包括连接器、查询缓存、分析器、优化器、执行器等，涵盖MySQL的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

   **存储引擎层**负责数据的存储和提取。其架构模式是插件式的，支持InnoDB、MyISAM、Memory等多个存储引擎。现在最常用的存储引擎是InnoDB，它从MySQL 5.5.5版本开始成为了默认存储引擎。

   我们是客户端，使用`select * from T where ID=10`进行查询，会经历以下步骤：

   ①首先，使用**连接器**连接数据库。连接器负责跟客户端建立连接、获取权限、维持和管理连接。连接命令一般是这么写的：`mysql -h$ip -P$port -u$user -p`，首先会**建立TCP连接**，然后输入密码就完成了连接；与mysql的连接分为长连接和短连接，长连接默认为8小时，短连接会在几次查询后断开，再次使用的时候再连接；**但是全部使用长连接后，你可能会发现，有些时候MySQL占用内存涨得特别快，这是因为MySQL在执行过程中临时使用的内存是管理在连接对象里面的，这些资源会在连接断开的时候才释放。**可以考虑定期断开长连接或者使用`mysql_reset_connection`来重新初始化连接资源；

   ②连接建立完毕后就会在**查询缓存**里面查找请求。由于每一次查询之后都会将查询语句和查询结果用键值对的方式保存在缓存里，如果查询相同，则会立即返回查询结果；但是有一个弊端就是不使用与经常修改的数据库，只适合静态的数据库；（mysql8.0已经没有查询缓存这个模块了）

   ③如果查询缓存没有命中，就会使用**分析器**对语句进行分析，确定要执行的任务；

   ④在分析完成之后，会使用**优化器**对查询过程进行优化，例如决定索引查询的先后顺序等；

   ⑤在数据库中执行任务交给**执行器**处理。首先检查用户是否有权限查询这个表，如果有则调用数据库引擎的接口进行查询；在这里会调用innoDB引擎接口获取这个表的每一行，检查是否ID==10，是则保存结果，否则跳过；










### SQL语句专题

1. case语句

   1. 基础用法分为两种：

      1. ````bash
         case a
         when '1' then '男'
         when '2' then '女'
         else '其他' end
         ````

      2. ```bash
         case when a='1' then '男'
         when a='2' then '女'
         else '其他' end
         ```

      其中用法1一般就相当于c++的switch，用法2相当于if、else if；

   2. 对数据进行分组并分析；

      ![](img/sql1.png)

      首先按照country分成3组，然后统计每一组里面的人口数；

      ```bash
      SELECT  SUM(population), 
          CASE country  
              WHEN '中国' THEN '亚洲' 
              WHEN '印度' THEN '亚洲' 
              WHEN '日本' THEN '亚洲' 
              WHEN '美国' THEN '北美洲' 
              WHEN '加拿大'  THEN '北美洲' 
              WHEN '墨西哥'  THEN '北美洲' 
              ELSE '其他' 
          END 
      FROM    Table_A 
      GROUP BY
          CASE country 
              WHEN '中国' THEN '亚洲' 
              WHEN '印度' THEN '亚洲'
              WHEN '日本' THEN '亚洲' 
              WHEN '美国' THEN '北美洲' 
              WHEN '加拿大'   THEN '北美洲' 
              WHEN '墨西哥'   THEN '北美洲' 
              ELSE '其他' 
          END;
      ```

   3. 完成不同条件的分组：

      ![](img/sql2.png)

      这里虽然性别是一列，但是我们可以将其作为两列来展示；

      ```bash
      SELECT country, 
          SUM( CASE WHEN sex = '1' THEN  population ELSE 0 END),  --男性人口 
          SUM( CASE WHEN sex = '2' THEN  population ELSE 0 END)   --女性人口
      FROM  Table_A  
      GROUP BY country;
      ```

      因为group之后可以认为每一个country里面有很多行，使用case不会改变很多行这种情况，所以直接显示会显示第一行；使用sum将所有行合并成一行，所以显示就会正常；

   4. 根据条件进行update

      例如我们要将工资高于5000的人降薪百分之10，工资处于2000-5000的加薪百分之15，如何操作？

      ```bash
      --条件1
      UPDATE Personnel  SET salary = salary * 0.9  WHERE salary >= 5000;
      --条件2
      UPDATE Personnel  SET salary = salary * 1.15
      WHERE salary >= 2000 AND salary < 4600;
      ```

      使用两条进行操作会有一个缺点：如果一个人工资本来是5000，会**先降薪后加薪**，这是一个bug；因此必须一步到位：

      ```bash
      UPDATE Personnel
      SET salary =
      CASE WHEN salary >= 5000  　                THEN salary * 0.9
           WHEN salary >= 2000 AND salary < 4600  THEN salary * 1.15
      ELSE salary END; 
      ```

      对于不同的工资选择不同的操作，值得学习；

   5. 两个表格的数据是否一致的检查：

      如果有两个表，tbl_A,tbl_B，两个表中都有keyCol列。现在我们对两个表进行比较，tbl_A中的keyCol列的数据如果在tbl_B的keyCol列的数据中可以找到，返回结果'Matched',如果没有找到，返回结果'Unmatched'。
      要实现下面这个功能，可以使用下面两条语句：

      ```bash
      --使用IN的时候
      SELECT keyCol,
      CASE WHEN keyCol IN ( SELECT keyCol FROM tbl_B )  THEN 'Matched'
      ELSE 'Unmatched' END Label
      FROM tbl_A; 
      
      --使用EXISTS的时候
      SELECT keyCol,
      CASE WHEN EXISTS ( SELECT * FROM tbl_B  WHERE tbl_A.keyCol = tbl_B.keyCol )  THEN 'Matched'  ELSE 'Unmatched' END Label
      FROM tbl_A;
      ```

      这个用法就是根据条件输出不同的结果，而不是表中的结果，因此使用case会比较好；




2. 一些操作表的操作：

   1. alter table T add sex int default 1；在表T中增加一列sex，默认值为1；
   2. alter table T drop column sex；移除表T中sex这一列；
   3. alter table T drop primary key；移除表T中的主键；
   4. **alter table T add primary key （c_id,sex）；给表T添加主键（c_id,sex）**，添加之前要先移除主键；