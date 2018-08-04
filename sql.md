####更新数据，查询更新
```
UPDATE invite_user INNER JOIN (SELECT userID,teamID,inviterID FROM t_user) c SET invite_user.teamID=c.teamID,invite_user.pID=c.inviterID WHERE invite_user.userID = c.userID AND c.userID=300662


update entrust999 as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans` WHERE entrustID = 124747) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount

update entrust999 as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans` GROUP BY entrustID) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount


update `entrust_hhtb` as a  inner join(  SELECT sum(dealAmount) as dealAmount,entrustID FROM `trans_hhtb` WHERE entrustID = 124747) c on a.entrustID = c.entrustID set a.dealAmount = c.dealAmount
```
