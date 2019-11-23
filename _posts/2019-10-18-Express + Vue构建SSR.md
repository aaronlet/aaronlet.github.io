---
layout:     post
title:      Express + Vue构建SSR
subtitle:   Express and Vue builds SSR
date:       2019-09-12
author:     Aaron
header-img: img/blog/post-bg-ssrAndVue.jpg
catalog: true
tags:
    - TypeScript
    - Vue
---

最近简单的研究了一下`SSR`，对`SSR`已经有了一个简单的认知，主要应用于单页面应用，`Nuxt`是`SSR`很不错的框架。也有过调研，简单的用了一下，感觉还是很不错。但是还是想知道若不依赖于框架又应该如果处理`SSR`,研究一下做个笔记。

> ### 什么是SSR

把`Vue`组件渲染为服务器端的`HTML`字符串，将他们直接发送到浏览器，最后将静态标记`混合`为客户端上完全交互的应用程序。

> ### 为什么要使用SSR

1. 更好的SEO，搜索引擎爬虫爬取工具可以直接查看完全渲染的页面
2. 更宽的内容达到时间（time-to-content），当权请求页面的时候，服务端渲染完数据之后，把渲染好的页面直接发送给浏览器，并进行渲染。浏览器只需要解析`html`不需要去解析`js`。

> ### SSR弊端

1. 开发条件受限，`Vue`组件的某些生命周期钩子函数不能使用
2. 开发环境基于`Node.js`
3. 会造成服务端更多的负载。在`Node.js`中渲染完整的应用程序，显然会比仅仅提供静态文件`server`更加占用`CPU`资源，因此如果你在预料在高流量下使用，请准备响应的服务负载，并明智的采用缓存策略。

> ### 准备工作

在正式开始之前，在`vue`官网找到了一张这个图片，图中详细的讲述了`vue`中对`ssr`的实现思路。如下图简单的说一下。

下图中很重要的一点就是`webpack`，在项目过程中会用到`webpack`的配置，从最左边开始就是我们所写入的源码文件，所有的文件都有一个公共的入口文件`app.js`，然后就进入了`server-entry`(服务端入口)和`client-entry`(客户端入口)，两个入口文件都要经过`webpack`，当访问`node`端的时候，使用的是服务端渲染，在服务端渲染的时候，会生成一个`server-Bender`，最后通过`server-Bundle`可以渲染出`HTML`页面，若在客户端访问的时候则是使用客户端渲染，通过`client-Bundle`在以后渲染出`HTML`页面。so~通过这个图可以很清晰的看出来，接下来会用到两个文件，一个`server`入口，一个`client`入口，最后由`webpack`生成`server-Bundle`和`client-Bundle`，最终当去请求页面的时候，`node`中的`server-Bundle`会生成`HTML`界面通过`client-Bundle`混合到`html`页面中即可。

![](https://pic.xiaohuochai.site/blogssr1.png)

对于`vue`中使用`ssr`做了一些简单的了解之后，那么就开始我们要做的第一步吧，首先要创建一个项目，创建一个文件夹，名字不重要，但是最好不要使用中文。

```
mkdir dome
cd dome
npm init
```

`npm init`命令用来初始化`package.json`文件：

```JavaScript
{
  "name": "dome",   //  项目名称
  "version": "1.0.0",   //  版本号
  "description": "",    //  描述
  "main": "index.js",   //  入口文件
  "scripts": {          //  命令行执行命令 如：npm run test
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Aaron",     //  作者
  "license": "ISC"      //  许可证
}
```

初始化完成之后接下来需要安装，项目所需要依赖的包，所有依赖项如下：

```
npm install express --save-dev
npm install vue --save-dev
npm install vue-server-renderer --save-dev
npm install vue-router --save-dev
```

如上所有依赖项一一安装即可，安装完成之后就可以进行下一步了。前面说过`SSR`是服务端预渲染，所以当然要创建一个`Node`服务来支撑。在`dome`文件夹下面创建一个`index.js`文件，并使用`express`创建一个服务。

代码如下：

```JavaScript
const express = require("express");
const app = express();

app.get('*',(request,respones) => {
    respones.end("ok");
})

app.listen(3000,() => {
    console.log("服务已启动")
});
```

完成上述代码之后，为了方便我们需要在`package.json`添加一个命令，方便后续开发启动项目。

```JavaScript
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  }
}
```

创建好之后，在命令行直接输入`npm start`即可，当控制台显示`服务已启动`则表示该服务已经启动成功了。接下来需要打开浏览器看一下渲染的结果。在浏览器地址栏输入`locahost:3000`则可以看到`ok`两个字。

> ### SSR渲染手动搭建

前面的准备工作已经做好了，千万不要完了我们的主要目的不是为了渲染文字，主要的目标是为了渲染`*.vue`文件或`html`所以。接下来就是做我们想要做的事情了。接下来就是要修改`index.js`文件，将之前安装的`vue`和`vue-server-renderer`引入进来。

由于返回的不再是文字，而是`html`模板，所以我们要对响应头进行更改，告诉浏览器我们渲染的是什么，否则浏览器是不知道该如何渲染服务器返回的数据。

在`index.js`中引入了`vue-server-renderer`之后，在使用的时候，我们需要执行一下`vue-server-renderer`其中的`createRenderer`方法，这个方法的作用就是会将`vue`的实例转换成`html`的形式。

既然有了`vue-server-renderer`的方法，接下来就需要引入主角了`vue`,引入之后然后接着在下面创建一个`vue`实例，在`web`端使用`vue`的时候需要传一些参数给`Vue`然而在服务端也是如此也可以传递一些参数给`Vue`实例，这个实例也就是后续添加的那些`*.vue`文件。为了防止用户访问的时候页面数据不会互相干扰，暂时需要把实例放到`get`请求中，每次有访问的时候就会创建一个新的实例，渲染新的模板。

`creteRender`方法能够把`vue`的实例转成`html`字符串传递到浏览器。那么接下来由应该怎么做？在`vueServerRender`方法下面有一个`renderToString`方法，这个方法就可以帮助我们完成这步操作。这个方法接受的第一个参数是`vue`的实例，第二个参数是一个回调函数，如果不想使用回调函数的话，这个方法也返回了一个`Promise`对象，当方法执行成功之后，会在`then`函数里面返回`html`结构。

index.js改动如下：

```JavaScript
const express = require("express");
const Vue = require("vue");
const vueServerRender = require("vue-server-renderer").createRenderer();

const app = express();

app.get('*',(request,respones) => {
    const vueApp = new Vue({
        data:{
            message:"Hello,Vue SSR!"
        },
        template:`<h1>{{message}}</h1>` 
    });
    respones.status(200);
    respones.setHeader("Content-Type","text/html;charset-utf-8;");
    vueServerRender.renderToString(vueApp).then((html) => {
        respones.end(html);
    }).catch(error => console.log(error));
})

app.listen(3000,() => {
    console.log("服务已启动")
});
```

上述操作完成之后，一定要记得保存，然后重启服务器，继续访问一下`locahost:3000`，就会看到在服务端写入的`HTML`结构了。这样做好像给我们添加了大量的工作，到底与在`web`端直接使用有什么区别么？

接下来见证奇迹的时刻到了。在网页中右键`查看源代码`就会发现与之前的在`web`端使用的时候完全不同，可以看到渲染的模板了。如果细心的就会发现一件很有意思的事情，在`h1`标签上会有一个`data-server-rendered=true`这样的属性，这个可以告诉我们这个页面是通过服务端渲染来做的。大家可以去其他各大网站看看哦。没准会有其他的收获。

上面的案例中，虽然已经实现了服务端预渲染，但是会有一个很大的缺陷，就是我们所渲染的这个网页并不完整，没有文档声明，`head`等等等，当然可能会有一个其他的想法，就是使用`es6`的模板字符串做拼接就好了啊。确实，这样也是行的通的，但是这个仍是饮鸩止渴不能彻底的解决问题，如果做过传统`MVC`开发的话，就应该知道，`MVC`开发模式全是基于模板的，现在这种与`MVC`有些相似的地方，同理也是可以使用模板的。在`dome`文件夹下创建`index.html`，并创建好`HTML`模板。

模板现在有了该如何使用？在`createRenderer`函数可以接收一个对象作为配置参数。配置参数中有一项为`template`,这项配置的就是我们即将使用的`Html`模板。这个接收的不是一个单纯的路径，我们需要使用`fs`模块将`html`模板读取出来。

其配置如下：

```JavaScript
let path = require("path");
const vueServerRender = require("vue-server-renderer").createRenderer({
    template:require("fs").readFileSync(path.join(__dirname,"./index.html"),"utf-8")
});
```

现在模板已经有了，在`web`端进行开发的时候，需要挂在一个`el`的挂载点，这样`Vue`才知道把这些`template`渲染在哪，服务端渲染也是如此，同样也需要告诉`Vue`将`template`渲染到什么地方。接下来要做的事情就是在`index.html`中做手脚。来通知`createRenderer`把`template`添加到什么地方。

更改`index.html`文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
</head>
<body>
    <!--vue-ssr-outlet-->
</body>
</html>
```

可以发现，在`html`的`body`里面添加了一段注释，当将`vueServerRender`编译好的`html`传到模板当中之后这个地方将被替换成服务端预编译的模板内容，这样也算是完成一个简单的服务端预渲染了。虽然写入的只是简单的`html`渲染，没有数据交互也没有页面交互，也算是一个不小的进展了。

使用`SSR`搭建项目我们继续延续上个项目继续向下开发，大家平时在使用`vue-cli`搭建项目的时候，都是在`src`文件夹下面进行开发的，为了和`vue`项目结构保持一致，同样需要创建一个`src`文件夹，并在`src`文件夹创建`conponents,router,utils,view`,暂定项目结构就这样，随着代码的编写会逐渐向项目里面添加内容。

```
└─src
|   ├─components
|   ├─router
|   ├─utils
|   ├─view
|   └─app.js
└─index.js
```

初始的目录结构已经搭建好了之后，接下来需要继续向下进行，首先要做的就是要在`router`目录中添加一个`index.js`文件，用来创建路由信息（在使用路由的时候一定要确保路由已经安装）。路由在项目中所起到的作用应该是重要的，路由会通过路径把页面和组件之间建立联系，并且一一的对应起来，完成路由的渲染。

接下来在`router`下面的`index.js`文件中写入如下配置：

```JavaScript
const vueRouter = require("vue-router");
const Vue = require("vue");

Vue.use(vueRouter);

module.exports = () => {
    return new vueRouter({
        mode:"history",
        routes:[
            {
                path:"/",
                component:{
                    template:`<h1>这里是首页</h1>`
                },
                name:"home"
            },
            {
                path:"/about",
                component:{
                    template:`<h1>这里是关于我</h1>`
                },
                name:"about"
            }
        ]
    })
}
```

上面的代码中，仔细观察的话，和平时在`vue-cli`中所导出的方式是不一样的，这里采用了工厂方法，这里为什么要这样？记得在雏形里面说过，为了保证用户每次访问都要生成一个新的路由，防止用户与用户之间相互影响，也就是说Vue实例是新的，我们的`vue-router`的实例也应该保证它是一个全新的。

现在`Vue`实例和服务端混在一起，这样对于项目的维护是很不好的，所以也需要把`Vue`从服务端单独抽离出来，放到`app.js`中去。这里采用和`router`同样的方式使用工厂方式，以保证每次被访问都是一个全新的`vue`实例。在`app.js`导入刚刚写好的路由，在每次触发工厂的时候，创建一个新的路由实例，并绑定到`vue`实例里面，这样用户在访问路径的时候无论是`vue`实例还是`router`都是全新的了。

app.js：

```JavaScript
const Vue = require("vue");
const createRouter = require("../router")

module.exports = (context) => {
    const router = createRouter();
    return new Vue({
        router,
        data:{
            message:"Hello,Vue SSR!"
        },
        template:`
            <div>
                <div>
                    <h1>{{message}}</h1>
                    <ul>
                        <li>
                            <router-link to="/">首页</router-link>
                        </li>
                        <li>
                            <router-link to="/about">关于我</router-link>
                        </li>
                    </ul>
                </div>
                <router-view></router-view>
            </div>
        ` 
    });
}
```

做完这些东西貌似好像就能用了一样，但是还是不行，仔细想想好像忘了一些什么操作，刚刚把`vue`实例从`index.js`中抽离出来了，但是却没有在任何地方使用它，哈哈，好像是一件很尴尬的事情。

修改`index.js`文件：

```JavaScript
const express = require("express");
const vueApp = require("./src/app.js");
let path = require("path");
const vueServerRender = require("vue-server-renderer").createRenderer({
  template:require("fs").readFileSync(path.join(__dirname,"./index.html"),"utf-8")
});

const app = express();

app.get('*',(request,respones) => {
    
    //  这里可以传递给vue实例一些参数
    let vm = vueApp({})
    
    respones.status(200);
    respones.setHeader("Content-Type","text/html;charset-utf-8;");
    vueServerRender.renderToString(vm).then((html) => {
        respones.end(html);
    }).catch(error => console.log(error));
})

app.listen(3000,() => {
    console.log("服务已启动")
});
```

准备工作都已经做好啦，完事具备只欠东风啦。现在运行一下`npm start`可以去页面上看一下效果啦。看到页面中已经渲染出来了，但是好像是少了什么？虽然导航内容已经都显示出来了，但是路由对应的组件好像没得渲染噻。具体是什么原因导致的呢，`vue-router`是由前端控制渲染的，当访问路由的时候其实，在做首屏渲染的时候并没有授权给服务端让其去做渲染路由的工作。(⊙﹏⊙)，是的我就是这么懒...

这个问题解决方案也提供了相对应的操作，不然就知道该怎么写下去了。既然在做渲染的时候分为服务端渲染和客户端渲染两种，那么我们就需要两个入口文件，分别对应的服务端渲染的入口文件，另个是客户端渲染的入口文件。

在`src`文件夹下面添加两个`.js`文件(当然也可以放到其他地方，这里只是为了方便),`entry-client.js`这个文件用户客户端的入口文件，`entry-server.js`那么这个文件则就作为服务端的入口文件。既然入口文件已经确定了，接下来就是要解决刚才的问题了,首先解决的是服务端渲染，在服务端这里需要把用户所访问的路径传递给`vue-router`，如果不传递给`vue-router`的话，`vue-router`会一脸懵逼的看着你，你什么都不给我，我怎么知道渲染什么？

在`entry-server`中需要做的事情就是需要把`app.js`导入进来，这里可以向上翻一下`app.js`中保存的是创建vue实例的方法。首先在里面写入一个函数，至于为什么就不多说了（同样也是为了保证每次访问都有一个新的实例），这个函数接收一个参数（`[object]`），由于这里考虑到可能会有异步操作(如懒加载)，在这个函数中使用了`Promise`，在`Promise`中首先要拿到连个东西，不用猜也是能想到的，很重要的`vue`实例和`router`实例，so~但是在`app`中好像只导出了`vue`实例，还要根据当前所需要的去更改`app.js`。

app.js:

```JavaScript
const Vue = require("vue");
const createRouter = require("../router")

module.exports = (context) => {
    const router = createRouter();
    const app = new Vue({
        router,
        data:{
            message:"Hello,Vue SSR!"
        },
        template:`
            <div>
                <div>
                    <h1>{{message}}</h1>
                    <ul>
                        <li>
                            <router-link to="/">首页</router-link>
                        </li>
                        <li>
                            <router-link to="/about">关于我</router-link>
                        </li>
                    </ul>
                </div>
                <router-view></router-view>
            </div>
        ` 
    });
    return {
        app,
        router
    }
}
```

通过上面的改造之后，就可以在`entry-server.js`中轻松的拿到`vue`和`router`的实例了，现在查看一下当前`entry-server.js`中有那些可用参数，`vue`,`router`,提及到的`URL`从哪里来？既然这个函数是给服务端使用的，那么当服务端去执行这个函数的时候，就可以通过参数形式传递进来，获取到我们想要的参数，我们假设这个参数叫做`url`，我们需要让路由去做的就是跳转到对应的路由中（这一步很重要），然后再把对`router`的实例挂载到`vue`实例中，然后再把`vue`实例返回出去，供`vueServerRender`消费。那么就需要导出这个函数，以供服务端使用。

由于我们不能预测到用户所访问的路由就是在`vue-router`中所配置的，所以需要在`onReady`的时候进行处理，我们可以通过`router`的`getMatchedComponents`这个方法，获取到我们所导入的组件，这些有个我们就可通过判断组件对匹配结果进行渲染。

entry-server.js

```JavaScript
const createApp = require("./app.js");

module.exports = (context) => {
    return new Promise((reslove,reject) => {
        let {url} = context;
        let {app,router} = createApp(context);
        router.push(url);
        //  router回调函数
        //  当所有异步请求完成之后就会触发
        router.onReady(() => {
            let matchedComponents = router.getMatchedComponents();
            if(!matchedComponents.length){
                return reject({
                    code:404,
                });
            }
            reslove(app);
        },reject)
    })
}
```

既然实例又发生了变化，需要对应发生变化的`index.js`同样也需要做出对应的改动。把刚才的引入`vue`实例的路径改为`entey-server.js`，由于这里返回的是一个`Promise`对象，这里使用`async/await`处理接收一下，并拿到`vue`实例。不要忘了把`router`所需要的`url`参数传递进去。

index.js:

```JavaScript
const express = require("express");
const App = require("./src/entry-server.js");
let path = require("path");
const vueServerRender = require("vue-server-renderer").createRenderer({
  template:require("fs").readFileSync(path.join(__dirname,"./index.html"),"utf-8")
});

const app = express();

app.get('*',async (request,respones) => {
    respones.status(200);
    respones.setHeader("Content-Type","text/html;charset-utf-8;");

    let {url} = request;
    //  这里可以传递给vue实例一些参数
    let vm = await App({url});
    vueServerRender.renderToString(vm).then((html) => {
        respones.end(html);
    }).catch(error => console.log(error));
})

app.listen(3000,() => {
    console.log("服务已启动")
});
```

这下子就完成了，启动项目吧，当访问根路径的时候，就会看到刚才缺少的组件也已经渲染出来了，当然我们也可以切换路由，也是没有问题的。大功告成。。。好像并没有emmmmmmmmm，为什么，细心的话应该会发现，当我们切换路由的时候，地址栏旁边的刷新按钮一直在闪动，这也就是说，我们所做出来的并不是一个单页应用（手动笑哭），出现这样的问题也是难怪的，毕竟我们没有配置前端路由，我们把所有路由的控制权都交给了服务端，每次访问一个路由的时候，都会向服务端发送一个请求，返回路由对应的页面。想要解决这个问题，当处于前端的时候我们需要让服务端把路由的控制权交还给前端路由，让前端去控制路由的跳转。

之前在`src`文件夹下面添加了两个文件，只用到了服务端的文件，为了在客户端能够交还路由控制权，要对`web`端路由进行配置。由于在客户端在使用`vue`的时候需要挂载一个`document`，因为`vue`的实例已经创建完成了，所以，这里需要使用`$mount`这个钩子函数，来完成客户端的挂载。同样为了解决懒加载这种类似的问题so~同样需要使用`onReady`里进行路由的处理，只有当`vue-router`加载完成以后再去挂载。

在客户端是使用的时候很简单，只需要把路由挂载到`app`里面就可以了。

entry-client.js

```JavaScript
const createApp = require("./app.js");
let {app,router} = createApp({});

router.onReady(() => {
    app.$mount("#app")
});
```

整个项目的雏形也就这样了，由于服务端把路由控制权交还给客户端，需要复杂的`webpack`配置，so~不再赘述了，下面直接使用`vue-cli`继续（做的是使用需要用到上面的代码）。

> ### vue-cli项目搭建

在做准备工作的时候简单讲述了`vue`中使用`ssr`的运行思路，里面提及了一个很重要的`webpack`，因此这里需要借助`vue-cli`脚手架，直接更改原有的`webpack`就可以了，这样会方便很多。

这里建议大家返回顶部再次看一下`vue`服务端渲染的流程，在介绍中的`client-bundle`和`server-bundle`，，所以需要构建两个配置，分别是服务端配置和客户端的配置。

如想要实现服务端渲染需要对`vue-cli`中个`js`文件中的配置进行修改。以下只展示更改部分的代码，不展示全部。

文件分别是：

1. webpack.server.conf.js - 服务端webpack配置
2. dev-server.js - 获取服务端bundle
3. server.js - 创建后端服务
4. webpack.dev.conf.js - 客户端的bundle
5. webpack.base.conf - 修改入口文件

**客户端配置**

客户端生成一份客户端构建清单，记录客户端的资源，最终会将客户端构建清单中记录的文件，注入到执行的执行的模板中，这个清单与服务端类似，同样也会生成一份`json`文件，这个文件的名字是`vue-ssr-client-manifest.json`（项目启动以后可以通过地址/文件名访问到），当然必不可少的是，同样也需要引入一个叫做`vue-server-renderer/client-plugin`模块，作为`webpack`的插件供其使用。

首先要安装一下`vue-server-renderer`这个模块，这个是整个服务端渲染的核心，没有整个`ssr`是没有任何灵魂的。

```
npm install vue-server-renderer -S
```

安装完成之后，首先要找到`webpack.dev.conf.js`，首先要对其进行相关配置。

webpack.dev.conf.js

```JavaScript
//  添加引入  vue-server-render/client-plugin  模块
const vueSSRClientPlugin = require("vue-server-renderer/client-plugin");

const devWebpackConfig = merge(baseWebpackConfig,{
    plugins:[
        new vueSSRClientPlugin()
    ] 
});
```

添加了这个配置以后，重新启动项目通过地址就可以访问到`vue-ssr-client-manifest.json`（http://localhost:8080/vue-ssr-client-manifest.json ），页面中出现的内容就是所需要的`client-bundle`。

**服务端配置**

服务端会默认生成一个`vue-ssr-server-bundle.json`文件，在文件中会记录整个服务端整个输出，怎么才能生成这个文件呢？要在这个`json`文件，必须要引入`vue-server-renderer/server-plugin`,并将其作为`webpack`的插件。

在开始服务端配置之前，需要在`src`文件夹下面创建三个文件，`app.js`，`entry-client.js`，`entry-server.js`，创建完成之后需要对其写入相关代码。

src/router/index.js

```JavaScript
import vueRouter from "vue-router";
import Vue from "vue";
import HelloWorld from "@/components/HelloWorld";

Vue.use(vueRouter);
export default () => {
    return new vueRouter({
        mode:"history",
        routes:[
            {
                path:"/",
                component:HelloWorld,
                name:"HelloWorld"
            }
        ]
    })
}
```

app.js

```JavaScript
import Vue from "vue";
import createRouter from "./router";
import App from "./App.vue";

export default (context) => {
    const router = createRouter();
    const app = new Vue({
        router,
        components: { App },
        template: '<App/>'
    });
    return {
        app,
        router
    }
}
```

entry-server.js

```JavaScript
import createApp from "./app.js";

export default (context) => {
    return new Promise((reslove,reject) => {
        let {url} = context;
        let {app,router} = createApp(context);
        router.push(url);
        router.onReady(() => {
            let matchedComponents = router.getMatchedComponents();
            if(!matchedComponents.length){
                return reject({
                    code:404,
                });
            }
            reslove(app);
        },reject)
    })
}
```

entry-client.js

```JavaScript
import createApp from "./app.js";
let {app,router} = createApp();

router.onReady(() => {
    app.$mount("#app");
});
```

webpack.base.conf.js

```JavaScript
module.exports = {
    entry:{
        app:"./src/entry-client.js"
    },
    output:{
        publicPath:"http://localhost:8080/"
    }
};
```

webpack.server.conf.js(手动创建)

```JavaScript
const webpack = require("webpack");
const merge = require("webpack-merge");
const base = require("./webpack.base.conf");
//  手动安装
//  在服务端渲染中，所需要的文件都是使用require引入，不需要把node_modules文件打包
const webapckNodeExternals = require("webpack-node-externals");


const vueSSRServerPlugin = require("vue-server-renderer/server-plugin");

module.exports = merge(base,{
    //  告知webpack，需要在node端运行
    target:"node",
    entry:"./src/entry-server.js",
    devtool:"source-map",
    output:{
        filename:'server-buldle.js',
        libraryTarget: "commonjs2"
    },
    externals:[
        webapckNodeExternals()
    ],
    plugins:[
        new webpack.DefinePlugin({
            'process.env.NODE_ENV':'"devlopment"',
            'process.ent.VUE_ENV': '"server"'
        }),
        new vueSSRServerPlugin()
    ]
});
```

dev-server.js(手动创建)

```JavaScript
const serverConf = require("./webpack.server.conf");
const webpack = require("webpack");
const fs = require("fs");
const path = require("path");
//  读取内存中的.json文件
//  这个模块需要手动安装
const Mfs = require("memory-fs");
const axios = require("axios");

module.exports = (cb) => {
    const webpackComplier = webpack(serverConf);
    var mfs = new Mfs();
    
    webpackComplier.outputFileSystem = mfs;
    
    webpackComplier.watch({},async (error,stats) => {
        if(error) return console.log(error);
        stats = stats.toJson();
        stats.errors.forEach(error => console.log(error));
        stats.warnings.forEach(warning => console.log(warning));
        //  获取server bundle的json文件
        let serverBundlePath = path.join(serverConf.output.path,'vue-ssr-server-bundle.json');
        let serverBundle = JSON.parse(mfs.readFileSync(serverBundlePath,"utf-8"));
        //  获取client bundle的json文件
        let clientBundle = await axios.get("http://localhost:8080/vue-ssr-client-manifest.json");
        //  获取模板
        let template = fs.readFileSync(path.join(__dirname,"..","index.html"),"utf-8");
        cb && cb(serverBundle,clientBundle,template);
    })
};
```

根目录/server.js(手动创建)

```JavaScript
const devServer = require("./build/dev-server.js");
const express = require("express");
const app = express();
const vueRender = require("vue-server-renderer");

app.get('*',(request,respones) => {
    respones.status(200);
    respones.setHeader("Content-Type","text/html;charset-utf-8;");
    devServer((serverBundle,clientBundle,template) => {
        let render = vueRender.createBundleRenderer(serverBundle,{
            template,
            clientManifest:clientBundle.data,
            //  每次创建一个独立的上下文
            renInNewContext:false
        }); 
        render.renderToString({
            url:request.url
        }).then((html) => {
            respones.end(html);
        }).catch(error => console.log(error));
    });
})

app.listen(5000,() => {
    console.log("服务已启动")
});
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
</head>
<body>
    <div id="app">
        <!--vue-ssr-outlet-->
    </div>
    <!-- built files will be auto injected -->
</body>
</html>
```

以上就是所有要更改和添加的配置项，配置完所有地方就可以完成服务端渲染。此时需要在`package.json`中的`sctipt`中添加启动项：`http:node server.js`，就可以正常运行项目了。注意一定要去访问服务端设置的端口，同时要保证你的客户端也是在线的。

> 总结

这篇博客耗时3天才完成，可能读起来会很费时间，但是却有很大的帮助，希望大家能够好好阅读这篇文章，对大家有所帮助。

感谢大家花费很长时间来阅读这篇文章，若文章中有错误灰常感谢大家提出指正，我会尽快做出修改的。
