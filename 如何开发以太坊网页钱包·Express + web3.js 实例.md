```
10.15.1. 创建项目
安装以太坊环境

curl -s https://raw.githubusercontent.com/oscm/shell/master/lang/gcc/gcc.sh | bash
curl -s https://raw.githubusercontent.com/oscm/shell/master/lang/golang/golang-1.10.2.sh | bash
curl -s https://raw.githubusercontent.com/oscm/shell/master/blockchain/ethereum/centos/go-ethereum-1.8.7.sh | bash
curl -s https://raw.githubusercontent.com/oscm/shell/master/blockchain/ethereum/systemd/private.sh | bash

curl -s https://raw.githubusercontent.com/oscm/shell/master/lang/node.js/binrary/node-v10.1.0.sh | bash
curl -s https://raw.githubusercontent.com/oscm/shell/master/lang/node.js/binrary/profile.d.sh | bash
curl -s https://raw.githubusercontent.com/oscm/shell/master/blockchain/ethereum/truffle/truffle.sh | bash
安装开发包

npm install express
npm install web3
npm install ejs
10.15.2. 主程序 main.js
var express = require('express');
var app = express();

app.use(express.static('public'));
app.set("view engine","ejs");
app.set('views', __dirname + '/views');  

var async = require('async');

fs = require('fs');
var net = require('net');
var Web3 = require('web3');
var web3 = new Web3('/home/ethereum/.ethereum/geth.ipc', net);
const abi = fs.readFileSync( __dirname + '/abi/NKC.abi', 'utf-8');
const coinbase = "0xaa96686a050e4916afbe9f6d62fa646dd8c51070"
const contractAddress = "0x5F75DA091aBb25e055B91172C04371Ff4Dd563a0";

console.log(web3.version)


app.get('/', function (req, res) {
  //  res.send('Hello World');
   res.render("index",{}); 
})

app.get('/account.html', function (req, res) {
  var accounts;
  web3.eth.getAccounts(function(err, acc) {
    accounts = acc
    res.render("account",{"accounts":accounts}); 
  });
})

app.get('/new', function (req, res) {
  web3.eth.personal.newAccount(req.query.password).then(function(){
    res.redirect('/account.html');
  });
})

app.get('/balance.html', function (req, res) {

  web3.eth.getAccounts(function(err, accounts) {
    res.render("balance",{"accounts":accounts}); 
  });
})
app.post('/showbalance.html', function (req, res) {
  // web3.eth.getBalance(req.query.account).then(function(balance){
  //   res.render("transfer",{"account":req.query.account, "balance": balance}); 
  // });
  
  res.render("showbalance",{"account": "sss", "balance": 1000}); 
})

app.get('/getbalance.html', function (req, res) {
  var contract = new web3.eth.Contract(JSON.parse(abi), contractAddress, { from: coinbase , gas: 100000});
  web3.eth.getBalance(req.query.account).then(function(balance){
    contract.methods.balanceOf(req.query.account).call().then(function(token){
      // console.log(contract.symbol.call());
      // contract.methods.symbol().call().then(console.log);
      contract.methods.symbol().call().then(function(name){
        res.render("showbalance",{"account":req.query.account, "balance": web3.utils.fromWei(balance, 'ether'), "token": token, "name": name}); 
      });
      
    });
    
  });
})

app.get('/transfer.html', function (req, res) {
  var contract = new web3.eth.Contract(JSON.parse(abi), contractAddress, { from: coinbase , gas: 100000});
  web3.eth.getAccounts(function(err, accounts) {
    contract.methods.symbol().call().then(function(symbol){
      res.render("transfer",{"accounts":accounts, "symbol": symbol}); 
    });
  });
})

app.get('/send', function (req, res) {
  // console.log(req.query)
  web3.eth.personal.unlockAccount(req.query.from, req.query.password).then(function(error){
    if(req.query.token == "ETH"){  
      web3.eth.sendTransaction({
        from: req.query.from,
        to: req.query.to,
        value: web3.utils.toWei(req.query.amount ,'ether')
      },
      function(error, result){
          if(!error) {
              console.log("#" + result + "#")
              res.render("done",{"hash":result}); 
          } else {
              console.error(error);
          }
      });
      
    }else{
      var contract = new web3.eth.Contract(JSON.parse(abi), contractAddress, { from: req.query.from , gas: 1000000});
      contract.methods.transfer(req.query.to, req.query.amount).send().then(function(hash){
        console.log(hash)
        res.render("done",{"hash":hash.transactionHash}); 
      });
    }
  });
})

var server = app.listen(8080, function () {
 
  var host = server.address().address
  var port = server.address().port
 
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
 
})
10.15.3. ABI 文件 abi/NKC.abi
[{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"burn","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_value","type":"uint256"}],"name":"burnFrom","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"},{"name":"_extraData","type":"bytes"}],"name":"approveAndCall","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"},{"name":"","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"initialSupply","type":"uint256"},{"name":"tokenName","type":"string"},{"name":"tokenSymbol","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Burn","type":"event"}]			
10.15.4. 页面视图
10.15.4.1. views/account.ejs

<%- include header.ejs %>

<h1>Users</h1>
<ul id="accounts">
    <% accounts.forEach(function(account, index){ %>
    <li><%= index %>, <%= account %></li>
    <% }) %>
</ul>

<p>
新建账号
<form method="get" action="/new">
    密码：<input type="password" name="password" />
    <input type="submit" value="新建账号" />
</form>
</p>
10.15.4.2. views/balance.ejs

<%- include header.ejs %>

<h1>Account</h1>
<form method="get" action="/getbalance.html">

    <select name="account">
        <% accounts.forEach(function(account, index){ %>
        <option value ="<%= account %>"><%= account %></option>
        <% }) %>
    </select>
    <input type="submit" value="Submit" />
</form>
10.15.4.3. views/done.ejs

<%- include header.ejs %>

<p>转账完成</p>
<p>查看交易
<a href="https://etherscan.io/tx/<%= hash %>" target="etherscan">主网<%= hash %></a> <br />
</p>
10.15.4.4. views/header.ejs

<a href="/account.html">账号</a> | <a href="/balance.html">余额</a> | <a href="/transfer.html">转账</a>
<br /> 
<hr />
10.15.4.5. views/index.ejs

<%- include header.ejs %>

Welcome !!!
10.15.4.6. views/showbalance.ejs

<%- include header.ejs %>
<p>
<h1>Account: <%= account %>, Balance: <%= balance %></h1>
</p>

<p>
    Token: <%= token%> <%= name%> 
</p>
10.15.4.7. views/transfer.ejs

<%- include header.ejs %>

<h1>Account</h1>
<form method="get" action="/send">
    From：
    <select name="from">
        <% accounts.forEach(function(account, index){ %>
        <option value ="<%= account %>"><%= account %></option>
        <% }) %>
    </select>
    <br />
    To:
    <select name="to">
        <% accounts.forEach(function(account, index){ %>
        <option value ="<%= account %>"><%= account %></option>
        <% }) %>
    </select>

    <select name="token">
        <option value ="ETH">ETH</option>
        <option value ="<%= symbol %>"><%= symbol %></option>
    </select>

    <br />
    金额: <input type="text" name="amount" /> ETH
    <br />
    密码：<input type="password" name="password" />
    <br />
    <input type="submit" value="Submit" />
</form>
10.15.5. 启动 Node 服务
neo@MacBook-Pro ~/example % node main.js
浏览器访问 http://localhost:8080/ 可以进入钱包

```
