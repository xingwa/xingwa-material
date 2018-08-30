### server
```
// JavaScript Document



var http = require('http');
var zlib = require('zlib');
var btoa = require('btoa');



var test = { my: 'super', puper: [{"lastTime":"成都市","count":"0","record":[],"hrecord":[[908395,0.0437,1.0000,0,0,1,1535608069000,1,8,0,0],[908380,0.0437,1.0000,0,0,1,1535606984000,1,8,0,0],[908379,0.0437,1.0000,0,0,1,1535606975000,1,8,0,0],[888396,0.0457,1.0000,0,0,1,1535445944000,1,8,0,0],[887833,0.0457,1.0000,0,0,1,1535440986000,1,8,0,0],[887556,0.0457,1.0000,0,0,1,1535437768000,1,8,0,0],[887555,0.0457,1.0000,0,0,1,1535437753000,1,8,0,0],[887550,0.0457,1.0000,0,0,1,1535437270000,1,8,0,0],[887543,0.0456,1.0000,0,0,1,1535436898000,1,8,0,0],[887538,0.0456,1.0000,0,0,1,1535436704000,1,8,0,0]],"precord":[]}], awesome: 'pako' };
 
//var binaryString = pako.gzip(JSON.stringify(test), { to: 'string' });

//binaryString = Buffer.from(binaryString).toString('base64')

//var restored = JSON.parse(pako.inflate(binaryString, { to: 'string' }));

//console.log(restored)

//console.log(binaryString)

http.createServer(function (request, response) {
  response.writeHead(200,{'Access-Control-Allow-Origin':'*','Access-Control-Allow-Method':'GET,POST','Content-Type':'application/octet-stream'});
    //response.setHeader("Access-Control-Allow-Origin", "*");
   // response.setHeader('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS');
   // response.setHeader("Access-Control-Allow-Headers", "X-Requested-With");
   // response.setHeader('Access-Control-Allow-Headers', 'Content-Type');
   /// response.writeHead(200, {'Content-Type': 'text/plain'});

//console.log(Buffer.from('成都市'))

const arr = new Uint8Array(110);

arr.set([15, 26, 91, 123, 34, 108, 97, 115, 116, 84, 105, 109, 101, 34, 58, 34, 49, 53, 51, 53, 54, 48, 53, 52, 50, 57, 34, 44, 34, 99, 111, 117, 110, 116, 34, 58, 34, 48, 34, 44, 34, 114, 101, 99, 111, 114, 100, 34, 58, 91, 93, 44, 34, 104, 114, 101, 99, 111, 114, 100, 34, 58, 91, 91, 56, 56, 56, 51, 57, 54, 44, 48, 46, 48, 52, 53, 55, 44, 49, 44, 48, 44, 48, 44, 49, 44, 49, 53, 51, 53, 52, 52, 53, 57, 52, 52, 48, 48, 48, 44, 49, 44],1)

// Copies the contents of `arr`
//const buf1 = Buffer.from('成都市小天小区成都市小天小区成都市小天小区成都市小天小区{id:1000}');
const buf1 = Buffer.from(arr);



  zlib.gzip( encodeURIComponent(JSON.stringify(test)) , function (err, buffer) {
	        console.log(btoa(buffer))
            response.end(btoa(buffer));
        });




    // console.log(buf1.toString())
    //response.end(buf1.toString('base64'));
	//response.end(binaryString);
}).listen(8888);


console.log('Server running at http://127.0.0.1:8888/');


```














### client

```
<html>
<head>
<meta charset="UTF-8">
<script src="http://code.jquery.com/jquery-2.1.4.min.js"></script>
<script src="https://cdn.bootcss.com/pako/1.0.6/pako.min.js"></script>
<script type="text/javascript">  

function str2UTF8(str){
	var bytes = new Array(); 
	var len,c;
	len = str.length;
	for(var i = 0; i < len; i++){
		c = str.charCodeAt(i);
		if(c >= 0x010000 && c <= 0x10FFFF){
			bytes.push(((c >> 18) & 0x07) | 0xF0);
			bytes.push(((c >> 12) & 0x3F) | 0x80);
			bytes.push(((c >> 6) & 0x3F) | 0x80);
			bytes.push((c & 0x3F) | 0x80);
		}else if(c >= 0x000800 && c <= 0x00FFFF){
			bytes.push(((c >> 12) & 0x0F) | 0xE0);
			bytes.push(((c >> 6) & 0x3F) | 0x80);
			bytes.push((c & 0x3F) | 0x80);
		}else if(c >= 0x000080 && c <= 0x0007FF){
			bytes.push(((c >> 6) & 0x1F) | 0xC0);
			bytes.push((c & 0x3F) | 0x80);
		}else{
			bytes.push(c & 0xFF);
		}
	}
	return bytes;
}
//console.log(str2UTF8(JSON.stringify([{"lastTime":"1535605429","count":"0","record":[],"hrecord":[[888396,0.0457,1.0000,0,0,1,1535445944000,1,8,0,0],[887833,0.0457,1.0000,0,0,1,1535440986000,1,8,0,0],[887556,0.0457,1.0000,0,0,1,1535437768000,1,8,0,0],[887555,0.0457,1.0000,0,0,1,1535437753000,1,8,0,0],[887550,0.0457,1.0000,0,0,1,1535437270000,1,8,0,0],[887543,0.0456,1.0000,0,0,1,1535436898000,1,8,0,0],[887538,0.0456,1.0000,0,0,1,1535436704000,1,8,0,0],[887490,0.0456,1.0000,0,0,1,1535436319000,1,8,0,0],[887315,0.0456,1.0000,0,0,1,1535435394000,1,8,0,0],[886125,0.0456,1.0000,0,0,1,1535421633000,1,8,0,0]],"precord":[]}])));
 function byteToString(arr) {
			if(typeof arr === 'string') {
				return arr;
			}
			var str = '',
				_arr = arr;
			for(var i = 0; i < _arr.length; i++) {
				var one = _arr[i].toString(2),
					v = one.match(/^1+?(?=0)/);
				if(v && one.length == 8) {
					var bytesLength = v[0].length;
					var store = _arr[i].toString(2).slice(7 - bytesLength);
					for(var st = 1; st < bytesLength; st++) {
						store += _arr[st + i].toString(2).slice(2);
					}
					str += String.fromCharCode(parseInt(store, 2));
					i += bytesLength - 1;
				} else {
					str += String.fromCharCode(_arr[i]);
				}
			}
			return str;
		}


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

var req = new XMLHttpRequest();
    req.open('GET', "http://127.0.0.1:8888/");
    req.responseType = "arraybuffer";
    req.send();

    req.onreadystatechange = function () {

        if (req.readyState === 4) {
	
            var buffer = req.response;
           console.log('zhong',buffer)
            var dataview = new DataView(buffer);
			
		   var bytes = new Array(); 
            var ints = new Uint8Array(buffer.byteLength);
            for (var i = 0; i < ints.length ; i++) {
				
                ints[i] = dataview.getUint8(i);
				bytes.push(ints[i])
            }
        console.log(  ungzip(byteToString(bytes))   );
        }
    }
</script>
</head>
</html>


```
