#  一：什么是webSocket ？

　　webSocket是HTML5出的新协议，2011年成为国际标准。所有浏览器都已经支持了WebSocket协议支持，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。（兼容性：ie10）

## 二：和传统的http请求比较

　　http：一个request只能有一个response。而且这个response也是被动的，不能主动发起。

　　webSocket：服务端就可以主动推送信息给客户端

![img](https://img-blog.csdnimg.cn/img_convert/7ac9a6ed83dca6314b5fc98aee179b78.png)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

​    *在websocket请求的请求头中会像cors一样加入origin字段，服务端可以根据这个字段来判断是否通过该请求*

（6）协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

> ```
> ws://server.example.com:80/path
> ```

## 三：和轮询比较

### 3.1 长链接

初始化时候建立连接，一直不关闭。

​	服务器和客户端能以最快速度发送数据。

​	服务器能主动发送数据

​	<font color=red>长期占用网络资源</font>

​	<font color=red>服务器操作很敏感</font>

### 3.2 轮询

发起链接-->发送数据-->获取数据-->关闭

​	不会长期占用网络资源

​	方便服务器重启

​	<font color=red>每次请求都需要先建立连接</font>

​	<font color=red>服务器不能主动发送数据</font>



## 四：webSocket握手

**由浏览器发送的webSocket请求**：

1. GET /chat HTTP/1.1

2. Host: server.example.com

3. Connection: Upgrade

   *表示要求协议“升级”*

4. Upgrade: websocket

   *表示要“升级”成WebSocket协议。*

5. Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==

   *标头，自动生成，不可更改，这有助于确保服务器不接受来自非WebSocket客户端的连接。*

6. Sec-WebSocket-Protocol: chat,superchat

   *子协议名，可用户定义，在创建时第二个参数定义*

7. Sec-WebSocket-Version: 13

8. Origin: http://example.com





1. 　　HTTP/1.1 101 Switching Protocols
2. 　　Upgrade: websocket
3. 　　Connection: Upgrade
4. 　　Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
5. 　　Sec-WebSocket-Protocol: chat



## 五、客户端的 API

### 5.1 WebSocket 构造函数

WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。

> ```javascript
> var ws = new WebSocket('wss://localhost:8080?token=token');
> ```

执行上面语句之后，客户端就会与服务器进行连接。

### 5.2 webSocket.readyState

`readyState`属性返回实例对象的当前状态，共有四种。

> - CONNECTING：值为0，表示正在连接。
> - OPEN：值为1，表示连接成功，可以通信了。
> - CLOSING：值为2，表示连接正在关闭。
> - CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

下面是一个示例。

> ```javascript
> switch (ws.readyState) {
>   case WebSocket.CONNECTING:
>     // do something
>     break;
>   case WebSocket.OPEN:
>     // do something
>     break;
>   case WebSocket.CLOSING:
>     // do something
>     break;
>   case WebSocket.CLOSED:
>     // do something
>     break;
>   default:
>     // this never happens
>     break;
> }
> ```

### 5.3 webSocket.onopen

实例对象的`onopen`属性，用于指定连接成功后的回调函数。

> ```javascript
> ws.onopen = function () {
>   	ws.send('Hello Server!');
> }
> ```

如果要指定多个回调函数，可以使用`addEventListener`方法。

> ```javascript
> ws.addEventListener('open', function (event) {
>   	ws.send('Hello Server!');
> });
> ```

### 5.4 webSocket.onclose

实例对象的`onclose`属性，用于指定连接关闭后的回调函数。

> ```javascript
> ws.onclose = function(event) {
>       var code = event.code;
>       var reason = event.reason;
>       var wasClean = event.wasClean;
>       // handle close event
> };
> 
> ws.addEventListener("close", function(event) {
>       var code = event.code;
>       var reason = event.reason;
>       var wasClean = event.wasClean;
>       // handle close event
> });
> ```

### 5.5 webSocket.onmessage

实例对象的`onmessage`属性，用于指定收到服务器数据后的回调函数。

> ```javascript
> ws.onmessage = function(event) {
>       var data = event.data;
>       // 处理数据
> };
> 
> ws.addEventListener("message", function(event) {
>       var data = event.data;
>       // 处理数据
> });
> ```

注意，服务器数据可能是文本，也可能是二进制数据（`blob`对象或`Arraybuffer`对象）。

> ```javascript
> ws.onmessage = function(event){
>       if(typeof event.data === String) {
>         	console.log("Received data string");
>       }
> 
>       if(event.data instanceof ArrayBuffer){
>             var buffer = event.data;
>             console.log("Received arraybuffer");
>       }
> }
> ```

除了动态判断收到的数据类型，也可以使用`binaryType`属性，显式指定收到的二进制数据类型，可动态更改。

> ```javascript
> // 收到的是 blob 数据
> ws.binaryType = "blob";
> ws.onmessage = function(e) {
>   	console.log(e.data.size);
> };
> 
> // 收到的是 ArrayBuffer 数据
> ws.binaryType = "arraybuffer";
> ws.onmessage = function(e) {
>   	console.log(e.data.byteLength);
> };
> ```

### 5.6 webSocket.send()

实例对象的`send()`方法用于向服务器发送数据。

发送文本的例子。

> ```javascript
> ws.send('your message');
> ```

发送 Blob 对象的例子。

> ```javascript
> var file = document.querySelector('input[type="file"]').files[0];
>   ws.send(file);
>   ```

发送 ArrayBuffer 对象的例子。

> ```javascript
> // Sending canvas ImageData as ArrayBuffer
> var img = canvas_context.getImageData(0, 0, 400, 320);
> var binary = new Uint8Array(img.data.length);
> for (var i = 0; i < img.data.length; i++) {
>   	binary[i] = img.data[i];
> }
> ws.send(binary.buffer);
> ```

### 5.7 webSocket.bufferedAmount

实例对象的`bufferedAmount`属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

> ```javascript
> var data = new ArrayBuffer(10000000);
> socket.send(data);
> 
> if (socket.bufferedAmount === 0) {
>   // 发送完毕
> } else {
>   // 发送还没结束
> }
> ```

### 5.8 webSocket.onerror

实例对象的`onerror`属性，用于指定报错时的回调函数。

> ```javascript
> socket.onerror = function(event) {
>   // handle error event
> };
> 
> socket.addEventListener("error", function(event) {
>   // handle error event
> });
> ```

## 六、浏览器查看

##### <img src="C:\Users\lin\AppData\Roaming\Typora\typora-user-images\image-20210921223726614.png" alt="image-20210921223726614" style="zoom: 67%;" />

### 表格列 Frames

Data: 消息负载。 如果消息为纯文本，直接显示。 二进制操作吗显示操作码名称和代码（Continuation Frame、 Binanry Frame、 Connection Close Frame、 Ping Frame或者Pong Frame）
 Length: 消息负载的长度 以字节为单位
 Time: 收到或者发送的时间。

### 消息颜色

- 发送到服务器的文本消息为浅绿色
- 接收到的文本消息为白色
- WebSocket 操作码 为浅黄色
- 错误为浅红色

## 七、心跳检测

*心跳包：它像心跳一样每隔固定时间发一次，以此来告诉服务器，这个客户端还活着。至于这个包的内容，是没有什么特别规定的，不过一般都是很小的包，或者一个空包，还能避免自动断开。*

```
req_type: 1,
timestamp: this.getTimestamp(),
```

有2中方案，一种是客户端主动发送上行心跳包，另一种方案是服务端主动发送下行心跳包。下面主要讲一下客户端也就是前端如何实现心跳包：

心跳检测步骤：
1客户端每隔一个时间间隔发生一个探测包给服务器
2客户端发包时启动一个超时定时器
3服务器端接收到检测包，应该回应一个包
4如果客户机收到服务器的应答包，则说明服务器正常，删除超时定时器
5如果客户端的超时定时器超时，依然没有收到应答包，则说明服务器挂了

## 八、重连处理

重连步骤：
1 先进行自动重连
2 失败再让用户选择是否继续重连

## 九、案例

定义超时参数
```
sendHeartTime: function (newValue, oldValue) {
    if (newValue == 55) {
        let sendData = {
            req_type: 8,
            req_no: this.getTimestamp().toString()
        }
        this.websocketsend(JSON.stringify(sendData));
	}
},
getHeartTime: function (newValue, oldValue) {
    if (newValue == 70) {
    	this.connectPrompt();
    }
},
```

初始化：

```
initWebSocket() {
    const ws_url = `wss://aaaaaaaaaaaaa?userid=&token=&version=&platform=micro-web`;
    this.websocket = new WebSocket(ws_url);
    this.websocket.onmessage = this.websocketonmessage;
    this.websocket.onopen = this.websocketonopen;
    this.websocket.onerror = this.websocketonerror;
    this.websocket.onclose = this.websocketonclose;
},
```
连接成功：

```
websocketonopen() {
    this.sendTimer = setInterval(() => {
        this.sendHeartTime++;
        this.getHeartTime++;
    }, 1000);
    console.log(timestampToTime(this.getTimestamp()) + '=>socket已经连接' + this.websocket.readyState);
},
```
监听信息接受
```
websocketonmessage(e) {
    this.getHeartTime = 0;
    const receiveData = JSON.parse(e.data);
    if (receiveData.resp_type === 2) {
    	
    } else if (receiveData.resp_type === 3) {

    }     
 }
```

关闭连接

```
websocketonclose(e) {
        if (!this.isSelfCloseSocket) {
          console.log(timestampToTime(this.getTimestamp()) + '=>异常关闭socket, 继续重连, socketState=' + this.websocket.readyState);
          this.connectPrompt();
        }
      },
      websocketonerror() {
        console.log(timestampToTime(this.getTimestamp()) + '=>webSocket连接出错');
        if (!this.isInitiativeCloseSocket) {
          // this.connectPrompt()
        }
      },
```

重连

```
connectPrompt() {
    this.wsCloseWebSocketActively();
    if (this.reConnectSocketCount <= 5) {
        this.reConnectSocketCount++
        this.reConnectSocket();
    } else {
        Toast.clear();
        this.$dialog.confirm({
            title: '提示',
            message: '连接已断开，是否重新连接？',
        }).then(() => {
            this.reConnectSocketCount = 0;
            this.reConnectSocket();
        });
    }
},

reConnectSocket() {
        setTimeout(() => {
          Toast.loading({
            duration: 0,mask: true,message: '正在重新连接，请稍候...',
          });
          console.log(timestampToTime(this.getTimestamp()) + '=>CLOSED: webSocket重连次数=' + this.reConnectSocketCount);
          window.clearInterval(this.sendTimer);
          this.initWebSocket();
        }, 2000)
      },
```

