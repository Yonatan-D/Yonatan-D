# 检测网页更新后通知用户刷新页面

{docsify-my-updater}



伪代码

```js
// 页面访问开始轮询
startPolling()

// 监听切换标签页，显示时开始轮询，隐藏时停止轮询
window.addEventListener('visibilitychange', async () => {
    if (document.visibilityState === 'visible')
        startPolling()
    if (document.visibilityState === 'hidden')
        stopPolling()
})

// 监听网页聚焦，没有开始轮询就开始
window.addEventListener('focus', () => {
    if (noStartPolling)
        startPolling()
})

// 监听网页失焦，停止轮询
window.addEventListener('blur', () => {
    stopPolling()
})
```



