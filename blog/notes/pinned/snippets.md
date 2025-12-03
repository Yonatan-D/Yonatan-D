# Snippets

## 重写全局对象里的方法函数

以微信小程序 wx 对象为例，重写 showLoading 和 hideLoading 函数，在调用时进入断点并输出文件位置，方便调试

```js
wx = wxProxy(wx);

function wxProxy(wx) {
  const newWx = { ...wx };

  // 代理函数，在调用时进入断点并输出文件位置
  const proxyFn = (fn) => {
    return (object) => {
      console.trace();
      debugger
      return fn(object);
    }
  }

  // 重写 showLoading 和 hideLoading 函数
  newWx.showLoading = proxyFn(wx.showLoading);
  newWx.hideLoading = proxyFn(wx.hideLoading);

  return newWx;
}
```

更多用法（解决加载过程中loading重复刷新、路由跳转拦截）：https://github.com/Yonatan-D/wechat-snippets

## 前端使用 Path.join: 用JavaScript实现node.js中的path.join方法

使用：

```js
join('../var/www', '../abc')
> "../var/abc"

join('../var/www', '\abc')
../var/www/abc
```

**原文代码: http://ourjs.com/detail/5b6f0dcd9ed90647f8af6197**

> Node.JS中的 path.join 非常方便，能直接按相对或绝对合并路径，使用： path.join([path1], [path2], [...])，有时侯前端也需要这种方法，如何实现呢？   
> 其实直接从 node.js 的 path.js 拿到源码加工一下就可以了： 
> 1. 将 const 等 es6 属性改为 var，以便前端浏览器兼容
> 2. 添加一个判断路戏分隔符的变量 sep，即左斜杠还是右斜杠，以第一个路戏分隔符为准
> 3. 将引用的变量和函数放到一个文件里就可以了：
>
> Path 的源码： https://github.com/nodejs/node/blob/master/lib/path.js

他们的博客还有很多有趣的实现: http://ourjs.com/userinfo/ourjs

PS：把页面相对路径处理成绝对路径的做法我也用过：[fixRelativeUrl](https://github.com/Yonatan-D/wechat-snippets/blob/4d6071bcc1601985711276e003f2908cc832862e/enableGrayVersion/index.js#L283)

```js
/**
 * 把页面相对路径处理成绝对路径
 * @param {string} base 
 * @param {string} relative 
 * @returns {string} 
 */
const fixRelativeUrl = (base, relative) => {
  if (relative.startsWith('/')) {
    return relative.replace('/', '');
  }

  let stack = base.split('/');
  let parts = relative.split('/');
  stack.pop(); // 去掉当前文件名

  for (let i = 0; i < parts.length; i++) {
    if (parts[i] == '.') continue;
    if (parts[i] == '..') stack.pop();
    else stack.push(parts[i]);
  }

  return stack.join('/');
}
```

## egg controller 入参改造

基于 [egg-router-plus](https://github.com/eggjs/egg-router-plus) 插件的 namespace 方法拓展，改造 [proxyFn](https://github.com/eggjs/egg-router-plus/blob/cdfec8012f8e051a6b55d848cd4112191e566085/lib/router.js#L78) 函数，将 ctx.query、ctx.request.body、ctx.params 合并到第一个参数中，并将 ctx 作为第二个参数传入，return 的内容赋值给 ctx.body, 方便统一处理返回结果。这样在写 controller 时不管是 get/post 请求，都可以直接从第一个参数中解构获取到所有参数，并且在返回结果时，直接 return 即可，无需再手动赋值给 ctx.body。

```js
const apiController = new Proxy(call_fn, {
  apply(targetFn, ctx) {
    const callArgs1 = { ...ctx.query, ...ctx.request.body, ...ctx.params };
    const callArgs2 = { ctx };
    return Reflect.apply(targetFn, ctx, [callArgs1, callArgs2])
      .then(res => {
        if (!ctx.body) ctx.body = res;
      }, fail => {
        throw fail;
      })
  }
})

// 使用示例：
async queryOrderInfo({ orderId }, { ctx }) {
  const { workOrderNumbers = [] } = ctx.User;
  if (!workOrderNumbers.includes(orderId)) {
    return { code: 403, message:'无权限查看订单信息' };
  }
  const data = await ctx.service.work.queryOrderInfo(orderId);
  return { code: 200, data };
}
```

## egg + pm2 + pkg 线上部署及打包

我的示例工程: https://gitee.com/yonatan/egg-example.git

pm2 入口文件, 在 pkg 同样适用

```js
const egg = require('egg');
const minimist = require('minimist');

const options = minimist(process.argv.slice(2));

egg.startCluster({
  baseDir: __dirname,
  ...options
});
```

[Egg.js 进程管理为什么没有选型 PM2 ？](https://www.zhihu.com/question/298718190)


## 批量 require 指定目录下的 *.Controller.js 文件

```js
const controllers = require('require-all')({
  dirname: path.join(__dirname, './controllers'),
  filter: /(.+Controller)\.js$/,
  resolve: (controller) => {
    return controller(app);
  }
});

if (!app.Controller) app.Controller = {};
app.Controller = { ...app.Controller, ...controllers };
```

## gulp 混淆加密

使用方式：npx gulp --srcDir=/path/to/project_dir

```js
// gulpfile.js
const gulp = require('gulp');
const { series } = gulp;
const del = require('del');
const javascriptObfuscator = require('gulp-javascript-obfuscator');
let minimist = require('minimist');
let options = minimist(process.argv.slice(2));
let { srcDir } = options;
const distDir = ['dist'];

function clean(cb) {
  return del([
    `output/${srcDir}/**/*`
  ], cb);
}

function buildServer(cb) {
  let output = `output/${srcDir}`;

  gulp.src([`source/${srcDir}/**/*.*`, `!source/${srcDir}/**/*.js`])
    .pipe(gulp.dest(`${output}`).on("end", cb));

  gulp.src([`source/${srcDir}/**/*.js `, ...distDir.map(dir => `!source/${srcDir}/${dir}/**/*.js`)])
    .pipe(javascriptObfuscator({
      compact: true,
      controlFlowFlattening: false,
      deadCodeInjection: false,
      debugProtection: false,
      debugProtectionInterval: false,
      disableConsoleOutput: false,
      identifierNamesGenerator: 'hexadecimal',//16机制变名
      log: false,
      renameGlobals: false,
      seed: 0,
      rotateStringArray: true,//
      selfDefending: true,//主动防御
      shuffleStringArray: true,//
      splitStrings: false,
      stringArray: true,//
      stringArrayEncoding: 'base64',
      stringArrayThreshold: 0.75,//
      unicodeEscapeSequence: false,
      target: 'node'
    }))
    .pipe(gulp.dest(`${output}`).on("end", cb));
}

function copyDist(cb) {
  distDir.forEach(dist => {
    let output = `output/${srcDir}/${dist}`;

    gulp.src([`source/${srcDir}/${dist}/**/*.js`])
      .pipe(gulp.dest(`${output}`).on("end", cb));
  })
}

exports.default = series(clean, buildServer, copyDist);
```

## UMD 工作原理

AMD 和 CommonJS 的综合，AMD 通常在浏览器端使用，异步加载模块，CommonJS 通常在服务端使用，同步加载模块，为了能兼容，所以想出了这么一个通用的解决方案

```js
(function(root, factory){
  // 先判断是否支持AMD(define是否存在)
  if(typeof define === 'function' && define.amd) {
    // 如果有依赖，例如：define(['vue'], factory); 下面同理，采用对应的模块化机制，加载依赖项
    define(factory);
  // 判断是否支持CommonJS模块格式    
  } else if(typeof module === 'object' && module.exports) {
    module.exports = factory();
  // 前两种都不存在，将模块挂载到全局(window或global)  moduleName代表模块名称 
  } else {
    root.[moduleName] = factory();
  }
}(this, function(){
  return {
    test: function(){}
  };
}))
```

## 检测图片是否存在

```js
const imageIsExist = (url) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => {
      if (this.complete == true){
        resolve(true);
        img = null;
      }
    }
    img.onerror = () => {
      resolve(false);
      img = null;
    }
    img.src = url;
  })
}
```

## iframe实现全屏，自适应浏览器高度

```html
<iframe id="iframe"
        name="iframe"
        height="100%"
        width="100%"
        src="https://notes.yonatan.cn"
        scrolling="auto"
        frameborder="0"
        onload="changeFrameHeight()">
</iframe>

<script>
  function changeFrameHeight() {
    var iframe = document.getElementById("iframe");
    iframe.height = document.documentElement.clientHeight;
  }
  window.onresize = function() {
    changeFrameHeight();
  }
</script>
```

> https://blog.csdn.net/qq_40542534/article/details/111238522

## js 清空数组

看到一处注释，“**千万不要**在布局函数里面对routes进行重新赋值，如`routes=[]`，但是你可以使用`routes.length=0`来清空路由数组，然后重新添加路由对象到数组里，总之，不要改变routes对象的指针指向。”

引起了我的思考。以往也在别的文章看过 arr.length = 0 的操作，这个操作好吗？

已知有多种方法来清空一个已经存在的数组：

```js
var arr = [1,2,3,4];
```

**方法1： 直接赋值**

```js
arr = [];
```

将空数组直接赋值给变量 arr，新数组与原无引用关系。对新数组的操作不会影响原数组。性能最快，但改变了引用。

**方法2： 设置 length**

```js
arr.length = 0
```

直接将数组长度设置为0，速度次之。当"strict mode"严格模式时，因 arr.length 是只读的，此方法将不起作用。

**方法3： splice**

```js
arr.splice(0, arr.length)
```

这种方法会返回删除的所有元素，并形一个新的数组，保持对数组的引用。性能最慢。

也有其它诸如 pop 循环删除的方法，各有利弊，很难选出一个写法规范且一定不会错的方法，但这里性能考虑是可以放最后的，性能问题可以等出现时再优化，也许能从其它方面做优化。最后想说倒也不必纠结，开发无银弹，如果你的场景用哪个都没啥区别，就用着一个先，等到真的有问题出现再去优化，到时候看是引用问题，性能问题，亦或是代码规范问题。

## js 首字母大写

```js
const firstUpperCase = ([first, ...rest]) => first?.toUpperCase() + rest.join('');
```

## new.target 检测是否是 new 调用

```js
class A {
  constructor() {
    if (new.target === A) {
      throw new TypeError('A is not a constructor');
    }
  }
}

const a = new A(); // 报错 "A is not a constructor"
```

## 正则表达式的 new RegExp 与字面量方式

```js
// 字面量方式
const regex1 = /pattern/flags;

// 构造函数方式
const regex2 = new RegExp('pattern', 'flags');
const regex3 = new RegExp(/pattern/, 'flags');
```

注意事项：

1. 转义字符处理

    - 字面量 - 只需要一次转义。例如：/\d+/
    - 构造函数 - 需要两次转义（字符串转义 + 正则转义）。例如：new RegExp('\\d+')

2. 字面量在代码加载时编译，构造函数在代码运行时编译

3. 构造函数可以动态构建，字面量不行
