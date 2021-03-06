# 10daysWeb
**A just-for-learning web framework that can be developed in 10 days.**

![PyPI](https://img.shields.io/pypi/pyversions/10daysweb.svg) ![PyPI](https://img.shields.io/pypi/status/10daysweb.svg) ![PyPI](https://img.shields.io/pypi/v/10daysweb.svg)

# 啰嗦
出于某些原因，我需要一个自己开发的轮子，大约只有十天时间。

于是我打算开发一个python web框架，这是我一直想做却又未完成的事。

我打算每天迭代，一遍写一遍查阅资料，记录新的想法和发现。

这样如果有谁与我处境相似，这个项目也许能够有所帮助。

最好能用成品再搭个博客什么的。

即使没有成功，也不会一无所获。

我们开始吧。

## Day 1
**万事开头难，相信我不是唯一一个在项目开始时感到无从下手的人。**

首先我下载了热门框架Flask的0.1版本的源码，三百余行的代码已经包含了一个web框架所必要的全部功能，还附带了一个使用示例。[如何下载最早的commit代码](#如何下载最早的commit代码)

对于我要实现的第一个最简单版本来说，flask仍然过于复杂了，我只提炼出`route`这个关键部件在第一版中实现。

`Route`用来管理一个web应用具体响应哪些路径和方法。通过装饰器，框架在启动时注册所有的用户函数，并在符合条件时自动调用。

    @testApp.route('/', methods=['GET'])
    def hello():
        return 'hello world'

而`Rule`则具体表示某个需要被响应的路径，它主要由`url`, `methods`和`endpoint`组成。

`methods`包含一系列HTTP Method，表示要处理的请求类型。而`endpoint`则是实际产生返回内容的`Callable`对象，可以是函数或者类。

关于http包含哪些method，以及后续我们需要参考的报文格式和状态码，参见[RFC 2616](#https://tools.ietf.org/html/rfc2616)。

现在我们还缺少一段代码，用于监听和收发http报文，python3.4以后加入的asyncio提供了这个功能，而[官方文档](#http://asyncio.readthedocs.io)恰好给了我们一个极简的示例。

`asyncio.start_server`需要三个基本参数，收到请求时的自动调用的`client_connected_cb`，以及需要监听的地址和端口。

`client_connected_cb`则需要支持两个参数，`reader`和`writer`，份别用于读取请求报文和回写响应报文。

我在`client_connected_cb`中添加了简易的获取请求的路径的代码，用于和注册好的应用函数匹配。

同样我也已经定义了包含所有Http method的宏，不过还没有与请求进行匹配。

这样我们就得到了一个可以运行的''Web框架''，目前只能算是prototype，不过已经足够让我们印出那句世纪名言了。

    Hello World!

## Day 2
**我们有了一个原型，但很多方面亟待完善**

我使用了一个开源第三方库来解析http报文，并实现了`Request`和`Response`来抽象请求。

我从rfc文档中摘取了http的状态码，和methods一起放在`utils.py`中。

尝试定义了一个异常，初步的设向是它可以让框架的使用者随时使用异常直接返回http的错误状态，`content`则是为了支持自定义的错误页面，但这部分仍不确定，也许我会使用`@error_handler`的形式来提供自定义异常时的行为。

添加了log，但在我的终端中还没有输出，待解决。

我使用了标准库`asyncio`，因为我希望这个框架是支持异步的，调整后的`handle`方法提现了处理一个请求的基本思路，但它看起来仍然很糟糕，对于异步我还未完全理清思路。

## Day 3

在代码方面，今天的改动并不大。

梳理了`handle`方法的逻辑, 我强制规定用户函数必须是协程，但日后也必须提供数据库，文件读写相关的异步封装API，否则框架仍然不是`真*异步`。

调整了流读取报文的处理策略，交由第三方解析库来判断报文是否结束。这方面并不用太过纠结，因为真正部署时讲会有nginx/apache之流替我们打理。

之后的主要工作：

 - 完成`Debug模式`，实现自动重加载用户函数
 - 添加静态文件路由和模式匹配路由支持
 - 引入模板引擎及其异步调用封装

## Day 4

添加了动态url匹配支援，现在可以在以如下形式匹配路径:

    @app.route('/<name>', methods=['GET'])
    async def show_name(request, name):
        return Response(content=f'hello {name}')

思考以后感觉静态文件路由完全可以由用户自行添加动态匹配来支持，即使不行还有web服务器来做，于是决定先放下这部分。

添加了`errorhandler`装饰器，现在可以通过它自定义异常时的行为和返回报文

调整了异常捕获机制，现在在找不到对应的用户方法时，能够正确的抛出404异常，而在用户方法中非预期中异常，则统一作为500状态处理

## Day 5 & 6

加入了`run_before`装饰器，用于在运行启动服务器前的初始化代码，默认传入事件循环loop参数

把这个~~丢人~~框架上传到了pip，现在可以通过`pip install 10daysweb`安装使用

尝试写一个todolist应用作为演示，康了半天前端觉得有些仓促，决定接入~~Telegram Bot~~微信小程序

加入了unitest，初步编写了一个url匹配的测试样例

## Day 7

新增信号装饰器，初步想法是用于服务器启动前和结束后初始化和关闭数据库连接池

    @app.signal(type='run_before_start')
    def foo(loop):
        '''init database connection pool'''

增加了对应的未知信号类型异常，微信小程序api编写中。


## 如何下载最早的commit代码

作为一个知名的开源项目，Flask在github已经积累了数千此提交。

最可恨的是，github在Commit列表页面竟然没有提供一个按页跳转的功能。

下面一个不是很优雅，但确实更快的方法

首先在本地`git clone`下目标项目

使用`--reverse`参数倒置结果，拿到提交历史上最早的commit id

    git log --reverse

在github上随意打开一个commit，替换掉url中的id即可。

哦，你还需要点一下`Browse files`