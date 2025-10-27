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
