# 对于有多种可替代解决方案的业务逻辑，提供一种快速更换的思路

## 什么是"有多种可替代解决方案的业务逻辑"?
### 举几个例子说明：

客户端的http请求操作，可以实现的方案有Retrofix、OkHttp、Volley等；
客户端的数据库存储方案可以为Realm、greenDao、OrmLite等；
图片加载的方案可以是Fresco、Glide、Picasso、UIL等。
### 如何快速替换？
先来描述一下需求，比如说，目前正在用的http请求是Volley，现在发现使用OkHttp来封装一套会更好。又比方说，目前正在用的数据存储方案是OrmLite，现在使用greenDao或者Realm会更好，在类似这些情况下，如何做到不修改Activity/Fragment/Presenter代码的情况下，把Volley的http请求实现更换成Okhttp的实现，把OrmLite更换成greenDao或者Realm？

### 解决问题的关键词：设计模式中的——工厂方法模式。
## 本质：利用接口进行解耦。

说到这里，可能很多有经验的朋友已经会心一笑，是的，老实说这篇文章可能对老司机没有太大的意义，但是如果看到这里还是心存疑问，或者你不知道什么是工厂方法模式，也不知道如何使用接口解耦什么的，没关系，请继续往下看。

Talk is cheap, show me the code.
下面，我们就用Volley更换到OkHttp这个例子来说明一下如何做到不修改Activity/Fragment/Presenter的代码情况下，更快地更换业务逻辑实现的代码。

在开始之前先来思考一下上述所说“可替代解决方案”的含义，为何这些方案是可替代的？是因为它们具有相同的共性，它们所要解决的问题是相同的，比如说http请求框架，无论是Volley/OkHttp/Retrofix，它们所要实现的都是http请求中的get/post/put/delete这些方法，数据库存储框架中无论是Realm/greenDao/OrmLite，它们要实现的都是增删改查这些方法。

“博主你别再瞎逼逼，赶紧说重点…”
我：dalao我错了，下面说重点…

先来概括一下我们的实现思路：

把http请求框架的共性方法抽取到接口中，我们把这个接口称为“请求接口”；
创建一个用于返回请求结果的接口，我们把这个接口称为“回调接口”；
分别用Volley和OkHttp实现“请求接口”；
创建一个类来返回上述接口的对象，我们把这个类叫做“工厂”类；
在Activity/Fragment/Presenter中，使用“工厂”返回的这个接口对象调用get/post/put/delete方法，并在“回调接口”中得到请求结果。
本文完整代码https://github.com/V1sk/HttpRequestExample，可以先clone到本地再看文章，为了方便阅读，下文中的代码将省略非重点部分。

Step1：把http请求框架的共性方法抽取到接口中（也就是上述说的get/post/put/delete这些方法）
