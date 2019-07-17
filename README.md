# Vue开发记录
*前期先以收集遇到的问题以及新的知识点为主，后面数量多起来了再进行总结分类，以及提出更优美的解决方式*
1. webapp调用安卓或者IOS方法的时候，判断移动端环境
```javascript
    check(){
        let u = navigator.userAgent; 
        let isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1;   //判断是否是 android终端
        let isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);     //判断是否是 ios终端
        if(isAndroid === true){
            <!-- 执行业务代码，调用安卓方法 -->
            return 'Android';
        }else if(isIOS === true){
            <!-- 执行业务代码，调用IOS方法 -->
            return 'IOS';
        }else{
            return 'PC';
        }
    }
```
2. 移动端input框兼容IOS(解决IOS设备上面input框上移之后不恢复问题)
```javascript
*针对Vue，在main.js文件引入一下内容
// 解决在IOS设备上面，input框聚焦之后上移，失去焦点之后未恢复的bug问题
(/iphone|ipod|ipad/i.test(navigator.appVersion)) && document.addEventListener(
  'blur',
  event => {
    // 当页面没出现滚动条时才执行，因为有滚动条时，不会出现这问题
          // input textarea 标签才执行，因为 a 等标签也会触发 blur 事件
      if (
          document.documentElement.offsetHeight <=
          document.documentElement.clientHeight &&
          ['input', 'textarea'].includes(event.target.localName)
      ) {
          document.body.scrollIntoView() // 回顶部
      }
  },
  true
)
// 样式文件中如果出现以下样式，就注释掉
    -moz-user-select:none;/*火狐*/
    -webkit-user-select:none;/*webkit浏览器*/
    -ms-user-select:none;/*IE10*/
    -khtml-user-select:none;/*早期浏览器*/
    user-select:none;
```
3. 记一次vue.js的中央事件总线event bus重复执行的bug
```javascript
//需求分析:做商城的时候，商品一级分类list列表页面进入某一个一级分类下的页面，页面包含该一级分类的二级分类tab和商品列表，需要在进入这个页面的时候，就默认显示第一个二级分类下面的商品
#处理方式是在进入该页面的时候，当获取到二级分类的时候，就发送二级分类tabs的第一个id，通过事件发射到商品组件，商品组件接收到id进行商品数据请求。这样就会导致商品数据接口进行n+1次请求。
// 事件总线是在App.vue中new一个空的Vue赋值

//发射

*该业务涉及到页面的跳转，中央事件在页面销毁的时候不会自动销毁，需要手动销毁，可以在生命周期beforeDestroy或者destroyed销毁都可以



