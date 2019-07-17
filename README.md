# Vue开发记录
*前期先以收集遇到的问题以及新的知识点为主，后面数量多起来了再进行总结分类，以及提出更优美的解决方式*
1. webapp调用安卓或者IOS方法的时候，判断移动端环境
```javascipt
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
    },
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
