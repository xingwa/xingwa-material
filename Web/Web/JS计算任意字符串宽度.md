```
QuoJS(针对移动设备的微型JavaScript 库)

QuoJS 是一个微型的，模块化的，面向对象的和简洁的JavaScript库，它简化了HTML文档遍历。针对快速发展的移动互联网的事件处理和Ajax交互，以其优雅的，文档丰富和连贯的API让你写出强大，灵活的和跨浏览器的代码。

QuoJS 是被设计来改变你写 JavaScript 的方式，一个5-6k的gzip压缩库用它完美的API处理了大部分基本工作，这样你就可以集中精力做自己的事了。QuoJS 在开源MIT下许可，这样你就可以随意使用和更改代码。

目前很多 JavaScript 库对移动端是不友好的，他们体积很大，集成了很多桌面设备需要的东西，所以移动端的性能不是最佳的。对触摸事件或 帮助开发人员创造一个好和酷的 JavaScript 的语义API支持不好。








http://www.jq22.com/yanshi2534（turn.js实现翻书效果）





（JS计算任意字符串宽度）

<!DOCTYPE html>
<html> 
<head>
  <script src="jquery.min.js"></script>
</head>
<body>
  <div id='labelText' style='color:black;line-height:1.2;white-space:nowrap;top:0px;left:0px;position:fixed;display:block;visibility:visible;'>
    
  </div>
 
  <script>
    var str="Live like you were dying, Love because you do.";
    str = str.substring(0,str.length);
    $("#labelText").css({
      "font-size": "12px",
      "font-family": "Microsoft YaHei"
    }).html(str);
    var width = $("#labelText").width();
    console.log(width);
</script>
</body>


```
