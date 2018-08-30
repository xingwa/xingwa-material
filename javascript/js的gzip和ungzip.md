### 其他语言（https://blog.csdn.net/a2241076850/article/details/54600192）

```
<html>
<head>
<meta charset="UTF-8">
<script src="http://code.jquery.com/jquery-2.1.4.min.js"></script>
<script src="https://cdn.bootcss.com/pako/1.0.6/pako.min.js"></script>
<script type="text/javascript">  
function gzip(string) {  
string = encodeURIComponent(string)
var charData    = string.split('').map(function(x){return x.charCodeAt(0);});  
var binData     = new Uint8Array(charData);  
var data= pako.gzip(binData);  
var strData= String.fromCharCode.apply(null, new Uint16Array(data));  
return btoa(strData);  
}  
function ungzip(string)  
{  
var strData     = atob(string);  
var charData    = strData.split('').map(function(x){return x.charCodeAt(0);});  
var binData     = new Uint8Array(charData);  
var data= pako.ungzip(binData);  
var strData= String.fromCharCode.apply(null, new Uint16Array(data));  

return decodeURIComponent(strData);  
}  
test="{id:1000}";  
var s=gzip(test); 
console.log(s) 
console.log(ungzip(s)) 
</script>
</head>
</html>


```
