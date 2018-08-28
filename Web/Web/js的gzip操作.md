```
https://blog.csdn.net/zunwei/article/details/49886115
http://nodeca.github.io/pako/#ungzip

https://www.jb51.net/article/116026.htm



// var pako = require('pako');

// var test = { my: 'super', puper: [456, 567], awesome: 'pako' };

// var binaryString = pako.deflate(JSON.stringify(test), { to: 'string' });

// console.log(binaryString)

var zlib = require('zlib');
// var input = 'welcone to pkcms.cnwelcone to pkcms.cnwelcone to pkcms.cnwelcone to pkcms.cn';
 
// zlib.gzip(input, function(err, buffer) {
//     if (!err) {
//         console.log("gzip (%s): ", buffer.length, buffer.toString('base64'));
//     }
// });


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>pako.js实现gzip的压缩和解压</title>
</head>
<body>
<script type="text/javascript" src="pako.js"></script>
<script >
var b64Data   = 'H4sIAAAAAAAAAJ3UMQ7CMAwF0KugP2ewEzdpcxXUAbWAOiHUMqCqdyeVQAobfGXIYL8hP5ZXnEdkeNEk6vUgXTbLonC4zMjHFY/5Wm511ekdTsOCLKVp2rlIKOA2jTuBot/cr7BhobEwsbAloY8kDGyqoQ5H/oHsdwQ21cCmaspCz0L2jcYOgLHhNGw4TT1yVmBpuS9PZHWY35siqnxvimEvpE9FY4peQhfbhO0FDnuFqWAEAAA=';
var ticketData = '{"ed":"20170121 09:44:01","fs":[{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"003","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"005","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"004","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"007","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"008","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"026","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"033","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"034","oids":["0"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"035","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"037","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"038","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"041","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"042","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"047","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"046","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"048","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"051","oids":["1"]},{"usg":[1,1,1,1,1,1,1],"act":0,"fid":"053","oids":["4"]}],"qty":1,"sd":"20161021 09:44:01","cd":"72016102116762039687"}';
// Output to console
var s = unzip(b64Data);
console.log("unzipped:");
console.log(s);
var data = zip(ticketData);
console.log("zipped:");
console.log(data);
function unzip(b64Data){
  var strData   = atob(b64Data);
  // Convert binary string to character-number array
  var charData  = strData.split('').map(function(x){return x.charCodeAt(0);});
  // Turn number array into byte-array
  var binData   = new Uint8Array(charData);
  // // unzip
  var data    = pako.inflate(binData);
  // Convert gunzipped byteArray back to ascii string:
  strData   = String.fromCharCode.apply(null, new Uint16Array(data));
  return strData;
}
function zip(str){
  var binaryString = pako.gzip(str, { to: 'string' });
  return btoa(binaryString);
}
</script>
</body>
</html>

```
