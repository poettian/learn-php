curl的使用
=======

#### 使用POST发送数据

在谈到curl的post前，先了解http协议中post相关的一些基础知识：

##### http协议之post

http协议规定请求分为三个部分：状态行、请求头、消息主体。

> <method> <request-URL> <version>
> <headers>
>
> <entity-body>

协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

但是发送的数据需要服务端能够解析才有意义，服务端一般根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。

以 post 发送数据常见的 Content-Type 有：

*  application/x-www-form-urlencoded
*  multipart/form-data
*  application/json
*  text/xml

其中，前两者都是浏览器原生支持的，而且现阶段标准中原生 <form> 表单也[只支持这两种方式](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4)（通过 <form> 元素的 `enctype` 属性指定，默认为 `application/x-www-form-urlencoded`。其实 `enctype` 还支持 `text/plain`，不过用得非常少）并且可以被php自动解析，从 $_POST 中能够取出格式化好的数据，而后两者只能通过 php://input 取到原始数据后再人为解析。



具体见：[四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)

##### curl之post

结合 `curl --help` 来看，curl 中与post相关的参数有：

```curl
 -b, --cookie STRING/FILE  Read cookies from STRING/FILE (H)
 -c, --cookie-jar FILE  Write cookies to FILE after operation (H)
 -d, --data DATA     HTTP POST data (H)
     --data-raw DATA  HTTP POST data, '@' allowed (H)
     --data-ascii DATA  HTTP POST ASCII data (H)
     --data-binary DATA  HTTP POST binary data (H)
     --data-urlencode DATA  HTTP POST data url encoded (H)
 -H, --header LINE   Pass custom header LINE to server (H)
 -o, --output FILE   Write to FILE instead of stdout
 -X, --request COMMAND  Specify request command to use
```

具体来分析：

`--cookie` 指明发送请求时需要携带的cookie，这也可以通过 

```curl
-H  'Cookie: _ga=GA1.2.1957966148.1543568730; GCID=65990ce-33df77d-           3ba9ad2-3f26d82; _gid=GA1.2.699799405.1548145119;'
```

来实现，两种方法都可以。

`--cookie-jar` 指明响应中的cookie应该写入哪个文件

`--header` 就是发送请求时的header信息，如：

```curl
-H 'Referer: http://www.poettian.com' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'
```

`--output` 指明响应体写入哪个文件而不是标准输出

`--request` 指明使用 GET|POST|PUT|DELETE 哪种方法来发送请求

**如果采用 post 发送请求**，默认是使用 `Content-Type: application/x-www-form-urlencoded`，除非通过 `--header` 明确指明 Content-Type：

`--data` 此参数会将字符串原封不动的放入请求体中；此外还可以使用 `--data '@/path/to/file'` 来将文件中的数据放入请求体，但是会剔除文件内容中的回车符和换行符

`--data-raw` 此参数和 `--data` 相似，只是当使用 @ 时，只是把它视为普通的一个字符

`--data-ascii` 

`--data-binary` 此参数指明post数据为纯二进制数据，如果是@file_name,则保留文件中的回车符和换行符，不做任何转换

`--data-urlencode` 此参数会将post数据中的 键值(键名不会) 和 & 进行url编码