```
（正经的解决办法）

在项目中安装：npm install --global prettier

在项目的根目录下（同级index.html文件）新建.prettierrc.json，并配置：

{
    "singleQuote":true,//使用单引号而不是双引号
    "semi":false//在语句结尾处不打印分号
}
至此，当我们格式化时就不会出现不需要的分号与双引号了
--------------------- 
作者：尖沙咀段坤写Bug 
来源：CSDN 
原文：https://blog.csdn.net/qq_39009348/article/details/82708208 
版权声明：本文为博主原创文章，转载请附上博文链接！

```
