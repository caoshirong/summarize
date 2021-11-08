# 工作日常总结——移动端

` ` `1、当input框的类型为type=“number”时，会出现可输入字母e的情况（e在数学中为2.718），如果在此时用了.replace(/^\d/, '')函数，则会导致输入框无法正确输入，所以可以增加键盘keypress事件，使用：/[\d]/.test(String.fromCharCode(event.keyCode))来阻止e的输入` ` `

` ` `2、布局：移动端常用的按钮sticky底部，此时如果用定位实现则会出现当键盘弹起时会出现，底部按钮遮挡input框；因此最好的解决方法是设置body{min-height:100vh;dipslay:flex} .main{flex:1};从而实现当主内容区域较少时，底部按钮居于底部，而内容较多时，自动撑开` ` `

` ` `3、安卓下拉刷新问题：body{height：100vh;overflow:hidden}` ` `
