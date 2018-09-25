---
layout: post
title:  "如何优雅的使用vue+Dcloud（Hbuild）开发混合app"
date:   2018-08-08
tags:  js 工具

color: 'rgb(154,133,255)'

---
#### webpack-dev-middleware-hard-disk插件
可以利用这个插件在运行dev的时候生成物理文件。

- ##### 首先现在项目中安装一下该插件，npm i --save-dev webpack-dev-middleware-hard-disk。然后只需要在项目build/webpack.dev.conf.js中添加下面这段代码：
```js
var compiler = webpack(devWebpackConfig)

var devMiddleware = require('webpack-dev-middleware-hard-disk')(compiler, {
  publicPath: devWebpackConfig.output.publicPath,
  quiet: true
})
```
这样就会在项目中生成dist文件夹，里面有一个app.js，而且如果修改代码保存会自动更新app.js。注意这里不会像build那样生成index.html，不过问题也不大，只需要把项目根目录下的index.html复制到dist文件夹下，然后把app.js引入index.html，代码如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>app</title>
  </head>
  <body>
    <div id="app">
        <script src="app.js"></script>
    </div>
  </body>
</html>
```
但是如果只配置这一个地方，只会在dist生成app.js，如果项目比较大，app.js的体积会比较大，如果我们还想像build那样生成vendor.js和manifest.js（app.js、vendor.js、manifest.js的区别这里不做过多解释），还需要继续在build/webpack.dev.conf.js文件里面的plugins添加下面代码：
```js
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks (module) {
        return (
        module.resource &&
        /\.js$/.test(module.resource) &&
        module.resource.indexOf(
            path.join(__dirname, '../node_modules')
        ) === 0
        )
    }
}),

new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest',
    minChunks: Infinity
}),

new webpack.optimize.CommonsChunkPlugin({
    name: 'app',
    async: 'vendor-async',
    children: true,
    minChunks: 3
}),
```
这样就会在dist文件夹下生成三个文件，app.js,manifest.js,vendor.js，由于还是运行在dev环境下，dev环境默认开了热更新，所以这三个文件也是热更新的。

然后在index.html中引入分别引入这三个文件，引入顺序为manifest.js > verdor.js > aoo.js 代码如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>app</title>
  </head>
  <body>
    <div id="app">
        <script src="manifest.js"></script>
        <script src="vendor.js"></script>
        <script src="app.js"></script>
    </div>
  </body>
</html>
```
到此dist文件夹就处理完了，在Hbuild中运行,然后就是把上面生成的dist文件夹再Hbuild中打开，首先打开Hbuild开发工具，然后依次 文件>打开目录 选择dist文件夹，然后我们需要吧dist文件夹转换成app项目：


这样就会在dist目录添加一个manifest.json文件，这样就成功转为了一个app项目：



用数据线把手机插入电脑（打开开发者调试模式），然后再Hbuild中依次 运行>手机运行> 选择你的手机 ：


然后稍等一会，就会在手机上运行看到效果。这时如果打开vue项目进行开发，比如修改了app.vue，只需要Ctrl+s保存一下app.vue，就会在手机上看到更改后的效果。


#### 处理返回键

##### 还有一个比较常见的问题，由于Vue做出来的页面是一个SPA，在Android机中如果按下了物理返回键，整个应用都会退出，解决方法是重写物理返回键，这样就能按路由一级一级地返回了。因为主界面是由原生实现的，所以Vue只能返回到对应模块的首页，比如从 /repair/apply -> /repair -> null ，想要回到原生主界面，需要后端向前端注入一段脚本，在模块首页的后退按钮被点击时，执行一段方法告知Android调用自身的逻辑，然后Android关闭当前页面并回到主界面，例如
```js
    //在main.js中加入该方法
window.AndroidMethod = function (msg) {
  if (window.android !== null && typeof(window.android) !== "undefined") {
    window.android.callAndroid(msg);
  }
}
```

##### 在头部组件header.vue中，可以使用如下方式：
```js
<!--回到主界面,isFirstPage通过props传入-->
<a v-if="isFirstPage" class="back" @click="backToHomePage"></a>

<!--普通返回-->
<a v-else @click="goback" class="back"></a>

methods:{
    goback() {
        window.history.length > 1 ? this.$router.go(-1) : this.$router.push('/')
    },
    backToHomePage() {
        AndroidMethod('backToHomePage')
    }
}
```
##### 这样可以将模块首页的返回和子路由的返回区分开来。

如果使用其他的打包工具，比如apiCloud或者HBuilder，它们都有各自的阻止物理返回按键的方法：
```js
//apiCloud
api.addEventListener({
  name: 'keyback'
}, function(ret, err){

  });
});

//HBuilder  
//https://blog.csdn.net/qq_25252769/article/details/76913083
document.addEventListener('plusready', function() {
    var webview = plus.webview.currentWebview();
    plus.key.addEventListener('backbutton', function() {
        webview.canBack(function(e) {
            if(e.canBack) {
                webview.back();
            } else {
                webview.close();
            }
        })
    });
});
```
同样的，把这些代码放在main.js中即可，打包后在真机里运行时会执行这些方法，普通环境是不存在这些变量的。

#### 接收后端返回的数据
有时候，我们希望在Vue初始化时就能设置一些从服务器获取的常量，比如userID等，之后在各个组件中就能很方便地访问。设置全局变量很简单，直接挂载在Vue.prototype后面即可：
```js
axios.get('http://localhost/index.php').then(res => {
  Vue.prototype.uid = res.data.uid
  Vue.prototype.appid = res.data.appid
  
  new Vue({
    el: '#app',
    router,
    store,
    render: h => h(App)
  })
})
```
在组件中使用this.uid、this.appid就能访问到从服务器获取的常量了。如果是普通的js文件(比如api，utils等等)，可以通过

```js
import Vue from 'vue'

Vue.prototype.uid
```
来访问。我们可能还希望这些数据在初始化时也能同时保存到Vuex中，先来看一下最初的Store/index.js文件：
```js
import Vue from 'vue'
import Vuex from 'vuex'
import * as actions from './actions'
import * as getters from './getters'
import state from './state'
import mutations from './mutations'

export default new Vuex.Store({
  actions,
  getters,
  state,
  mutations
})
```
但这样就没有往Vuex中存入数据的机会，这时就需要对Store文件夹中的index.js做一些小的封装，使其返回一个方法：
```js
function buidler(data) {
  return new Vuex.Store({
    actions,
    getters,
    state: data,
    mutations
  })
}

export default buidler
```
然后修改main.js中调用Vuex的方式，最初的代码如下：
```js
import store from './store'

new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```
修改后的代码如下：
```js
import store from './store'

axios.get('http://localhost/index.php').then(res => {
  Vue.prototype.uid = res.data.uid
  Vue.prototype.appid = res.data.appid

  new Vue({
    el: '#app',
    router,
    store:store(res.data),
    render: h => h(App)
  })
})
```
在组件的created方法中用MapGetters输出一下uid和appid，发现值可以被打印出来，~~说明这种实现方式是可以采用的。~~

##### 更新：
###### 在后续的测试中，发现一些机型，特别是华为机(实测iOS没有此问题)，对这种延后初始化Vue的方式兼容不好，表现在所有路由的切换动画全部失效，页面后退时会重新渲染页面(执行组件created方法中的内容)，设置keep-alive也没有效果。不过水平有限，实在弄不懂为什么会这样。为了兼容，就不能采用上面的方式了。最后使用了在请求头中携带cookie的方式，具体为webview加载vue页面时，在requestURL中注入cookie，在cookie中设置需要传递的值，下面是用PHP模拟的一个小例子，PHP加载HTML页，并注入cookie，在HTML加载时取到cookie。


```php
<?php
    header("Set-Cookie:testCookie=exist");

    include('index.html');

    echo 'cookie测试';
?>
```
HTML：
```html
<body>
    <script>
        console.log(document.cookie)
    </script>
</body>
```
这样就能在main.js里同步拿到userID，Vue也不用延迟初始化了，Android机的表现效果和iOS一致。拿到userID后，可以保存到配置文件config中，在每个组件中访问config.uid就能拿到。

#### better-scroll 
App中最常见的组件就是滚动数据列表，由此又很容易联想到better-scroll这个插件。better-scroll虽然好用，但如果使用不当还是会造成不小的麻烦，一些错误甚至无从排查。这里主要记录一下下拉刷新和上拉加载更多的实现。

---
最外层的div限制滚动内容的位置，srcoll是官网提供的已经封装好的组件，里面正常置入ul>li形式的列表就行了，ul和li都不需要特殊的样式。由于官网提供的例子中整合了许多文件，查阅起来不是很方便，于是将其剥离出来，写了一个只有上拉加载和下拉刷新的Demo，方便以后使用。使用scroll时要慎用v-show指令，比如我希望使用下面的代码来控制没有数据时容器的显示与隐藏，由于数据是异步加载，刚开始时容器不显示直到数据加载好为止。
```js
<div v-show="dataList.length > 0">
  <scroll></scroll>
</div>

data() {
  return {
    dataList: []
  }
}
```

但这样会造成scroll组件内部高度计算错误(offsetHeight被计算成0，这是由于容器处于display:none状态)，如果此时列表的数据没有达到滚动要求，上拉和下拉的提示文字会显示在列表下方，网速慢时也无法使用上拉下拉功能。解决方法是使用v-if指令，这样容器的min-height高度就能被正确计算了。

#### 图片上传

另外一个功能是图片上传，这个功能并非由前端完成，而是和上面一样，通过后台返回的一段函数体拿到上传图片的路径并展示出来

相册和拍摄都由后台调起，前端只需要进行简单的传值就行了： 
```js
AndroidMethod('photo')
AndroidMethod('video')
```
代码是和后端约定好的，所以不需要操心。真正需要关注的是从后台返回的图片上传路径，拿到这个路径后要在前台展示，并且保存时要带上一个或多个路径组成的字符串。

这个方法同样是和后台约定好的方法：
```js
window.getUpload = function (path) {
    //这里要将path保存起来拿到组件里使用
}
```

这里就需要使用全局变量将path保存起来，假定这个全局变量叫做uploadImgUrl，初始化时是一个空数组，只有当用户从相册里选择图片上传后才将拿到的路径赋给这个全局变量。

Vue组件中要监听这个全局变量的变化，就不能使用Vue.prototype.uploadImgUrl这种方式了，因为Vue要监听某个变量的变化，必须将这个变量放在data中，改进一下之前的代码：

```js
import store from './store'

axios.get('http://localhost/index.php').then(res => {
  Vue.prototype.uid = res.data.uid
  Vue.prototype.appid = res.data.appid

  let vm = new Vue({
    el: '#app',
    router,
    data() {
      return {
        uploadImgUrl: []
      }
    },
    store:store(res.data),
    render: h => h(App)
  })

  window.getUpload = function (path) {
    vm.uploadImgUrl = path
  }

})
```
上面将变量存放在根组件的data中，在其它组件内就可以通过以下形式访问到
```js
this.$root.$data.uploadImgUrl
```
虽然拿到了路径，但问题还没有结束，因为这个值是动态变化的，需要使用计算属性来监测它的变化，下面是核心代码：
```js
<template>
  <li>
    <div v-for="(item,index) in imgsList">
        <img :src="item" width="80" height="80" alt="">
        <i @click="deleteImg(index)"></i>
    </div>
  </li>
</template>

<script>
  export default {
    data() {
      return {
        loadedImgs:[] //保存已上传的图片
      }
    },
    methods: {
       deleteImg(index) {
        for (let i = 0; i < this.loadedImgs.length; i++) {
          if (index === i) {
            this.loadedImgs.splice(i, 1)
            break
          }
        }
      },
      computed: {
        imgsList() {
          this.loadedImgs = this.loadedImgs.concat(this.$root.$data.uploadImgUrl)
          this.$root.$data.uploadImgUrl = []　　//每次合并完重置一下
          return this.loadedImgs
        }
      },
      created() {
        this.$root.$data.uploadImgUrl = []　　//组件创建时先重置一下之前的值
      }
    }
  }
</script>
```
通过computed计算属性，无论是添加图片或者删除图片都能正确展示了。

#### 真机调试
在真机上调试非常不方便，很多调试信息看不到，不过vconsole这个插件解决了这个问题，安装方法非常简单，在依赖里(开发环境或正式环境均可)安装vconsole，然后在main.js中
```js
import Vconsole from 'vconsole'
new Vconsole()
```
打开页面就能看到右下角多出了一个vConsole的图标，项目中所有console.log的信息都会输出到这个vConsole面板里。