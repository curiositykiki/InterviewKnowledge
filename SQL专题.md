1. 组合两个表，无论person有没有address都需要返回信息：

   ![](SQLimg/1.png)

   首先可以看到Address中PersonId是一个外键，这就是这两个表联结属性；然后因为不管有没有address都需要返回信息，所以应该是外部联结于Person表，因此：

   ```bash
   select FirstName,LastName,City,state
   
   from Person left outer join Address
   
   on Person.PersonId=Address.PersonId
   ```

   

   

   

2. 获取表中薪水第二高的是多少

   ![](SQLimg/2.png)

   首先按照常规做法先查找薪水第二高的数值，方法是按照薪水降序排序之后选择偏移为1，显示第一个数值；因为有可能会不存在，所以用到一个ifnull函数：

   IFNULL(expression_1,expression_2);

   IFNULL函数是MySQL控制流函数之一，它接受两个参数，**如果第一个参数不是NULL，则返回第一个参数。 否则，IFNULL函数返回第二个参数**。因此可得：

   ```bash
   select 
   
   IFNULL(
   
     (select distinct Salary
   
     from Employee
   
     order by Salary desc
   
     limit 1 offset 1),
   
     NULL
   
   )
   
   AS SecondHighestSalary;
   ```

   

   

   

3. 查看表中第n高的薪水是多少？

   ![](SQLimg/3.png)

   因为还是有可能为空，跟刚才一样需要使用ifnull，并且可能有重复记录我们需要去重，**注意，limit m，n表示从第m个记录开始取n个记录，**可得：

   ```bash
   CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
   
   BEGIN
   
     set N:=N-1;#注意不要放在return里面
   
    RETURN (
   
      \# Write your MySQL query statement below.
   
      select
   
      ifnull(
   
   ​     (
   
   ​       select distinct salary
   
   ​       from employee
   
   ​       order by salary desc
   
   ​       limit N,1
   
   ​     )
   
      ,NULL)
   
      as getNthHighestSalary
   
    );
   
   END
   ```

   

   

   

4. 实现排名系统，同分的名次相同，但是不占用排名的名额。

   ![](SQLimg/4.png)

   使用子表的思想，例如我们取得其中一个记录4.00，然后跟这个表里面的数据比较，统计有多少人的分数大于等于我，然后去重，4.00就只有本身符合条件，所以结果为1；以此类推3.85有4.00和3.85大于等于3.85，所以结果为2；这里查询的时候要去重，最后输出不需要去重；然后Rank是关键字，所以要使用‘Rank’，得：

   ```bash
   select s1.score,
   
     (
   
   ​    select count(distinct score)
   
   ​    from scores
   
   ​    where score>=s1.score
   
     )  'Rank'
   
   from scores s1
   
   order by s1.score desc
   ```

   

   

   

5. 找到表中连续出现三次及以上的数字，注意是需要连续出现的

   ![](SQLimg/5.png)

   可以直接三个字表，然后比较Id连续的情况下，Num还一样的那个Num，注意要去重，否则可能会出现多次同一个答案，得：

   ```bash
   select distinct l1.num ConsecutiveNums
   
   FROM 
   
     Logs l1,
   
     Logs l2,
   
     Logs l3
   
   where 
   
     l1.Id=l2.Id-1
   
     and l2.Id=l3.Id-1
   
     and l1.Num=l2.Num
   
     and l2.Num=l3.Num
   ```

   

   

   

6. 有一个职员表，里面有员工和他的上司，找到工资比他上司高的员工

   ![](SQLimg/6.png)

   这题应该比较简单了，就是用一个子表来搜索其上司的工资，比较返回工资比上司高的就行：

   ```bash
   select e1.name Employee
   from employee e1
   where e1.salary>(select salary from employee where e1.managerid=id)
   ```

   

   

   

7. 返回出现重复的邮箱

   ![](SQLimg/7.png)

   这题很简单，就是按照邮箱分组，然后统计每一个分组里面的数量，大于1就是有重复。count（\*）就是统计当前有多少行，having就是对每一个group都执行一遍：

   ```bash
   select distinct email
   from person
   group by email
   having count(*)>1
   ```

   

   

   

8. 给定两个表，一个客户一个订单，找到没有下过单的客户

   ![](SQLimg/8.png)

   将两个表外部联结，约束为customers.id=orders.customerid，此时如果没有下过单的客户其customerid应该是null，我们就找到这个null的客户就行了：

   ```bash
   select name Customers
   from customers left join orders
   on customers.id=orders.customerid
   where customerid is NULL
   ```

   

   

   

9. 给定一个职员表和部门表，返回每个部分工资最高的人的相关信息，可以有多个人并列

   ![](SQLimg/9.png)

   首先我们需要明确用E.departmentid与D.id建立一个约束，以取的正确的Department；然后由于有可能多个人工资相同，所以这里的判断是某个人的工资=这个部门的最高工资，为了保证返回的行数唯一并且部门正确，我们子查询采取的约束是**当前的D.id=子查询里面的departmentid**；

   ```bash
   select D.name Department,E.name Employee,E.salary
   from employee E,Department D
   where E.departmentid=D.id
   and E.salary=(select max(salary) from employee where employee.departmentid=D.Id)
   ```

   

   

   

10. 给定一个职员表和一个部门表，返回各个部门工资前三的人，工资可能相同

    ![](SQLimg/10.png)

    首先就是将E.departmentid与D.id联结起来，相当于分了组；然后在每一组里检查当前这个人的工资在他的组内比他高工资的人是否少于3个，工资不重复；注意这里的子查询一样要限制E.departmentid=D.id，这样才能保证子查询内的范围正确：

    ```bash
    select D.name Department,E.name Employee,E.salary
    from employee E,department D
    where E.departmentid=D.id
    and (select count(distinct E2.salary) from Employee E2 where E2.departmentid=D.id and E2.salary>E.salary)<3
    ```

    

    

    

11. 删除邮箱重复的行，保留Id最小的哪一行。

    ![](SQLimg/11.png)

    注意到保留最小的Id，那就可以找到邮箱不重复的最小的id，然后删除不再这个集合里面的行就行了：

    ```bash
    delete from Person
    where id not in(
        select t.id
        from (
            select min(id) id from person group by email    
        ) t
    )
    ```

    

    

    

12. 找到比昨天天气要热的那些天的id

    ![](SQLimg/12.png)

    很简单，约束就两个，一个是温度比昨天高，第二个是日期。使用DATE_SUB（date，INTERVAL N DAY）表示从date里面减去N天得到的时候，所以可以得到：

    ```bash
    select w1.id
    from weather w1,weather w2
    where w1.temperature>w2.temperature
    and w2.recorddate=DATE_SUB(w1.recorddate,INTERVAL 1 DAY)
    ```

    

    

    

13. 给定两个表，一个记录约车订单的时间，用户，司机和结果；第二个记录用户或者司机是否被拉黑了；返回10.01到10.03这三天，每天的订单取消率（今天取消订单/今天所有订单），拉黑的用户司机不算。

    ![](SQLimg/13-1.png)

    ![](SQLimg/13-2.png)

    首先按照三个条件筛选，用户没有被拉黑，司机没有被拉黑，时间为10.01到10.03；然后按照时间分组；使用round（num，n）表示显示num小数点后n位，if（exp，val1，val2）表示exp为真就返回val1，否则返回val2；统计每一个分组内的取消订单/今日所有订单：

    ```bash
    select T.request_at Day,round(count(if(T.status!='completed',true,null))/count(*),2) 'Cancellation Rate'
    
    from Trips T
    
    where T.client_id in (select users_id from users where banned='No')
    
    and T.driver_id in (select users_id from users where banned='No')
    
    and request_at between '2013-10-01' and '2013-10-03'
    
    group by request_at
    ```

    

    

    

14. 给定一个表，如果面积超过3百万或者人口超过25百万就是大国，返回这些大国

    ![](SQLimg/14.png)

    就直接筛选就行：

    ```bash
    select name,population,area
    from World
    where area>3000000 or population>25000000
    ```

    

    

15. 给定一个选课表，返回选课学生大于等于5的课程，学生有可能重复

    ![](SQLimg/15.png)

    就直接分组统计就行，注意这里先where再group表示筛选之后再分组；group再having表示分组之后再筛选：

    ```bash
    select class
    from courses 
    group by class
    having count(distinct student)>=5
    ```

    

    

16. 给定一个访客表，当有连续三天或者更多天访客数达到100及以上的时候，返回所有的这些行

    ![](SQLimg/16.png)

    跟之前找连续三行的方法有点像，但是区别在于，这里要返回所有的行，也就是说，我们应该能够返回在连续三行中任意位置的行，而不是仅仅返回头部：

    ```
    SELECT DISTINCT a.*
    FROM stadium a, stadium b, stadium c
    WHERE a.people >= 100 AND b.people >= 100 AND c.people >= 100
    AND (
        (a.id + 1 = b.id AND a.id + 2 = c.id) OR # here, b, c
        (b.id + 1 = a.id AND b.id + 2 = c.id) OR # b, here, c
        (c.id + 1 = b.id AND c.id + 2 = a.id)    # b, c, here
    )
    ORDER BY a.id
    ```



17. 给定一个电影评价表，返回评价不是boring，id为奇数的电影，按照rating降序排序

    ![](SQLimg/17.png)

    首先筛选出id为奇数的行，然后筛选出description不为boring的电影，最后排序：

    ```bash
    select c1.*
    from cinema c1
    where c1.id%2=1
    and c1.description<>'boring'
    order by rating desc
    ```



18. 有一个座位表，交换相邻两个同学的座位：

    ![](SQLimg/18.png)

    

思路比较奇特，就是改变所有同学的id，然后按id排序；例如，1号同学最终的id为2，2号同学最终的id为1，所以我们可以将奇数id+1，偶数id-1的方法改变id；同时检查最后一个id是否是奇数，如果是则不变，如果是偶数则id-1；仔细品味：

```bash
select if(id<(select count(*) from seat),if(id mod 2 = 1,id+1,id-1),if(id mod 2 = 1,id,id-1)) as id,student
from seat
order by id
```



19. 给定一个表，替换所有的性别，只能使用一句update

    ![](SQLimg/19.png)

    使用一句update来替换所有的sex，我们可以判断如果sex=‘m’就替换为‘f’，否则替换为‘m'：

    ```bash
    update salary set sex = if(sex='f','m','f')
    ```

    

20. 小知识，如果清空一个表：一般要删除一个表，我们可以drop table；但是这样的结果就是这个表不存在了，如果我们只是想清空呢？可以使用truncate table，这样这个表还是存在的，只是数据清空了；



21. 给定一个表包含每个部门每个月的收入，将其输出调整一下：

    ![](SQLimg/20.png)

    这里就是一个基础的调整输出了：

    ```bash
    select id, 
    	sum(case when month = 'jan' then revenue else null end) as Jan_Revenue,
    	sum(case when month = 'feb' then revenue else null end) as Feb_Revenue,
    	sum(case when month = 'mar' then revenue else null end) as Mar_Revenue,
    	sum(case when month = 'apr' then revenue else null end) as Apr_Revenue,
    	sum(case when month = 'may' then revenue else null end) as May_Revenue,
    	sum(case when month = 'jun' then revenue else null end) as Jun_Revenue,
    	sum(case when month = 'jul' then revenue else null end) as Jul_Revenue,
    	sum(case when month = 'aug' then revenue else null end) as Aug_Revenue,
    	sum(case when month = 'sep' then revenue else null end) as Sep_Revenue,
    	sum(case when month = 'oct' then revenue else null end) as Oct_Revenue,
    	sum(case when month = 'nov' then revenue else null end) as Nov_Revenue,
    	sum(case when month = 'dec' then revenue else null end) as Dec_Revenue
    from department
    group by id
    order by id
    ```

    **为什么需要sum呢？**

    这里我们按照id分组了，然后可以认为每一组里面有很多不同月份的revenue；然后我们使用case对其进行处理，其实**case处理之后的结果还是很多行的**，所以显示只会显示第一行，结果就是12个月份里面只有一个月份是有数值，其他都是null；因此使用sum可以将多行的数据合并成一个数据，这样就能正常显示了；

    没有sum：

    | id   | Jan         | Feb         |
    | ---- | ----------- | ----------- |
    | 1    | 123<br>null | null<br>321 |

    这时候如果select只会显示：

    | id   | Jan  | Feb  |
    | ---- | ---- | ---- |
    | 1    | 123  | null |

    有sum：

    | id   | Jan  | Feb  |
    | ---- | ---- | ---- |
    | 1    | 123  | 321  |



22. 将一个表的行和列进行互换，例如：

    ![](SQLimg/22-1.png)

    转换为：

    ![](SQLimg/22-2.png)

    或者：

    ![](SQLimg/22-3.png)

    这题还是挺有意思的，这个转换之后显示更加清楚了。

    ```sql
    SELECT user_name ,
        MAX(CASE course WHEN '数学' THEN score ELSE 0 END ) 数学,
        MAX(CASE course WHEN '语文' THEN score ELSE 0 END ) 语文,
        MAX(CASE course WHEN '英语' THEN score ELSE 0 END ) 英语
    FROM test_tb_grade
    GROUP BY USER_NAME;
    ```

    这里使用分组group先按照姓名分组了，然后在组内使用case进行筛选，最后使用max取得筛选之后课程的得分显示出来。





