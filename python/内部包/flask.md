# flask-compress 和JSON压缩

最近工作中遇到的一个小问题（小问题我都得写一写，就是这么一个鸡毛的女人！），前端要加载的JSON数据重达6M左右，打开Chrome的控制台，就能看到在一个请求中，状态已经变成200了，但仍然要10s+ 去加载这么重的数据，于是，在给老板演示基于本地代码的效果的时候，就有几十秒的静默时间。嗯……慢到我想给老板表演胸口碎大石（没有）。

![img](https:////upload-images.jianshu.io/upload_images/9402357-ddb44185e7557404.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/300/format/webp)

来啊～

  描述一下场景条件：

​    \1. 前端显示数据为几百K

​    \2. 前端实际加载了超过5M数据（有别用，不细说）

​    \3. 服务器在国外，浏览器在国内

  针对于上述问题，要采取JSON压缩的方式提高性能。网上给出两种策略： 1. 使用flask-compress 2.配置Nigix。 下面我们先来看一下flask-compress的使用。

  写一个小小的模拟例子。 鉴于是在本地，网络传输很快，所以首先把Chrome 控制台Network的offline设置为Fast 3G，来模拟一个比较限制的网速，如下图：

![img](https:////upload-images.jianshu.io/upload_images/9402357-7ce7376aeccc89e8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

限速

  然后下面是基本代码：

  前端： 点击按钮请求后端数据，只显示‘content_to_show’

   后端：读取一个大小为5M的文件内容，为‘content_not_show’，并和‘content_to_show’包成JSON返回（今天看了《毒液》，嘻嘻）

![img](https:////upload-images.jianshu.io/upload_images/9402357-5520a4c3e33c2fc8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

前端

![img](https:////upload-images.jianshu.io/upload_images/9402357-55ddd182df38be39.png?imageMogr2/auto-orient/strip|imageView2/2/w/938/format/webp)

后端

  点击按钮，看一下Chrome的记录，28.34s加载了5M数据。

![img](https:////upload-images.jianshu.io/upload_images/9402357-3b6773facb188409.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

无压缩访问结果

  看一下Waterfall，可见Content Download花了27.78s，几乎可以说是网速限制了一切。日常生活中，这不是网的错，是我的，谁让我没有办理光纤入户呢？

![img](https:////upload-images.jianshu.io/upload_images/9402357-8e8422b13d3532b4.png?imageMogr2/auto-orient/strip|imageView2/2/w/784/format/webp)

无压缩waterfall

  然后呢，我们打开flask-compress的开关，发送爱心 ，biubiu～

![img](https:////upload-images.jianshu.io/upload_images/9402357-0bd60998dce6f556.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/640/format/webp)

  这是README: https://github.com/jmcarp/flask-compress/blob/master/README.md

  步骤超级简单： 1. 打开冰箱，2. 把大象塞进去，3. 关上冰箱门

![img](https:////upload-images.jianshu.io/upload_images/9402357-3526183a31f0e647.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/346/format/webp)

嗯？

​    哦，走错片场了。

​    重来。步骤很简单：

​    \1. 安装flask-compress

​    \2. import flask-compress

​    \3. compress app

​    下面是使用flask-compress的代码，加两行，好很多～

![img](https:////upload-images.jianshu.io/upload_images/9402357-ef1ff9446fd409cc.png?imageMogr2/auto-orient/strip|imageView2/2/w/938/format/webp)

压缩后端代码

   然后我们再看一下效果，Size变成了5.4K，时间为毫秒级。很满意有没有！！

![img](https:////upload-images.jianshu.io/upload_images/9402357-74cb32344bc4f66b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

压缩结果1

 再看看Waterfall，可喜可贺～

![img](https:////upload-images.jianshu.io/upload_images/9402357-b57366a6bcd7fee8.png?imageMogr2/auto-orient/strip|imageView2/2/w/780/format/webp)

压缩的waterfall

  下面让我们对比一下response的参数，左边是压缩的，右边的是不压缩的。

![img](https:////upload-images.jianshu.io/upload_images/9402357-4c41c1d3c272f8b8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

response对比

  那多出来的参数怎么来的呢？ 这就要看flask-compress做了什么了。

![img](https:////upload-images.jianshu.io/upload_images/9402357-d96dd558825e8d53.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/400/format/webp)

Look what u made me do

  打开源文件，代码注释加空行总共119行，嗯，优秀的代码都这么短。

  flask-compress做的事情就是，向app中插入自己的after_request方法，在这个after_request方法中进行数据压缩，参数设置：

​    ***if (app.config['COMPRESS_REGISTER']*** ***and*** ***app.config['COMPRESS_MIMETYPES']):***

​         ***app.after_request(self.after_request)***

after_response中主要的代码，根据各种参数判断要不要做压缩，压缩完了设置encoding等参数，告诉浏览器如何压缩的，使浏览器知道如何解压。

![img](https:////upload-images.jianshu.io/upload_images/9402357-08e86e212f920448.png?imageMogr2/auto-orient/strip|imageView2/2/w/1060/format/webp)

flask-compress的after_request方法

![img](https:////upload-images.jianshu.io/upload_images/9402357-2ecf2b531eb8f7fa.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp)

这个人说完了吗？

  回答上面图片：没有。

  这里提出一个问题： 根据flask-compress 的代码，它是作用在application级别的，也就是说，经过这个application的请求都执行这个after_response方法，compress是需要消耗CPU的，如果不想所有的response都压缩怎么处理？

​    我认为有两种解决办法：

​    **1. 设置flask-compress提供的各种参数，来达到控制的效果**

​    **2. 这个是我想的，抽取压缩方法，结合flask的make_response方法，压缩特定的请求。**

  分开来说，第一点，这种产品肯定会考虑到我们的问题，如果每个请求都压缩，就有点笨了。可以设置两个参数控制是否压缩：COMPRESS_MIN_SIZE 和 COMPRESS_MIMETYPES。

  COMPRESS_MIN_SIZE 控制最小压缩阈值，小于这个值的就不压缩，默认值是500. COMPRESS_MIMETYPES 控制的是压缩类型，不在类型范围内不压缩，默认值是['text/html','text/css','text/xml','application/json','application/javascript']。设置方式如下，测试代码我就不写啦，大家自己玩玩～ 

![img](https:////upload-images.jianshu.io/upload_images/9402357-fbb8f65a2a283f15.png?imageMogr2/auto-orient/strip|imageView2/2/w/814/format/webp)

参数设置

  另外，还有第三个参数，COMPRESS_LEVEL，这个参数是控制压缩级别，就是gzip的压缩级别，1-9，1的压缩速度最快，压缩率最小，9 反之，默认值是6。在上面的例子中看不出来效果，我在项目中做测试的时候，有一种压缩使用了很多时间经历，特别设置了这个参数，效果立刻不一样～

  第二种，通过make_response方法获得response，复制flask-compress的代码内容，进行压缩。 经过测试，一样可以达到效果，作用域就变成了当前方法了。

![img](https:////upload-images.jianshu.io/upload_images/9402357-08c448692247d670.png?imageMogr2/auto-orient/strip|imageView2/2/w/1102/format/webp)

红色代码摘自flask-compress

  除了flask-compress，常用的压缩方法还有配置nigix。 这篇就不说啦，如果我能活过接下来的UAT，那我再继续写（基于前面的天津游记过了半年仍然没有后续，这个nigix的文章，嗯……have a nice life……）

![img](https:////upload-images.jianshu.io/upload_images/9402357-44b19cdb2eefe6b8.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/708/format/webp)

我会写的！

==============================

  写之前觉得很有意思，终于写完了，反而觉得像是在做测评，写完了就不好玩了。果然写东西就是要享受过程。 比如，一开始没有注意到Chrome控制网速的开关，本地load数据，500M也就用了几秒好嘛～看了之后差点崩溃，我去哪里找个远端服务器嘛～难道去找哆啦A梦・陈宝石？ 说来还是我脑子好使～🤪办法总比困难多～自己制造的困难自己放弃嘛（不是） ～

  这篇废话连篇的文章，就当作是又一个光棍节给自己的小小献礼。

  P.S. 最好的浏览器是IE，我说这个应该没人反对吧？



作者：MinamiSun
链接：https://www.jianshu.com/p/0e8cd091e2da
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



### 主动退出

```python
from flask import request

def shutdown_server():
    func = request.environ.get('werkzeug.server.shutdown')
    if func is None:
        raise RuntimeError('Not running with the Werkzeug Server')
    func()

@app.route('/shutdown', methods=['POST'])
def shutdown():
    shutdown_server()
    return 'Server shutting down...'
```

