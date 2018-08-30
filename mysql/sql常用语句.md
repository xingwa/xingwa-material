####更新数据，查询更新
```
UPDATE invite_user INNER JOIN (SELECT userID,teamID,inviterID FROM t_user) c SET invite_user.teamID=c.teamID,invite_user.pID=c.inviterID WHERE invite_user.userID = c.userID AND c.userID=300662


update entrust999 as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans` WHERE entrustID = 124747) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount

update entrust999 as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans` GROUP BY entrustID) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount


update `entrust_hhtb` as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans_hhtb` WHERE entrustID = 124747) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount













mysql中用命令行复制表结构的方法主要有一下几种: 

1.只复制表结构到新表
1 CREATE TABLE 新表 SELECT * FROM 旧表 WHERE 1=2;
或

1 CREATE TABLE 新表 LIKE 旧表 ;
注意上面两种方式，前一种方式是不会复制时的主键类型和自增方式是不会复制过去的，而后一种方式是把旧表的所有字段类型都复制到新表。

 
2.复制表结构及数据到新表
1 CREATE TABLE 新表 SELECT * FROM 旧表
 
3.复制旧表的数据到新表(假设两个表结构一样) 
1 INSERT INTO 新表 SELECT * FROM 旧表
 
4.复制旧表的数据到新表(假设两个表结构不一样)
1 INSERT INTO 新表(字段1,字段2,.......) SELECT 字段1,字段2,...... FROM 旧表
```
