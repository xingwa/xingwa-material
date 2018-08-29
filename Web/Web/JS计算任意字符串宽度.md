```

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
