title:
  DWR(Direct Web Remoting)应用
date:
  2016/1/25 22:31
categories: JAVA
tags: [Java,DWR,应用]
---
![image](http://ww4.sinaimg.cn/large/7c2c72d3gw1f0c575bmymj20630583yk.jpg)
# DWR (Direct Web Remoting) -- 一种用于改善web页面与Java类交互的远程服务器端Ajax开源框架
最近做项目要用到后端向客户端推送数据的功能，普通的http请求都是客户端发出请求，服务端对其进行响应，然后请求结束，是无法满足很多实时需求较高的应用的，于是就去搜索了一下相关的技术。发现采用实时通讯方案，常见的有
> * 轮询 即客户端通过一定的时间间隔以频繁请求的方式向服务器发送请求，来保持客户端和服务器端的数据同步。
> * 长连接 即HTML5中的一种新的协议：Websocket 实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯。

由此可见，长连接是一个很好的选择，但是对于Websocket，其没有DWR框架成熟，而且是HTML5的协议，部分浏览器并不支持，所以还是选择DWR来实现推送功能。但是HTML5是趋势，Websocket也是一个很值得学习的技术。
> * DWR官方下载地址：[DWR Download](http://directwebremoting.org/dwr/downloads/index.html )
> * DWR中文手册：[DWR.pdf](http://www.niubilities.com/dwr/DWR.pdf ) (推荐)
> * Maven:
```xml
<dependency>
	<groupId>org.directwebremoting</groupId>
	<artifactId>dwr</artifactId>
	<version>3.0.1-RELEASE</version>
</dependency>
```

<!-- more -->
## DWR 快速开始
DWR可以使得Java和JavaScript中进行便捷的互动。
![image](http://ww2.sinaimg.cn/large/7c2c72d3gw1f0fnemec2jj20ff08hdgm.jpg)
#### 通过DWR使用javascript来调用java类中的方法
1. 将dwr.jar放入到项目的WEB-INF/lib中
2. 由于dwr的日志依赖于Commons Logging，所以还需要引入[commons-logging.jar](http://commons.apache.org/proper/commons-logging/download_logging.cgi )包
3. 在web.xml中添加dwr的<servlet>和<servlet-mapping>
```xml
<servlet>
	<servlet-name>dwr-invoker</servlet-name>  
	<servlet-class>org.directwebremoting.servlet.DwrServlet</servlet-class>
	<init-param>
		<param-name>debug</param-name>
		<param-value>true</param-value>
	</init-param>
</servlet>

<servlet-mapping>
	<servlet-name>dwr-invoker</servlet-name>
	<url-pattern>/dwr/*</url-pattern>
</servlet-mapping>
```
4. 在WEB-INF下创建dwr的配置文件dwr.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE dwr PUBLIC "-//GetAhead Limited//DTD Direct Web Remoting 3.0//EN" "http://getahead.org/dwr/dwr30.dtd">
<dwr>  
	<allow>  
		<create creator="new" javascript="MessagePush">  
			<param name="class" value="net.saisimon.dwr.MessagePush" />
		</create>			
	</allow>  
</dwr>
```
 dwr配置文件是决定那些class是javascript可以操作的，可以具体到某个方法上。

5. 编写net.saisimon.dwr.MessagePush类
```java
package net.saisimon.dwr;

public class MessagePush {
	public String sayHello(String name) {          
		return "Hello " + name;  
	}
}
```
6. 将dwr包中的engine.js放入Web项目中，并引入到页面中。
```html
<script src='/[YOUR-WEBAPP-CONTEXT]/dwr/engine.js'></script>
<!-- [YOUR-SCRIPT]为配置文件中javascript名称，例子中为MessagePush，该文件是dwr自动生成的，只需要引入 -->
<script src='/[YOUR-WEBAPP-CONTEXT]/dwr/interface/[YOUR-SCRIPT].js'></script>
```
7. 编写index.html
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>dwr</title>
<script type='text/javascript' src='dwr/engine.js'></script>
<script type="text/javascript" src="dwr/interface/MessagePush.js"></script>
<script type="text/javascript">
	function hello() {
		MessagePush.sayHello("Saisimon", callback);
	}
	function callback(message) {
		alert(message);
	}
</script>
</head>
<body>
	<button onclick="hello()" >测试</button>
</body>
</html>
```
8. 运行web项目，访问 http://localhost:8080/[projectName]/index.html 点击测试，应该alert出"Hello Saisimon"。这样就通过dwr使用javascript访问了java类的方法。

#### 通过DWR实现反向Ajax
 DWR可以通过三种方式实现反向Ajax
> Polling -- 轮询:定时向服务端发送请求，使客户端数据获得同步。
> Comet -- HTTP长连接服务器推送:服务器实时地将更新的信息传送到客户端，而无须客户端发出请求。(HTML5定义了Websocket协议来来取代Comet成为服务器推送的方法)
> Piggyback -- 服务端数据有更新时会等待客户端的请求，更新的数据会随着下一次请求一同发送到客户端。
 
 这三种方式都各有千秋，Polling和Comet的实时性比Piggyback要高，但是需要额外的网络连接。Polling对于服务端的开销很大。所以具体使用那种还是要看自己项目的需求。

1. 要想使用DWR实现反向Ajax，需要在web.xml中的DWR Servlet里添加<init-param>参数
```xml
<init-param>
	<param-name>activeReverseAjaxEnabled</param-name>
	<param-value>true</param-value>
</init-param>
```

2. 修改DWR配置文件
```xml
<dwr>  
	<allow>  
		<create creator="new" javascript="MessagePush">  
			<param name="class" value="net.saisimon.dwr.MessagePush"/> 
		</create>
		<create creator="new" javascript="TestPush">
			<param name="class" value="net.saisimon.dwr.TestPush"/>
		</create>   
	</allow>  
</dwr>
```

3. 编写TestPush与MessagePush类
```java
//MessagePush.java
package net.saisimon.dwr;

import org.directwebremoting.ScriptSession;
import org.directwebremoting.WebContextFactory;

public class MessagePush {
	//推送到指定用户
	public void push(String username) {
		ScriptSession scriptSession = WebContextFactory.get().getScriptSession();
		scriptSession.setAttribute("username", username);
		System.out.println("push : " + username);
	}
}

//TestPush.java
package net.saisimon.dwr;

import org.directwebremoting.Browser;
import org.directwebremoting.ScriptSession;
import org.directwebremoting.ScriptSessionFilter;
import org.directwebremoting.ScriptSessions;

public class TestPush {
	//推送message到指定用户username
	public void sendMessageAuto(String username, String message){
		final String autoMessage = message;
		final String name = username;
		Browser.withAllSessionsFiltered(new ScriptSessionFilter() {
			@Override
			//对推送的的用户进行过滤
			public boolean match(ScriptSession session) {
				String uname = (String) session.getAttribute("username");
				if (uname != null && !"".equals(uname) && uname.equals(name)) {
					return true;
				} else {
					return false;
				}
			}
		}, new Runnable(){
			public void run(){
				//调用javascript的showMessage函数
				ScriptSessions.addFunctionCall("showMessage", autoMessage);
			}
		});
	}
}
```

4. 编写web页面
```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>dwr</title>
<script type='text/javascript' src='http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js'></script>
<script type='text/javascript' src='dwr/engine.js'></script>
<script type='text/javascript' src='dwr/util.js'></script>
<script type="text/javascript" src="dwr/interface/MessagePush.js"></script>
<script type="text/javascript">
	$(function() {
		//开启反向Ajax
		dwr.engine.setActiveReverseAjax(true);
		dwr.engine.setNotifyServerOnPageUnload(true);
	});
	//通过该方法与后台交互，确保推送时能找到指定用户  
	function push() {
		var username = $("#username").val();
		MessagePush.push(username);
	}
	//推送信息  
	function showMessage(autoMessage) {
		var msg = $("<div>"+ autoMessage +"</div>");
		$(".list").append(msg);
	}
</script>
</head>
<body>
	<div>
		<input type="text" name="username" id="username" />
		<button onclick="push()" >提交</button>
	</div>
	<div>服务器推送信息列表</div>
	<div class="list">
	</div>
</body>
</html>

<!-- msg.html -->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>dwr</title>
<script type='text/javascript' src='http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js'></script>
<script type='text/javascript' src='dwr/engine.js'></script>
<script type='text/javascript' src='dwr/util.js'></script>
<script type='text/javascript' src='dwr/interface/TestPush.js'></script>
<script type="text/javascript">
	function pushMessage() {
		var msg = $("#message").val();
		var username = $("#username").val();
		//调用TestPush类中的sendMessageAuto方法
		TestPush.sendMessageAuto(username,msg);
	}
</script>
</head>
<body>
	<div>
		username: <input type="text" name="username" id="username" />
		message: <input type="text" name="message" id="message" />
		<br />
		<input type="button" value="Send" onclick="pushMessage()" />
	</div>
</body>
</html>
```

5. 测试
> 运行web项目，访问 http://localhost:8080/[projectName]/index.html ，输入任意username，例如saisimon，提交到后台，后台打印出"push : saisimon"。
> 访问 http://localhost:8080/[projectName]/msg.html ，输入刚才的username与想要推送的message，例如"hello"，然后提交。
> 查看index.html中的推送信息列表，即可看到刚才推送的"hello"信息。
> 新建一个页面，依旧访问 http://localhost:8080/[projectName]/index.html ，输入另一个username，例如java，提交到后台，后台打印出"push : java"。
> 在msg.html里输入java和想要推送的message，例如"hi"，然后提交。
> 查看两个index.html中的推送信息列表，可以发现输入java的页面里有了推送的"hi"信息，但是输入saisimon的页面没有变化。这就可以实现将信息准确的推送到指定用户的页面中了。

## DWR实现原理
客户端第一次请求 http://localhost:8080/[projectName]/index.html 时，dwr会先初始化，根据默认的defaults.properties和dwr.xml，获得系统参数。
当客户端请求 /[YOUR-WEBAPP-CONTEXT]/dwr/interface/[YOUR-SCRIPT].js 时，dwr会拦截请求，InterfaceHandler会根据scriptName和servletPath来创建[YOUR-SCRIPT].js文件。这就是为什么可以不需要自己写这个js文件的原因，dwr会根据用户写的dwr.xml来自动生成该文件。
```javascript
if (typeof dwr == 'undefined' || dwr.engine == undefined) throw new Error('You must include DWR engine before including this file');

(function() {
  if (dwr.engine._getObject("MessagePush") == undefined) {
    var p;
    
    p = {};

    /**
     * @param {class java.lang.String} p0 a param
     * @param {function|Object} callback callback function or options object
     */
    p.sayHello = function(p0, callback) {
      return dwr.engine._execute(p._path, 'MessagePush', 'sayHello', arguments);
    };
    
    dwr.engine._setObject("MessagePush", p);
  }
})()
```
当运行MessagePush.sayHello方法时，会执行engine.js的_execute方法，engine.js会发送请求 http://localhost:8080/index/dwr/call/plaincall/MessagePush.sayHello.dwr ，dwr统一使用UrlProcessor来处理请求。根据不同的url来选择不同的Handler，/call/plaincall/ 对应的Handler为PlainCallHandler。具体url对应的handler都写在defaults.properties文件中。
```java
handler.handle(request, response);
```
handler会根据request得到Calls，然后通过java反射执行请求的java方法，获得Replies。
最后通过java调用javascript的回调函数，将Replies中的数据输出到客户端，engine.js根据返回调用callback函数。
 
## 参考资料
* [DWR官网](http://directwebremoting.org/dwr/index.html )
* [DWR中文手册](http://www.niubilities.com/dwr/DWR.pdf )
