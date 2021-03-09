---
layout: post
title: "http断点续传原理:http头Range、Content-Range"
date: 2021-02-27
tags: 随笔

color: "rgb(154,133,255)"
---
HTTP 请求头 `Range`
----------------

所谓断点续传，也就是要从文件已经下载的地方开始继续下载。在以前版本的 HTTP 协议是不支持断点的，HTTP/1.1 开始就支持了。一般断点下载时才用到 Range 和 Content-Range 实体头。

**Range** 

用于请求头中，指定第一个字节的位置和最后一个字节的位置，一般格式：

```js
Range:(unit=first byte pos)-\[last byte pos\] 
```

**Content-Range**

用于响应头，指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式： 

Content-Range: bytes (unit first byte pos) - \[last byte pos\]/\[entity legth\] 

**请求下载整个文件:** 

1.  GET  /test.rar  HTTP/1.1 
    
2.  Connection:  close 
    
3.  Host:  116.1.219.219 
    
4.  Range:  bytes=0-801 //一般请求下载整个文件是bytes=0- 或不用这个头
    

**一般正常回应**

1.  HTTP/1.1 200 OK 
    
2.  Content-Length:  801      
    
3.  Content-Type:  application/octet-stream 
    
4.  Content-Range:  bytes  0-800/801 //801:文件总大小
    
node静态服务器断点续传实现
---
> 当用户从静态文件服务器上获取诸如歌曲这样的流媒体文件时，如果网络连接断开，重连后未做处理，就需要重新下载这个文件。所以为了避免这种情况，我们的服务器需要一种断点续传的功能。而http 1.1中正好规定了一种`Range`机制，我们可以通过这种机制来进行分片传输。

### Range

当一个服务器支持`Range`时，客户端可以将需要发送的内容分成很多份发送给服务端，服务端可以每次接收部分内容。有了这样的能力，遇到断网的情况，我们可以在客户端记录下已经传送的文件范围，网络恢复后再将剩余部分发送给服务端，这样就实现了断点续传。

`Range`的在http 1.1中的具体定义可以参考文档：

*   rfc2616

具体流程如下：

*   浏览器请求内容。
    
*   服务器告诉浏览器，该内容可以使用 Accept-Ranges 消息头进行分部分请求。
    
        response.setHeader('Accept-Ranges', 'bytes');
    
*   浏览器重新发送请求，用 Range 消息头告诉服务器需要的内容范围。
    
    发送的Range格式：**字节数（bytes）= (开始)-(结束)**
    
    这是浏览器告知服务器所需分部分内容范围的消息头. 注意开始和结束位置是都包括在内的，而且是从0开始的. 这个消息头也可以不发送两个位置，其含义如下:
    
    *   如果结束位置被去掉了，服务器会返回从声明的开始位置到整个内容的结束位置内容的最后一个可用字节。
    *   如果开始位置被去掉了，结束位置参数可以被描述成从最后一个可用的字节算起可以被服务器返回的字节数。

而服务器会分如下两种情况响应浏览器的请求:

*   如果范围是合理的，服务器会返回所请求的部分内容，并带上 206 Partial Content 状态码. 当前内容的范围会在 Content-Range 消息头中申明。 发送的Content-Range格式：**字节数（bytes）=(开始)-(结束)/(总数)**
*   如果范围是不可用的(例如，比内容的总字节数大), 服务器会返回 416 请求范围不合理 Requested Range Not Satisfiable 状态码. 可用的范围也会在 Content-Range 消息头中声明。

### nodejs简单实现

```js
    getStream(req, res, filepath, statObj) {    
      let start = 0;    
      let end = statObj.size - 1;    
      let range = req.headers['range'];    
      if (range) {        
        res.setHeader('Accept-Range', 'bytes');        
        res.statusCode = 206;        
        let result = range.match(/bytes=(\d*)-(\d*)/);        
        if (result) {            
          start = isNaN(result[1]) ? start : parseInt(result[1]);            
          end = isNaN(result[2]) ? end : parseInt(result[2]) - 1;        
          }    
      }    
      return fs.createReadStream(filepath, {       
         start, 
         end    
         });}
```

### 待续

### 自我小结

在搭建node静态服务器过程中学习利用Http1.1的`Range`来实现断点续传功能

###### 参考资料

*   [Node.js中实现HTTP 206内容分片](https://link.juejin.im?target=http%3A%2F%2Fdeveloper.51cto.com%2Fart%2F201409%2F451095.htm)