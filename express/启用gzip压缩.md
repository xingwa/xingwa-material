
##### 待安装包安装完之后，我们再在app.js文件里，增加这两句代码。（请确保这个包在所有中间件之前加载）

```
var compression = require('compression');  
app.use(compression());  

```
