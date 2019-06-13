# node
node是JavaScript语言的服务器运行环境
所谓运行环境有两层意思：首先，JavaScript语言通过node在服务器上运行，在这个意义上，node有点像JavaScript虚拟机
其次node提供大量工具库，使得JavaScript语言与操作系互动（读写文件，新建子进程），node在这个意义上是JavaScript的工具库
# node内部采用goole的v8引擎，作为JavaScript语言解释器，通过自行开发的libuv，调用操作系统资源
# 安装和更新
官方提供编译好的二进制包，可以把它解压到/user/local目录下面
$ tar -xf node-someversion.tgz
然后建立符号链接，把它们加到$PATH里面的路径...
# 1.3基本用法
安装完成后，运行node.js程序，就是使用node命令读取JavaScript脚本。
当前目录的demo.js脚本文件，可以这样执行
$ node demo或$ node demo.js
使用-e参数，可以执行代码字符串 $node -e 'console.log("hellworld")'
# 1.4repl环境
在命令行键入node命令，后面没有文件名，就进入一个node.js的repl环境（读取-求值-输出）循环可以直接运行JavaScript命令
$ node
>1+1
2
如果使用参数-use_strict,则repl在严格模式下运行
repl是node与用户互动的shell，基本各种的shell都可以在里面使用
# 1.5异步操作
node采用v8引擎处理JavaScript脚本，最大的特点就是单线程运行，一次只能运行一个任务，这导致node大量异步操作
（按顺序插入任务的尾部）
由于这种特性，某一个任务的后续操作，往往采用回调函数的形式来进行定义？
node约定，如果摸个函数需要回调函数作为参数，则回调函数是最后一个参数，另外，回调函数本身的一个参数，约定为上一步传入的错误对象
callback(null,"value was true")
var callback = function (error, value) {
 if(error) {
   return console.log(error)
 }
 console.log(value);// 一个提示
}
上面代码中，callback的一个参数是error对象，第二个参数才是数据参数。因为回调函数主要用于异步操作，
当回调函数运行的时候，前期操作早就结束了，错误的执行栈早就不存在，传统的错误捕获机制对于异步操作行不通，所以只能用回调函数处理
try {
  db.User.get(userId, function(err, user) {
    if(err) {
      throw err
    }
  })
}.catch(e) {
  console.log('oh no!');
}
！上面的代码中，方法是一个异步操作，等到抛出错误时，可能它所在的代码块早就结束了，导致错误无法被捕捉,
所以node规定，一旦异步操作发生错误，就把错误对象传递到回调函数
如果没有发生错误，回调对的第一个参数就传入null。这种写法的好处，就是只要判断回调函数的第一个参数，就知道没有出错，
如果不是null，就肯定出错了，另外这样还可以层层传递错误
if (err) {
  if(!err.noPermission){ // 除了放过no permission错误意外，其他错误传给下一个回调函数
    return next(err);
  }
}
# 1.6 全局对象和全局变量
node提供以下几个全局对象，它们的模块都是调用的
gobal:表示node所在的全局环境，类似于浏览器的window对象。需要注意的是，如果在浏览器
声明一个全局变量，实际上是声明了一个全局对象属性，比如var x=1等同window.x =1但是node不是这样，至少在模块不是这样，
在模块文件中声明var x=1，该变量不是global对象的属性，（！因为模块的全局变量都是私有的其他模块都无法取到）
process：该对象表示node所处的当前进程，允许开发者与该进程互动
console：指向node内置的console模块，提供命令行环境中标准输入，输出功能
node还提供一些全局函数
setTimeout():用于在指定毫秒之后，运行回调函数。
clearTimeout():用于终止一个setTimeout的新的定时器
setInterval():用于每隔一毫秒调用回调函数
require():用来加载模块
Buffer():用于操作二进制数据
Node提供两个全局变量，都以两个下划线开头
！__filename：指向当前运行的脚本文件名
！__dirname:指向当前运行的脚本所在的目录
除此之外，还有一些对象时节上是模块内部的局部变量，指向的对象根据模块不同而不同，但是所有模块都使用，可以看做是伪全局变量，
主要为module，module.exports,exports
# 模块化结构
node.js采用的是模块化结构，按照CommonJS规范定义和使用模块。模块与文件是一一对应关系，即加载一个模块就是加载对应的
一个模块文件。
require命令用于指定加载模块，加载时可以省略脚本的后缀名
var circle = require('./circle.js');
require方法的参数是模块文件的名字。
第一种情况是参数中含有文件路径，根据当前脚本所在的目录。
第二种是参数中不含有文件路径，node到模块的安装目录，去寻找已安装的模块
var bar = require('bar');
这时候，一个模块本身就是一个目录，目录中包含多个文件，这个时候，node在package.json文件中，寻找main属性所指明的模块入口文件
{
"name":"bar",
"main":"./lib/bar.js"
}
!模块一旦被加载，就会被系统缓存。如果第二次还加载该模块，则会返回缓存中的版本，这意味着模块实际上只会执行一次，
如果希望模块多此执行，则可以让模块返回一个函数，然后多次调用该函数？
# 2.2核心模块
如果只是在服务器运行JavaScript代码，用处不大，服务器脚本语言有很多。
！node.js的用处在于，它本身提供了一系列功能模块，与操作系统互动。
核心功能魔铠列表：
http：提供http服务器功能
url：解析url
fs：与文件系统交互
querystring：解析URL的查询字符串
child_process: 新建子进程？
util：提供一系列实用小工具
path：处理文件路径
crypto：提供加密和解密功能，基本是对OpenSSL的包装
上面这些核心模块，源码都在node的lib子目录中。为了提高运行速度，它在安装时都会被编译成二进制文件，
核心模块总是最先加载，如果你自己写了一个http模块，require('http')加载的还是核心模块
# 2.3自定义模块
node模块采用CommonJS规范。
// foo.js
module.exports = function(x) {
  console.log(x);
}
上面这个模块的使用方法是
var m = require('./foo');
m("这是自定义模块"); // 打印在运行栏
module变量是整个模块文件的顶层变量，它的exports属性就是模块向外输出的接口，如果直接输出一个函数，
那么调用模块就是调用一个函数。
模块也可以输出一个对象
var out = new Object();
function p(string) {
  console.log(string);
}
out.print = p; // 把一个对象的属性赋值一个函数
modue.exports = out;
# 3.异常处理
node是单线程运行环境，一旦抛出的异常没有被捕获，会引起整个进程的崩溃。
node有三个方法，传播一个错误：
1.使用throw语句抛出一个错误对象，即抛出异常。
try {
  process.nextTick(funtion () {
    throw new Error("error");
  });
}catch (err) {
  console.log(err);
}
try {
  setTimeout(function(){
    throw new Error("error");
  },1)
} catch (err) {
  console.log(err);
}
上面的代码分别用process.nextTick和setTimeout方法，在下一轮事件循环抛出两个异常，代表异步操作抛出的错误
2.将错误对象传递给回调函数，由回调函数负责发出错误

3.通过eventemitter接口，发出一个error事件?

