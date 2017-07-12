---
layout: post
title:  "Bulding your first blockchain app with Ethereum"
date:   2017-07-12 21:14:42
tags: [ethereum, dapp, blockchain, javascript]
categories: [javascript, ethereum]
comments: true
---

### Introduction
Today we are going to make our first simple decentralized app using the blockchain.

The app is going to be a simple micro blog service.

### Setup
{% highlight bash %}
mkdir dapp-demo
cd dapp-demo
npm init
npm install ethereumjs-testrpc web3 solc http-server --save
{% endhighlight %}

ethereumjs-testrc is for running a local dev blockchain

web3 is the ethereum js client 

solc is for compiling smart contracts

http-server for running the web app in a dev env

### Creating the smart contract
MyBlog.sol
{% highlight javascript %}
pragma solidity ^0.4.11;

contract MyBlog {

  bytes32 private blogName;
  
  bytes32[] private messages;

  function MyBlog(bytes32 title) {
    blogName = title;
  }

  function addPost(bytes32 message) {
    messages.push(message);
  }

  function getMessages() returns (bytes32[]) {
    return messages;
  }

  function getBlogName() returns (bytes32) {
    return blogName;
  }

}
{% endhighlight %}
Read more about [solidity](http://solidity.readthedocs.io/){:target="_blank"}

### Compiling the smart contract and deploying the code to the blockchain
compile.js
{% highlight javascript %}
fs = require('fs');
Web3 = require('web3');
solc = require('solc');

web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
code = fs.readFileSync('src/contract/MyBlog.sol').toString();
compiledCode = solc.compile(code);
abiDefinition = JSON.parse(compiledCode.contracts[':MyBlog'].interface);
console.log(abiDefinition);
MyBlogContract = web3.eth.contract(abiDefinition);
byteCode = compiledCode.contracts[':MyBlog'].bytecode;

// Get first account from the blockchain to deploy the contract
blogTitle = "Demo Blog";
MyBlogContract.new(blogTitle ,{data: byteCode, from: web3.eth.accounts[0], gas: 4700000}, function(err, myContract){
    if(!err && myContract.address) {
        console.log(myContract.address);
    }
});
{% endhighlight %}

Run the test blockchain by executing ./node_modules/.bin/testrpc

Then execute 'node compile.js' and save the outputs. 

### Building the web app

index.html
{% highlight html %}
<html>
    <head>
        <title>Decentralized Blog</title>
        <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
        <script src="https://cdn.rawgit.com/ethereum/web3.js/develop/dist/web3.js"></script>
        <script src="app.js"></script>
    </head>
    <body>
        <ul id="messages"></ul>
        <input type="text" id="message"/>
        <button id="send" onClick="addPost()">Send</button>
    </body>
</html>
{% endhighlight%}

app.js (Use the outputs from the compile.js)
{% highlight javascript %}
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
// Extracted from compiledCode.contracts[':MyBlog'].interface
var abi = JSON.parse('[{"constant":false,"inputs":[],"name":"getMessages","outputs":[{"name":"","type":"bytes32[]"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"getBlogName","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"message","type":"bytes32"}],"name":"addPost","outputs":[],"payable":false,"type":"function"},{"inputs":[{"name":"title","type":"bytes32"}],"payable":false,"type":"constructor"}]');
BlogContract = web3.eth.contract(abi);
// Extracted from myContract.address
blog = BlogContract.at('0x69c99b6a9dacb80df84d1fed7ea1555b802f09b8');

$(document).ready(function() {
    $container = $('#messages');
    getPosts();
});

function getPosts() {
    posts = blog.getMessages.call();

    // Print them
    $container.html('');
    for (var i = 0; i < posts.length; i++) {
        $container.append('<li>'+web3.toAscii(posts[i])+'</li>');
    }
}

function addPost() {
    $message = $('#message');
    blog.addPost($message.val(), {from: web3.eth.accounts[0]}, function() {
        alert('Success!');
        $message.val('');
        getPosts();
    })
}
{% endhighlight%}

Note:

Difference between contract.getMessages() and contract.getMessages.call()

call() is for read operations, doesn't cost gas and doesn't write in the blockchain.

Calling a method directly, like getMessages(), is for writing in the blockchain and costs gas.

Read more about [gas](https://ethereum.stackexchange.com/questions/3/what-is-meant-by-the-term-gas){:target="_blank"}

### Running

Run the website http-server -a localhost -p 8000 -c-1 ./

Download the full code at [github](https://github.com/Nytyr/Ethereum-Dapp-Blog){:target="_blank"}