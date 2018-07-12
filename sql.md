####更新数据，查询更新
```
UPDATE invite_user INNER JOIN (SELECT userID,teamID,inviterID FROM t_user) c SET invite_user.teamID=c.teamID,invite_user.pID=c.inviterID WHERE invite_user.userID = c.userID AND c.userID=300662
```
