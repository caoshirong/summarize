# 工作日常总结——移动端

1. > 当 input 框的类型为 type=“number”时，会出现可输入字母 e 的情况（e 在数学中为 2.718），如果在此时用了.replace(/^\d/, '')函数，则会导致输入框无法正确输入，所以可以增加键盘 keypress 事件，使用：/[\d]/.test(String.fromCharCode(event.keyCode))来阻止 e 的输入

2. > 布局：移动端常用的按钮 sticky 底部，此时如果用定位实现则会出现当键盘弹起时会出现，底部按钮遮挡 input 框；因此最好的解决方法是设置 body{min-height:100vh;dipslay:flex} .main{flex:1};从而实现当主内容区域较少时，底部按钮居于底部，而内容较多时，自动撑开

3. > 安卓下拉刷新问题：body{height：100vh;overflow:hidden}

4. > 禁止 ios 上缩放：以下存在一定缺陷
   ```
   document.addEventListener('touchstart', function (event) {
     if (event.touches.length > 1) {
       event.preventDefault()
     }
   })
   document.addEventListener('gesturestart', function (event) {
     event.preventDefault()
   })
   ```
5. > 跨域配置问题：
    ```
    1、接口响应头中的Set-Cookie字段中的path参数
    2、baseURL 的配置路径需要头部匹配path
    3、根据环境变量配置不同的转发参数地址
    ```
