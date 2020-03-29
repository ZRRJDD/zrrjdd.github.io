# WebSocket




# 使用背景。

> 最近项目需求，想自动完成 jupyter文件运行，以及Jupyter转成html，然后一列的文件复制删除等操作。想通过前端请求 服务器后台执行jupyter 以及将jupyter转成html。
难点：
- 后台执行jupyter文件以及转成html需要5分钟以后，http请求 是 request-response 一一对应的，而且一个http请求是有时间限制的。 等后台运行完，请求已经失效了。想通过服务器向前端推送消息

# WebSocket消息推送

## 一、背景

HTTP协议的无状态和被动性，使得B/S架构的服务器主动推送消息给浏览器比较困难，而通用的解决方案又有各种各样的问题，比如：ajax轮询会有很多无用的请求，浪费宽带；基于Flash的消息推送又有Flash支持不好，无法自动穿越防火墙等问题...

WebSocket就是在这种请求下出现的一个协议。

## 二、HTTP协议特点

B/S 架构的系统多使用HTTP协议，HTTP协议的特点：

### 1、简单快速
客户端向服务器请求服务时，只需要转送请求方法和罗静。请求方法常用的是GET,HEAD,POST。每种方法都规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP协议服务器的程序规模小，因而通信速度快。
### 2、灵活

HTTP允许传输任意类型的数据对象。在传输的类型由Content-Type加以标记。
### 3、无状态
即无状态协议。这指的是，HTTP协议不对请求和响应之间的通信状态进行保存。所以使用HTTP协议，每当有新的请求发送，就会有对应的新的响应产生，这样做的好处是更快的处理大量的事物，确保协议的可伸缩性

然而，随着时间的推移，人们发现静态的HTML着实无聊而乏味，增加动态生成的内容才会令Web应用程序变得更加有用。于是乎，HTML的语法在不断膨胀，其中最重要的是增加了表单（Form）；客户端也增加了诸如脚本处理、DOM处理等功能；对于服务器，则相应的出现了CGI（Common Gateway Interface）以处理包含表单提交在内的动态请求。

在这种客户端与服务器进行交互的Web应用程序出现之后，HTTP无状态的特性严重阻碍了这些交互式应用程序的实现，毕竟交互式需要承前启后的，简单的购物车程序也要知道用户在之前选择了什么商品。于是两种用于保持HTTP状态的技术出现了Session、Cookie

### 4、持久连接

HTTP协议初始版本中，每进行一次HTTP通信就要断开一个TCP连接。

早期这么做的原因是HTTP协议产生于互联网，因此服务器需要同时处理面向全世界十万、上百万客户端的网页访问，但每个客户端（即浏览器）与服务器之间交换数据的间歇性较大（即传输具有突发性、瞬时性）,并且网页浏览的联想性、发散性导致两次传送的数据关联性很低，如果安装上面的方式则需要再服务器端开的进程和句柄数据都是不可接受的，大部分通道实际上会很空闲、无端占用资源。因此HTTP的设计者有意利用这种特点将协议设计为请求时建立连接、请求完释放连接，以尽快将资源释放出来服务其他客户端。

但是当浏览器请求一个包含多张图片的HTML页面时，会增加通信量的开销。为了解决这个问题，HTTP/1.1 出现了持久连接（HTTP keep-alive）方法。其特点是，只要任意一端没有明确提出断开连接，则保持TCP连接状态，在请求首都字段中Connection：keep-alive即表明使用持久连接。

这样一来，客户端和服务器之间的HTTP链接就会被保持，不会断开（超过Keep-Alive规定的时间，意外断电等情况除外），当客户端发送另外一个请求时，就是用这条已经建立的连接。


### 5、支持B/S及C/S模式

## 三、消息推送方案

### 3.1、HTTP服务器无法主动推送消息

HTTP的生命周期通过Request来接地昂，也就是一个Request，一个Response，在HTTP1.0中， 一个http请求就结束了。

在HTTP1.1 中进行了改进，添加了一个keep-alive，就是说，在一个http连接中，可以发送多个Request，可以接收多个Response。但是HTTP中Request和Response是成对存在的。而且这个response是被动的，不能主动发起。

客户端通过浏览器发出一个请求，服务器端接受到这个请求后进行处理并将结果返回给客户端，客户端浏览器将信息呈现出来。

这种机制对于信息变化不是特别频繁的应用可以良好的支撑，但对于实时要求高、海量并发的应用来说显得捉襟见肘，在当前业务移动互联网蓬勃发展的趋势下，高并发与用户实时响应是Web经常面临的问题。比如金融证券的实时信息、Web导航应用中的地理位置获取，社交网络的实时消息推送。

### 3.2 实现服务器主动推送消息

常见的解决方案：

#### 1.Ajax 轮询
其原理简单易懂，就是浏览器定时想服务器发送Ajax请求，询问服务器是否有新信息。

但他的问题也是很明显的：当客户端以固定频率想服务器发送请求时，服务器端的数据可能并没有更新，带来很多无畏的请求，浪费宽带，效率低下。

适于小型应用。

#### 2、Flash Socket
AdobeFlash通过自己的Socket实现完成数据转换，再利用Flash暴露出相应的接口给JavaScript调用。从而达到实时传输目的。此方式比轮询要高效，且因为Flash安装率高，应用场景广泛。

但是移动互联网上Flash的支持并不好：IOS系统中无法支持Flash，Android虽然支持Flash但世界的使用效果差强人意，且对移动设备的硬件配置要求比较高。2012年Adobe官方宣布不再支持Android4.1+系统，宣告了Flash在移动终端上的死亡。

#### 3、长轮询（long poll），长连接

keep-alive Connection是指在一次TCP连接中完成多个HTTP请求，但是对每个请求仍然要单独发HTTP header； 长轮询是指从客户端（一般是指浏览器）不断主动的向服务器发HTTP请求查询是否有新数据。这两种模式有一个共同的缺点，就是除了真正的数据部分之外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。它们建立的“长连接”都是伪长连接，只不过好处是不需要对现在的HTTP server和浏览器架构做修改就能实现。

## 四、WebSocket简介
### 1、什么是WebSocket？

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器域服务器全双工（full-duples）通信-- 允许服务器主动发送消息给客户端。

### 2、实现原理

在实现websocket连接过程中，需要通过浏览器发出websocket连接请求，然后服务器发出响应，这个过程通常叫做：“握手”。在Websocket的API中，浏览器和服务器只需要做一个握手动作即可。然后，浏览器和服务器之间就形成了一个快速通道。两者之间就直接可以数据互相传输。

![Scoket通信模型](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492333_20191103111350941_136500547.png)

### 3、WebSocket 特点
- 建立在TCP协议之上，服务器端的实现比较容易
- 与HTTP协议有良好的兼容性。默认端口号是80和443，并且采用HTTP协议，因此握手时不容易屏蔽，能通过各种HTTP代理服务器
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送问题，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是ws（如果加密，则为wss），服务器网址就是URL。

### 4、WebSocket的优势
- 1.是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可互相主动请求。而HTTP长连接基于HTTP，是传统的客户端对服务器发起请求的模式。
- 2.HTTP长连接中，每次数据交换处理镇长的数据部分外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。WebSocket协议通过第一个request请求建立TCP连接后，之后的数据交换都不要发送HTTP header。


# WebSocket示例

## 1、配置POM文件

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>${spring.version}</version>
</dependency>
```

## 2、WebSocket服务端具体实现

### 2.1 创建WebSocket的处理类

**MyWebSockerHandler.java**

```java
package cn.com.lowrisk.lrhg.webSocket;

import cn.com.lowrisk.lrhg.common.utils.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.socket.*;
import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

/**
 * webSockerHandler 处理器
 */
public class MyWebSockerHandler implements WebSocketHandler {

    /**
     * 日志对象
     */
    protected Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 用于保存用户的session
     */
    private static ConcurrentMap<String,WebSocketSession> webSocketMap = new ConcurrentHashMap<>();

    /**
     * 建立连接
     * @param webSocketSession
     * @throws Exception
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession webSocketSession) throws Exception {
        logger.info("connect websocket success.......");
        String loginName = webSocketSession.getUri().getQuery().split("=")[1];
        webSocketMap.put(loginName,webSocketSession);
    }

    @Override
    public void handleMessage(WebSocketSession webSocketSession, WebSocketMessage<?> webSocketMessage) throws Exception {
        logger.info("receive websocket message.......");
        MyWebSocketMessage message = new MyWebSocketMessage(MyWebSocketMessage.MSG_TYPE_HEART,"发送给客户端消息",true,null);
        WebSocketMessage resp = new TextMessage(message.toJSONString());
        webSocketSession.sendMessage(resp);
    }

    @Override
    public void handleTransportError(WebSocketSession webSocketSession, Throwable throwable) throws Exception {

    }

    /**
     * 关闭连接
     * @param webSocketSession
     * @param closeStatus
     * @throws Exception
     */
    @Override
    public void afterConnectionClosed(WebSocketSession webSocketSession, CloseStatus closeStatus) throws Exception {
        logger.info("connect websocket closed.......");
        String loginName = webSocketSession.getUri().getQuery().split("=")[1];
        WebSocketSession currentWebSocketSession = getCurrentWebSocketSession(loginName);
        if (currentWebSocketSession != null){
            currentWebSocketSession.close();
            webSocketMap.remove(loginName);
        }
    }

    @Override
    public boolean supportsPartialMessages() {
        return false;
    }

    private WebSocketSession getCurrentWebSocketSession(String loginName){
        if (StringUtils.isBlank(loginName)){
            return null;
        }
        return webSocketMap.get(loginName);
    }

    public void sendMessage(String loginName,WebSocketMessage<?> webSocketMessage) throws IOException {
        getCurrentWebSocketSession(loginName).sendMessage(webSocketMessage);
    }

    public void sendAllMessage(WebSocketMessage<?> webSocketMessage) throws IOException {
        this.webSocketMap.values().stream().forEach(webSocketSession -> {
            try {
                webSocketSession.sendMessage(webSocketMessage);
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }

}


```
处理类就是处理：连接开始、关闭、处理消息等方法

### 2.2 创建握手（handshake）接口/拦截器

**HandshakeInterceptor.java**

```java
package com.cuit.secims.mw.ws;

import java.util.Map;

import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;



public class HandshakeInterceptor extends HttpSessionHandshakeInterceptor{

	
	// 握手前
	@Override
	public boolean beforeHandshake(ServerHttpRequest request,
			ServerHttpResponse response, WebSocketHandler wsHandler,
			Map<String, Object> attributes) throws Exception {
		
		System.out.println("++++++++++++++++ HandshakeInterceptor: beforeHandshake  ++++++++++++++"+attributes);
		
		return super.beforeHandshake(request, response, wsHandler, attributes);
	}


	
	// 握手后
	@Override
	public void afterHandshake(ServerHttpRequest request,
			ServerHttpResponse response, WebSocketHandler wsHandler,
			Exception ex) {
		
		
		System.out.println("++++++++++++++++ HandshakeInterceptor: afterHandshake  ++++++++++++++");

		
		super.afterHandshake(request, response, wsHandler, ex);
	}
	
}

```

**这个主要作用是可以在握手之前做一些事情，把所有需要的东西放入到Attributes里面，然后可以在WebSocketHandler的session中，取到相应的值，具体可参考HTTPSessionHandshakeInterceptor，这里也可以实现HandshakeInterceptor接口。**

### 2.3 封装消息类

**MyWebSocketMessage.java**

```java
package cn.com.lowrisk.lrhg.webSocket;


import com.alibaba.fastjson.JSON;

/**
 * WebSocket消息类
 */
public class MyWebSocketMessage {

    public static final String MSG_TYPE_HEART = "0";
    public static final String MSG_TYPE_ALL = "1";
    public static final String MSG_TYPE_BUSINESS = "2";

    private String msgType;
    private String msg;
    private boolean success = false;
    private Object data;

    public MyWebSocketMessage(){

    }

    public MyWebSocketMessage(String msgType,String msg,boolean success,Object data){
        this.msgType = msgType;
        this.msg = msg;
        this.success = success;
        this.data = data;
    }

    public static String getMsgTypeHeart() {
        return MSG_TYPE_HEART;
    }

    public static String getMsgTypeAll() {
        return MSG_TYPE_ALL;
    }

    public static String getMsgTypeBusiness() {
        return MSG_TYPE_BUSINESS;
    }

    public String getMsgType() {
        return msgType;
    }

    public void setMsgType(String msgType) {
        this.msgType = msgType;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public String toJSONString() {
        return JSON.toJSONString(this);
    }
}


```

## 3、注册处理类及握手处理。

**这里有两种实现方式：**
- 1.通过注解方式。创建一个类实现注册
- 2.使用xml配置文件方式

### 3.1 注解方式：创建MyWebSocketConfig

```java
package cn.com.lowrisk.lrhg.webSocket;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

/**
 * WebSocket配置类
 */

@Configuration
@EnableWebMvc
@EnableWebSocket
public class MyWebSocketConfig extends WebMvcConfigurerAdapter implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
//        注册handler
        registry.addHandler(myWebSocketHandler(),"/a/websocket").addInterceptors(new HttpSessionHandshakeInterceptor()).setAllowedOrigins("*");
        registry.addHandler(myWebSocketHandler(),"/a/sockjs/websocket/info").addInterceptors(new HttpSessionHandshakeInterceptor()).withSockJS();
    }

    @Bean
    public WebSocketHandler myWebSocketHandler(){
        return new MyWebSockerHandler();
    }

}


```

**注意：不要忘记在springmvc的配置文件中配置对此类的自动扫描**

```xml
<context:component-scan base-package="cn.com.lowrisk" />
```

### 3.2 xml配置方式

**spring-websocket.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
			http://www.springframework.org/schema/beans 
			http://www.springframework.org/schema/beans/spring-beans-4.0.xsd 
			http://www.springframework.org/schema/mvc 
			http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd 
			http://www.springframework.org/schema/context 
			http://www.springframework.org/schema/context/spring-context-4.0.xsd 
			http://www.springframework.org/schema/aop 
			http://www.springframework.org/schema/aop/spring-aop-4.0.xsd 
			http://www.springframework.org/schema/tx 
			http://www.springframework.org/schema/tx/spring-tx-4.0.xsd 
			http://www.springframework.org/schema/websocket
        	http://www.springframework.org/schema/websocket/spring-websocket-4.0.xsd">
			
			
			
			<!-- websocket处理类 -->
			<bean id="myHandler" class="com.cuit.secims.mw.ws.MyWebSocketHandler"/>
			
			<!-- 握手接口/拦截器 -->
			<bean id="myInterceptor" class="com.cuit.secims.mw.ws.HandshakeInterceptor"/>

			<websocket:handlers >
			    <websocket:mapping path="/websocket" handler="myHandler"/>
			    <websocket:handshake-interceptors>
			    	<ref bean="myInterceptor"/>
			    </websocket:handshake-interceptors>
			</websocket:handlers>
			
			<!--  注册 sockJS -->
			<websocket:handlers>
				<websocket:mapping path="/sockjs/websocket" handler="myHandler"/>
			    <websocket:handshake-interceptors>
			    	<ref bean="myInterceptor"/>
			    </websocket:handshake-interceptors>
			    <websocket:sockjs />
			</websocket:handlers>	
		
</beans>

```

其中`<websocket:sockjs />` 就是对sockJS的注册方式
**注意：xml的namespace 中要添加spring对websocket的支持**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
			http://www.springframework.org/schema/beans 
			http://www.springframework.org/schema/beans/spring-beans-4.0.xsd 
			http://www.springframework.org/schema/websocket
        	http://www.springframework.org/schema/websocket/spring-websocket-4.0.xsd">

```


## 4、客户端实现

引入js
```html
<!-- 引入 sockJS  -->
<script type="text/javascript" src="sockjs.min.js" ></script>
```
**客户端实现代码**
```js
var websocket;

createWebSocket();

function createWebSocket() {
    var wssUrl = (window.location.protocol == 'https:'?'wss:':'ws:')+"//"+window.location.host+"${ctx}/websocket?loginName=${fns:getUser().loginName}"
    // 首先判断是否 支持 WebSocket
    try {
        if('WebSocket' in window) {
            websocket = new WebSocket(wssUrl);
        } else if('MozWebSocket' in window) {
            websocket = new MozWebSocket(wssUrl);
        } else {
            websocket = new SockJS(window.location.protocol+"//"+window.location.host+"${ctx}/sockjs/websocket?loginName=${fns:getUser().loginName}");
        }
        initEventHandler()
    } catch (e) {
       
    }

}

//封装websocket 的 监听接口函数
function initEventHandler(){
    // 打开时
    websocket.onopen = function(evnt) {
        console.log("websocket.onopen");
        console.info("open："+ new Date());
    };
    // 处理消息时
    websocket.onmessage = function(evnt) {
        var result = JSON.parse(evnt.data);
        //处理服务器发送给客户端的消息
    };

    websocket.onerror = function(evnt) {
        console.log(evnt);
        console.log("  websocket.onerror  ");
    };

    websocket.onclose = function(evnt) {
        console.log("  websocket.onclose  ");
        console.info("close："+ new Date());
    };
}

    
```

# 至此WebSocket Demo实现完毕，但是在本地 测试都没有问题。但是放到线上以后，有各种问题。接下来就介绍一下，线上出现的问题。

首先说一下本地和线上环境的不同点：
- 1.本地 http ，线上https
- 2.本地 Tomcat +项目 ，线上 nginx + Tomcat + 项目


# Q1:在域名Https 使用ws 访问报错。


## 1、ws 和wss 是什么 ？

WebSocket使用：`ws` 或`wss` 的统一资源标识符，类似`HTTP` 或`HTTPS`，其中`wss` 表示在TLS之上的WebSocket，相当于HTTPS，如：
```
http -> new WebSocket('ws://xxx')
https -> new WebSocket('wss://xxx')
```
**也就是说https应该使用wss协议做安全链接，且wss下不支持IP地址的写法，写成域名方式**

经过测试，部分报错的浏览器的确是因为这个原因导致的代码异常，即在https下把ws换成wss请求即可，看到这里心细的也许会发现，是**部分浏览器**，实际上浏览器上并没有严格的限制http下一定使用ws，而不能使用wss，经过测试http协议下同样可以使用wss协议连接，https下同样也能使用ws连接，那么出问题的是那一部分呢？

- 1.Firefox环境下https不能使用ws连接
- 2.Chrome内核版本低于50的浏览器是不允许https下使用ws链接
- 3.Firefox环境下https使用wss链接需要安装证书。

默认情况下，WebSocket的ws协议的端口号是：80 ；运行在TLS之上后，wss协议默认端口号是443,。其实说白了，wss是ws基于SSL的安全传输，与HTTPS一样的的道理。
如果你的网站是HTTPS协议的，那你就不能使用`ws:\\`了，浏览器会block掉链接，和HTTPS下允许HTTP请求一样，如下图：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492334_20191104082144866_129.png)

```
Mixed Content: The page at 'https://domain.com/' was loaded over HTTPS, but attempted to connect to the insecure WebSocket endpoint 'ws://x.x.x.x:xxxx/'. This request has been blocked; this endpoint must be available over WSS.
```
这种情况，毫无疑问我们就需要使用 `wss:\\` 安全协议了，我们是不是简单的把 `ws:\\` 改为 `wss:\\` 就行了？那试试呗。

改好了又报错：
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492335_20191104082347201_2344.png)

```
VM512:35 WebSocket connection to 'wss://IP地址:端口号/websocket' failed: Error in connection establishment: net::ERR_SSL_PROTOCOL_ERROR
```
很明显 SSL 协议错误，说明就是证书问题了。记着，这时候我们一直拿的是` IP地址 + 端口号` 这种方式连接 WebSocket 的，这根本就没有证书存在好么，况且生成环境你也要用 `IP地址 + 端口号` 这种方式连接 WebSocket 吗？肯定不行阿，要用域名方式连接 WebSocket 阿。

## 2、Nginx 配置域名支持 WSS

不用废话，直接在配置 HTTPS 域名位置加入如下配置：
```json
location /websocket {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```


# Q2:Spring的WebSocket访问时403

解决方案：
- 1.xml方式
- 2.代码方式

## 1.xml方式
在配置WebSocket的xml语句中：
```xml
<websocket:handlers allowed-origins="*">
</websocket:handlers>
```

## 2.代码方式

在WebSocketConfig 配置类中 ：
注册完handler，然后设置 allowedOrigins
```java
registry.addHandler(myWebSocketHandler(),"/a/websocket").addInterceptors(new HttpSessionHandshakeInterceptor()).setAllowedOrigins("*");
```
![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1585492335_20191104082932798_9566.png)


# Q3:WebSocket 每1分钟自动断开连接

**原因：使用了nginx服务，nginx配置：**
proxy_read_timeout(Default: 60s),如果一直没有数据传输，连接会在过了这个时间之后自动关闭
参考：[http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout)
Defines a timeout for reading a response from the proxied server. The timeout is set only between two successive read operations, 
not for the transmission of the whole response. If the proxied server does not transmit anything within this time, the connection is closed.


解决方案：
- 可以通过心跳保持连接

参考：[http://nginx.org/en/docs/http/websocket.html](http://nginx.org/en/docs/http/websocket.html)
the proxied server can be configured to periodically send WebSocket ping frames to reset the timeout and check if the connection is still alive.

**JS实现代码**

```js
var websocket;
var lockReconnect = false;

createWebSocket();

function createWebSocket() {
    var wssUrl = (window.location.protocol == 'https:'?'wss:':'ws:')+"//"+window.location.host+"${ctx}/websocket?loginName=${fns:getUser().loginName}"
    // 首先判断是否 支持 WebSocket
    try {
        if('WebSocket' in window) {
            websocket = new WebSocket(wssUrl);
        } else if('MozWebSocket' in window) {
            websocket = new MozWebSocket(wssUrl);
        } else {
            websocket = new SockJS(window.location.protocol+"//"+window.location.host+"${ctx}/sockjs/websocket?loginName=${fns:getUser().loginName}");
        }
        initEventHandler()
    } catch (e) {
        //重复链接
        reConnect()
    }

}

//封装websocket 的 监听接口函数
function initEventHandler(){
    // 打开时
    websocket.onopen = function(evnt) {
        console.log("websocket.onopen");
        console.info("open："+ new Date());
        //心跳检测重置
        heartCheck.reset().start();
        
        $.getJSON("${ctx}/jupyter/jupyterReportTask/allNoFinishTask",function (data) {
            console.log("首次连接获取：")
            console.log(data)
        });
        
        
    };
    // 处理消息时
    websocket.onmessage = function(evnt) {
        var result = JSON.parse(evnt.data);
        //如果获取到信息，心跳检测重置
        heartCheck.reset().start();
        if (result['msgType'] == '0'){
            // console.info("收到服务器消息：" + new Date())
        } else if (result['msgType'] == '1'){
            console.log("全部未完成任务");
            console.info(result.data)
        } else if (result['msgType'] == '2'){
            console.log(result);
            if (result.success){
                layer.alert(result.msg,
                    {"title":"<b>成功信息</b>",offset:'rb',anim: 2, btn: ['立即查看']},
                    function(index, layero){
                        layer.close(index);
                        // updateIframe(baseUrl+result.htmlPath);
                        // getHistoryFile(result.filePath)
                    }
                );
            } else {
                layer.alert("<span style='color: red;'>"+result.msg+"</span>",{"title":"<span style='color: red;'><b>错误提示</b></span>",offset:'rb',anim: 2});
            }
        }


    };

    websocket.onerror = function(evnt) {
        console.log(evnt);
        console.log("  websocket.onerror  ");
        reConnect()
    };

    websocket.onclose = function(evnt) {
        console.log("  websocket.onclose  ");
        console.info("close："+ new Date());
        reConnect()
    };
}

function reConnect() {
    if (lockReconnect){
        return;
    }
    lockReconnect = true;
    setTimeout(function () {
        console.info("尝试重新链接...." + new Date());
        createWebSocket();
        lockReconnect = false;
    },5000)
}

var heartCheck = {
    timeout:5000,
    timeoutObj:null,
    serverTimeoutObj:null,
    reset:function () {
        clearTimeout(this.timeoutObj);
        clearTimeout(this.serverTimeoutObj);
        return this;
    },
    start:function () {
        var self = this;
        this.timeoutObj = setTimeout(function () {
            websocket.send("HeartBeat:"+new Date());
            // console.info("客户端发送心跳："+ new Date());
            self.serverTimeoutObj = setTimeout(function () {
                websocket.close()
            },self.timeout);
        },self.timeout)
    }
}
```

# 参考链接

- [Spring MVC 自学杂记（四） -- Spring+SpringMVC+WebSocket](https://blog.csdn.net/mybook201314/article/details/70173674)
- [WebSocket实现Java后台消息推送](https://www.cnblogs.com/freud/p/8397934.html)
- [项目实例 -- Websocket消息推送](https://blog.csdn.net/w1992wishes/article/details/79583543)
- [WebSocket 结合 Nginx 实现域名及 WSS 协议访问](https://www.cnblogs.com/mafly/p/websocket.html)
- [http/https与websocket的ws/wss的关系](https://blog.csdn.net/Garrettzxd/article/details/81674251)
- [spring的websocket访问时403](https://blog.csdn.net/Y_mmmmmmm/article/details/53470353)
- [websocket自动断开连接问题](https://blog.csdn.net/iterJiaY/article/details/52637251)