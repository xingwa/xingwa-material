### LINUX获取前一天日期的方法
```
获取前一天日期的方法 ..
写SHELL时,有时候很有用的..

linux下 
前一天的日期
date -d "1 day ago" +"%y%m%d"

前一个月的日期
date -d "1 month ago" +"%y%m%d"

类似的还有
date -d "-1 day ago 1 month ago" +"%y%m%d"
date -d "1 day ago -1 year ago 1 month ago" +"%y%m%d"


下面是SHELL的例子：

DAYDEL=`date -d "1 month ago" +%m%d`   //一个月前的日期

echo $DAYDEL
```
