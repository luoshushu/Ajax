##### Ajax的全称Asynchronous JavaScript + XML(异步JavaScript和XML)。

### 什么是Ajax？

Ajax是一种技术方案，但并不是一种新技术。它依赖现有的CSS/HTML/JavaScript，而其中最核心的依赖是浏览器提供的**XMLHttpRequest**对象，是这个对象使得浏览器可以发出HTTP请求与接收HTTP响应。实现了在页面不刷新个情况下和服务器进行数据交互。



## 怎么实现？

### 栗子：

hello.json：
```
{
  "name":"luoshushu",
  "age":"25"
}
```

js:
```
//生成一个对象
var xhr = new XMLHttpRequest()
//请求类型：get
//请求地址：/hello.json(必须是http或者https开头)
//true：异步的方式。
//false：同步的方式
xhr.open('GET','/hello.json',true) 
//发送
xhr.send()
```
1、生成一个XMLHttpRequest对象。
2、初始化一个请求。
3、发送。


![请求成功](https://upload-images.jianshu.io/upload_images/5691870-e8e4c0ad9de3377e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)



### 同步的获取：
```
var xhr = new XMLHttpRequest()
xhr.open('GET','/hello.json',false)  //false同步方式
xhr.send()
var data = xhr.responseText
console.log(data) 
```

![同步获取数据](https://upload-images.jianshu.io/upload_images/5691870-8f494a50e384e54f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)


结果数据也来到了。

缺点：如果网络请求特别慢，页面就卡住一直等待数据的请求，用户就无法操作。所以我们只能用异步的方式（不推荐同步）。

### 异步的获取：

```
  <script>
    var xhr = new XMLHttpRequest()
    xhr.open('GET', '/hello.json', true)
    xhr.send()
    //监听数据，内部数据到了会触发load
    xhr.addEventListener('load', function () {
      var data = xhr.responseText
      console.log(data)
    })
  </script>
```
因为代码是从上往下执行的，数据请求的速度没比不上代码的执行速度，所以需要**监听数据**的到来。



![异步的方式](https://upload-images.jianshu.io/upload_images/5691870-f42f4b643b9c63b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)



### 常见的Ajax用法：
```
<script>
  //生成一个对象
  var xhr = new XMLHttpRequest() 
  // 请求类型：get
  // 请求地址：/hello.json(必须是http或者https开头)
  // true：为异步方式。false：为同步方式
  //GET的数据放在open的请求地址中发送。(尾部 图1)
  //注意格式（请求资源的地址？发送的数据）   
  //如：地址?username=luoshushu&passwore=123 

  xhr.open('GET', '/hello.json?username=luoshushu&passwore=123', true)
  //发送
  xhr.send()
  //监听数据
  xhr.addEventListener('load', function () {
    // status: HTTP响应状态代码
    console.log(xhr.status)
    // 如果响应状态是200到300以内，或者等于304
    // 200到300都是请求数据成功的
    // 304 重定向（缓存）
    if((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304){
      //responseText：响应文本（内容）
      var data = xhr.responseText
      console.log(data)
    }else{
      console.log('出错了')
    }
  })
  // 当内部出现错误的时候执行（可选）
  xhr.onerror = function () {console.log('连接失败')}
  // 当请求超时执行 (可选)
  xhr.ontimeout = function(){console.log('请求超时')}
  xhr.upload.onprogress = function(e) {
      //如果是上传文件，可以获取上传进度
  }

</script>

```
### 换一种写法
```
var xhr = new XMLHttpRequest()
xhr.open('GET', 'http://hello.php', true)
xhr.onreadystatechange = function(){
    if(xhr.readyState === 4) {
        if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
            //成功了
            console.log(xhr.responseText)
        } else {
            console.log('服务器异常')
        }
    }
}
xhr.onerror = function(){
    console.log('服务器异常')
}
xhr.send()
```

### POST写法：
```
 var xhr = new XMLHttpRequest() 
  xhr.open('POST', '/hello.json', true)
  //发送. psot数据要放在send里面
  xhr.send('username=luoshushu&passwore=123') 
  xhr.addEventListener('load', function () {
    if((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304){
      var data = xhr.responseText
      console.log(data)
    }else{
      console.log('出错了')
    }
  })
```

### 一些解释：

- new XMLHttpRequest() ：生成一个对象

- open：初始化一个请求。
  1、请求类型：get、post
  2、请求地址：/hello.json(必须是http或者https开头)
3、请求方式： true：为异步方式。false：为同步方式
- send：发送
- status: HTTP响应状态代码
- responseText：响应文本（内容）
- readySteat ： 交互流程（1，2，3，4）
0 － （未初始化）还没有调用send()方法 
1 － （载入）已调用send()方法，正在发送请求 
2 － （载入完成）send()方法执行完成，已经接收到全部响应内容 
3 － （交互）正在解析响应内容 
4 － （完成）响应内容解析完成，可以在客户端调用了 



![GET传过来的数据-图1](https://upload-images.jianshu.io/upload_images/5691870-5d78719f93af1a6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 封装一个Ajax:
```
/*封装一个Ajax*/
function ajax(opts){
    var url = opts.url //请求地址
    var type = opts.type || 'GET' //请求类型。默认GET
    var dataType = opts.dataType || 'json' //返回的数据类型。默认json
    var onsuccess = opts.onsuccess || function(){} //请求成功后
    var onerror = opts.onerror || function(){} //失败之后
    var data = opts.data || {} //如果用户传递数据了就选用户的，默认空
    //处理把用户传递的参数。如：用户名、密码等。
    var dataStr = []
    for(var key in data){
        dataStr.push(key + '=' + data[key])
    }
    dataStr = dataStr.join('&')
    
    // 如果类型===GET,那么用户的地址？dataStr  
    // 比如：http://api.luoshushu.com/weather.php?city=广州
    if(type === 'GET'){
        url += '?' + dataStr
    }
    //以下跟前面的“常见的Ajax用法”的解释差不多 
    var xhr = new XMLHttpRequest()
    xhr.open(type, url, true)
    xhr.onload = function(){
        if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
            //成功了
            if(dataType === 'json'){
                onsuccess( JSON.parse(xhr.responseText))
            }else{
                onsuccess( xhr.responseText)
            }
        } else {
            onerror()
        }
    }
    // 假设断网了
    xhr.onerror = onerror
    // 如果类型是POST
    if(type === 'POST'){
      // POST要在send中传递数据
        xhr.send(dataStr)
    }else{
      // GET直接调用
        xhr.send()
    }
}

/*--------------*/
/*      使用    */
/*--------------*/

ajax({
    url: 'http://api.luoshushu.com/weather.php',
    data: {
        city: '广州'
    },
    // 数据成功之后
    onsuccess: function(ret){
        console.log(ret)
    },
    // 失败之后执行
    onerror: function(){
        console.log('服务器异常')
    }
})

```


[全部代码地址](https://github.com/luoshushu/Ajax)

*PS：每个字都看完才明白,看完还不明白Ajax请点击下列文章*

[更多详细Ajax解释干货](https://segmentfault.com/a/1190000004322487)


