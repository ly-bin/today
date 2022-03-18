
##### 1、题目：https://leetcode-cn.com/problems/combine-two-tables/。   简单
##### 2、思路：
> 1、数据库的连接查询，为完任务😜😣
##### 3、代码
```
select p.firstName, p.lastName, a.city, a.state 
from Person p left join Address a on p.PersonId = a.PersonId
```
