---
title:  "让Tornado框架解析json形式的HTTTP请求"
date:   2019-04-05 21:43:43 +0800
categories: Tornado 框架
tags: python framework tornado http body parse
---
### 背景：

如今写Web应用避不开Restful的风头。

在Rest风格的API中，`POST`、`PUT`、`PATCH`都是常用方法，而且都需要在HTTP body中携带业务数据。而[Json](http://www.json.org/)由于数据结构合理且对数据类型表达清晰等优点，已经成为Web接口通讯中的首选数据表达格式。

如果你的接口选择用*Json*形式 (`Content-Type`为`application/json`)跟客户端进行通讯，很遗憾，Tornado框架并不支持其自动解析。是的，不支持 [手动无奈]

Tornado现在只支持`Content-Type`为`application/x-www-form-urlencoded`和`multipart/form-data`形式的body解析。

那么如何对body为json格式的请求自动进行解析呢？

### 实现：

我们只需要在请求到来之后，body取值之前，判断一下此请求是否为*Json*形式（选择在`RequestHanlder.prepare`中进行处理也是因为其会在`get`、`post`这些方法之前运行）。如果是，将收到的HTTP body进行一下解码以备后用就可以了：

```python
from tornado.web import RequestHandler
from tornado.escape import json_decode

class TestHandler(RequestHandler):
    def prepare(self):
        content_type = self.requests.headers.get('Content-Type')
        if (content_type.lower() in
                ('application/json', 'application/json;charset=utf-8')):
            try:
                self.args = json_decode(request_obj.request.body)
            except ValueError:
                self.args = None
                self.set_status(422, 'Unprocessable Entity')
                self.finish()
        # 然后使用self.args替代常用的self.get_arguments来获取解析后的结果.

    def post(self):
        body_obj = self.args
```

如果想要强制接口使用者必须使用*Json*形式的*Content-Type*，可以：

```python
def prepare(self):
    content_type = self.requests.headers.get('Content-Type')
    if (content_type.lower() in
            ('application/json', 'application/json;charset=utf-8')):
        try:
            self.args = json_decode(request_obj.request.body)
        except ValueError:
            self.args = None
            self.set_status(422, 'Unprocessable Entity')
            self.finish()
    else:
        self.set_status(406, 'Not Acceptable')
        self.set_header('Accept', 'application/json,application/json;charset=utf-8')
        self.finish()
    # 然后使用self.args替代常用的self.get_arguments来获取解析后的结果.
```

再进一步。如果嫌每个Handler都写prepare方法麻烦，那么就继承个子类出来，在项目中使用。  

```python
class MyRequestHandler(RequestHandler):
    def prepare(self):
        content_type = self.requests.headers.get('Content-Type')
        if (content_type.lower() in
                ('application/json', 'application/json;charset=utf-8')):
            try:
                self.args = json_decode(request_obj.request.body)
            except ValueError:
                self.args = None
                self.set_status(422, 'Unprocessable Entity')
                self.finish()
        else:
            self.set_status(406, 'Not Acceptable')
            self.set_header('Accept', 'application/json,application/json;charset=utf-8')
            self.finish()
            
class TestHandler(MyRequestHandler):
    def post(self):
        body_obj = self.args
```

### 总结：

That's all！

### 引申：

关于示例代码中涉及到的HTTP状态码，可以参见 [Restful HTTP Status Codes](https://www.restapitutorial.com/httpstatuscodes.html) 
