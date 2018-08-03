###### 递归查询文件并删除
```
   find ./ -name "xingwa.*" | xargs rm -f
```
###### 生成密码
```
echo "admin:$(openssl passwd -crypt 'admin')" >>/usr/local/nginx/conf/htpasswd （生成密码）
```
###### 根据文件号删除
```
ls -li
find ./ -inum 134490405 -exec rm -rf {} \; （生成密码）
```
