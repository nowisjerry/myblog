[TOC]



# vue.js 插件开发详解

## 前言

随着 Vue.js 越来越火，Vue.js 的相关插件也在不断的被贡献出来，数不胜数。比如官方推荐的 vue-router、vuex 等，都是非常优秀的插件。但是我们更多的人还只停留在使用的阶段，比较少自己开发。所以接下来会通过一个简单的 vue-toast 插件，来了解掌握插件的开发和使用。

## 认识插件

想要开发插件，先要认识一个插件是什么样子的。

Vue.js 的插件应当有一个公开方法 install 。这个方法的第一个参数是 Vue 构造器 , 第二个参数是一个可选的选项对象:

```js
MyPlugin.install = function (Vue, options) {
  Vue.myGlobalMethod = function () {  // 1. 添加全局方法或属性，如: vue-custom-element
    // 逻辑...
  }
  Vue.directive('my-directive', {  // 2. 添加全局资源：指令/过滤器/过渡等，如 vue-touch
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })
  Vue.mixin({
    created: function () {  // 3. 通过全局 mixin方法添加一些组件选项，如: vuex
      // 逻辑...
    }
    ...
  })
  Vue.prototype.$myMethod = function (options) {  // 4. 添加实例方法，通过把它们添加到 Vue.prototype 上实现
    // 逻辑...
  }
}
```

接下来要讲到的 vue-toast 插件则是通过添加实例方法实现的。我们先来看个小例子。先新建个js文件来编写插件：toast.js

```js
// toast.js
var Toast = {};
Toast.install = function (Vue, options) {
    Vue.prototype.$msg = 'Hello World';
}
module.exports = Toast;
```

在 main.js 中，需要导入 toast.js 并且通过全局方法 Vue.use() 来使用插件：

```js
// main.js
import Vue from 'vue';
import Toast from './toast.js';
Vue.use(Toast);
```

然后，我们在组件中来获取该插件定义的 $msg 属性。

```js
// App.vue
export default {
    mounted(){
        console.log(this.$msg);         // Hello World
    }
}
```

可以看到，控制台成功的打印出了 Hello World 。既然 $msg 能获取到，那么我们就可以来实现我们的 vue-toast 插件了。

## 开发 vue-toast

需求：在组件中通过调用 this.\$toast('网络请求失败') 来弹出提示，默认在底部显示。可以通过调用 this.\$toast.top() 或 this.$toast.center() 等方法来实现在不同位置显示。

整理一下思路，弹出提示的时候，我可以在 body 中添加一个 div 用来显示提示信息，不同的位置我通过添加不同的类名来定位，那就可以开始写了。

```js
// toast.js
var Toast = {};
Toast.install = function (Vue, options) {
    Vue.prototype.$toast = (tips) => {
        let toastTpl = Vue.extend({     // 1、创建构造器，定义好提示信息的模板
            template: '<div class="vue-toast">' + tips + '</div>'
        });
        let tpl = new toastTpl().$mount().$el;  // 2、创建实例，挂载到文档以后的地方
        document.body.appendChild(tpl);     // 3、把创建的实例添加到body中
        setTimeout(function () {        // 4、延迟2.5秒后移除该提示
            document.body.removeChild(tpl);
        }, 2500)
    }
}
module.exports = Toast;
```

好像很简单，我们就实现了 this.$toast() ，接下来显示不同位置。

```js
// toast.js
['bottom', 'center', 'top'].forEach(type => {
    Vue.prototype.$toast[type] = (tips) => {
        return Vue.prototype.$toast(tips,type)
    }
})
```

这里把 type 传给 \$toast 在该方法里进行不同位置的处理，上面说了通过添加不同的类名(toast-bottom、toast-top、toast-center)来实现，那 $toast 方法需要小小修改一下。

```js
Vue.prototype.$toast = (tips,type) => {     // 添加 type 参数
    let toastTpl = Vue.extend({             // 模板添加位置类
        template: '<div class="vue-toast toast-'+ type +'">' + tips + '</div>'
    });
    ...
}
```

好像差不多了。但是如果我想默认在顶部显示，我每次都要调用 this.\$toast.top() 好像就有点多余了，我能不能 this.​$toast() 就直接在我想要的地方呢？还有我不想要 2.5s 后才消失呢？这时候注意到 Toast.install(Vue,options) 里的 options 参数，我们可以在 Vue.use() 通过 options 传进我们想要的参数。最后修改插件如下：

```js
var Toast = {};
Toast.install = function (Vue, options) {
    let opt = {
        defaultType:'bottom',   // 默认显示位置
        duration:'2500'         // 持续时间
    }
    for(let property in options){
        opt[property] = options[property];  // 使用 options 的配置
    }
    Vue.prototype.$toast = (tips,type) => {
        if(type){
            opt.defaultType = type;         // 如果有传type，位置则设为该type
        }
        if(document.getElementsByClassName('vue-toast').length){
            // 如果toast还在，则不再执行
            return;
        }
        let toastTpl = Vue.extend({
            template: '<div class="vue-toast toast-'+opt.defaultType+'">' + tips + '</div>'
        });
        let tpl = new toastTpl().$mount().$el;
        document.body.appendChild(tpl);
        setTimeout(function () {
            document.body.removeChild(tpl);
        }, opt.duration)
    }
    ['bottom', 'center', 'top'].forEach(type => {
        Vue.prototype.$toast[type] = (tips) => {
            return Vue.prototype.$toast(tips,type)
        }
    })
}
module.exports = Toast;
```

这样子一个简单的 vue 插件就实现了，并且可以通过 npm 打包发布，下次就可以使用 npm install 来安装了。

### 源码地址：[vue-toast](https://github.com/lin-xin/vue-toast)

### 更多文章：[blog](https://github.com/lin-xin/blog)





https://segmentfault.com/a/1190000008869576