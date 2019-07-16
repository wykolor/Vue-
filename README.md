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
    },```
