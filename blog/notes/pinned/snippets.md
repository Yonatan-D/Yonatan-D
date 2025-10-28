# Snippets

## 重写全局对象里的方法函数

以微信小程序 wx 对象为例，重写 showLoading 和 hideLoading 函数，在调用时进入断点并输出文件位置，方便调试

```js
wx = wxProxy(wx);

function wxProxy(wx) {
  const newWx = { ...wx };

  // 代理showLoading/hideLoading
  const loadingApiProxy = (loadingApi) => {
    return (object) => {
      console.trace();
      debugger
      return loadingApi(object);
    }
  }

  // 代理wx.showLoading
  newWx.showLoading = loadingApiProxy(wx.showLoading);

  // 代理wx.hideLoading
  newWx.hideLoading = loadingApiProxy(wx.hideLoading);

  return newWx;
}
```

更多用法（解决加载过程中loading重复刷新、路由跳转拦截）：https://github.com/Yonatan-D/wechat-snippets

## egg controller 入参改造

[egg-router-plus](https://github.com/eggjs/egg-router-plus/blob/master/lib/router.js)

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
  let err = null;
  if (!workOrderNumbers.includes(orderId)) {
    err = '无权限查看订单信息';
  }
  const data = await ctx.service.work.queryOrderInfo(orderId);
  return [err, data];
}
```

## 批量 require 目录下的 *.Controller.js 文件

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

npx gulp --srcDir=/path/to/project_dir

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
<!-- https://blog.csdn.net/qq_40542534/article/details/111238522 -->

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
