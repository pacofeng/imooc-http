## 五层Network Layers
* Application: SNMP, HTTP, FTP
* Transport: TCP, UDP, port numbers
* Network: IP, routers
* Data Link: MAC, switches
* Physical: cable, RJ45

## HTTP的history
* HTTP/0.9：
  * 只有get命令
  * 没有header等描述数据
  * 服务器发送完毕就关闭TCP链接
* HTTP/1.0：
  * 增加命令，支持post，put，delete
  * 增加status code和header
  * 增加多字符集支持，多部分发送，权限，缓存
  * 服务器发送完毕就关闭TCP链接
* HTTP/1.1：
  * 持久连接，A TCP connection can be reused, saving the time to do 3 handshakes
  * 增加Pipeline，allow sending a second request before the answer for the first one is fully transmitted, lowering the latency of the communication.
  * 增加host，host different domains at the same IP address 
* HTTP/2：
  * 所有数据以二进制传输，It is a binary protocol rather than text.
  * 同一链接可以发送多个请求，不需要按照顺序，It is a multiplexed protocol. Parallel requests can be handled over the same connection, removing the order and blocking constraints
  * 头信息压缩及推送提高效率，It compresses headers and allows a server to populate data in a client cache

## TCP三次握手
防止服务端开启无用链接，避免资源浪费



HTTP报文格式
* 起始行：request line例如 GET /index.html HTTP/1.1。
    * 请求方法: GET, POST, HEAD, PUT, DELETE, OPTIONS, TRACE, CONNECT。
    * URL字段
    * HTTP协议版本
* 头： header
* 空行
* 数据：request body


CORS: Cross-origin resource sharing
请求成功发送到服务器，但是被浏览器拦截了，两种方法实现跨域
* Access-Control-Allow-Origin
* JSONP: 浏览器允许src实现跨域
```
// Access-Control-Allow-Origin
response.writeHead(200, {
    'Access-Control-Allow-Origin': '*', 
    // 允许X-Test-Cors请求头   
    'Access-Control-Allow-Headers': 'X-Test-Cors',
    // 允许这些方法
    'Access-Control-Allow-Methods': 'POST, PUT, DELETE',
    // 预请求期限，期限内不需要预请求
    'Access-Control-Max-Age': '1000'
})

// JSONP
<script src="http://127.0.0.1:8777/"></script>
```

跨域其他限制：
* method只能是
    * get
    * post
    * head
* Content-Type只能是
    * text/plain
    * multipart/form-data
    * application/x-www-form-urlencoded
* 请求头限制



缓存Cache-Control
* 缓存期限
    * max-age浏览器缓存
    * s-maxage代理服务器缓存
* 缓存地方
    * public任何地方可以缓存
    * private只能在浏览器缓存
* 缓存机制
    * no-cache可以缓存但需要到服务器端验证
    * no-store不缓存

```
response.writeHead(200, {
    'Content-Type': 'text/javascript',
    'Cache-Control': 'max-age=200, public'
});
```

问题：资源更新了但是浏览器cache旧资源
解决：在资源名字后面加hash，资源改变hash就改变

## 缓存流程：





## 资源验证：
* Last-Modified：对比上次修改时间来验证资源是否需要更新
    * 配合If-Modified-Since或者If-Unmodified-Since
* Etag：数据签名，对比资源的签名判断是否使用缓存
    * 配合If-Match或者If-Non-Match

```
const etag = request.headers['if-none-match'];
if (etag === '777') {
    response.writeHead(304, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
    })
    response.end()
} else {
    response.writeHead(200, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
    })
    response.end('console.log("script loaded twice")')
}
```

## Cookie
* 通过Set-Cookie设置
* 设置后每次请求会自动带上
* 键值对，key value pair，可以多个
* 属性
    * max-age和expires设置过期时间
        * 没有设置过期时间的话，关闭浏览器cookie就会被清除
    * secure只在https时发送
    * 设置httpOnly可以禁止用document.cookie访问（防止csrf攻击）
```
response.writeHead(200, {
    'Content-Type': 'text/html’,
    // 只有test.com能访问abc=456，其二级域名（a.test.com）也能访问
    'Set-Cookie': ['id=123; max-age=200 httpOnly', 'abc=456; domain=test.com']
})
```

## HTTP长连接
* Connection: keep-alive
* 复用TCP连接
* Chrome最多同时6个TCP连接
```
response.writeHead(200, {
    'Content-Type': 'image/jpg',
    'Connection': 'keep-alive' // or close
})
```

## 数据协商

* 发送
  * Accept: text/html,  text/plain, application/xml...
  * Accept-Encoding: gzip, deflate, br
  * Accept-Language: zh-CN; q=0.9, en; q=0.8
  * User-Agent浏览器信息
* 返回
  * Content-Type
  * Content-Encoding
  * Content-Language

## Redirect
* 301: 永久跳转，下次直接访问新的地址，影响SEO
* 302: 暂时跳转

```
if (request.url === '/') {
    response.writeHead(302, { // or 301
        'Location': '/new'
    })
    response.end()
}
if (request.url === '/new') {
    response.writeHead(200, {
        'Content-Type': 'text/html',
    })
    response.end('<div>this is content</div>')
}
```


## CSP: Content-Security-Policy
* 用来限制资源获取
    * 限制方式
        * default-src全局限制
        * 其他限制：img-src， style-src，
* 报告资源获取越权

```
response.writeHead(200, {
    'Content-Type': 'text/html',
    // 不允许inline的src (包括img, script...) => 防止xss攻击
    'Content-Security-Policy': 'default-src http: https',
    // 不允许外部script， 除了https://cdn.bootcss.com/
    'Content-Security-Policy': 'script-src \'self\' https://cdn.bootcss.com/',
    // 不允许提交外部form action
    'Content-Security-Policy': 'form-action \'self\'',
    // 生成report
    'Content-Security-Policy': 'report-uri /report'
});

// 通过meta设置，但是不能设置report
<meta http-equiv="Content-Security-Policy" content="script-src 'self'; form-action 'self';">
```

CSP Report样例



## 代理缓存
好处：一个用户获取的资源缓存到代理服务器，其他用户如果获取相同的资源可以直接到缓存那里获得

```
response.writeHead(200, {
    // 浏览器用max-age， 代理服务器用s-maxage
    'Cache-Control': 'max-age=200, s-maxage=2000',
    // url相同，再加上X-Test-Cache这个头信息的值相同的时候才采用缓存，用处：根据User-Agent和Content-Language的不同，使用不同的缓存
    'Vary': 'X-Test-Cache'
})
```

## HTTPS:
1. 客户端发送Random 1 + Cipher Siutes (客户端发送一套加密套件)
2. 服务端发送Random 2 + Certificate (public key) + Cipher Siute (服务端选择一种加密套件)
3. 客户端用public key生成Random 3 (预主秘钥)，再用public key加密预主秘钥，发送到服务端
4. 服务端用private key解密预主秘钥
5. 两边用Cipher Siute加密三个Random，生成主秘钥session key来加密和解密数据




## HTTP 2:
1. 信道复用和分帧传输：只需要建立一个TCP链接，而且可以并行发送数据
2. Server Push
```
response.writeHead(200, {
    'Content-Type': 'text/html',
    'Connection': 'keep-alive’,
    // Server Push例子
    'Link': '</test.jpg>; as=image; rel=preload'
});
```
