#axios全局封装
```JavaScript
/**axios封装
 * 请求拦截、相应拦截、错误统一处理
 */
import axios from 'axios';
import QS from 'qs';
import router from '../router';
import store from '../store/store';
// vant的toast提示框组件
import { Toast,Dialog} from 'vant'; 
// 环境的切换
if (process.env.NODE_ENV == 'development') {    //开发环境 配置跨域
    axios.defaults.baseURL = '/api';//从main.js中引入的处理跨域的
} else if (process.env.NODE_ENV == 'debug') {  //测试环境  
    axios.defaults.baseURL = '';
} else if (process.env.NODE_ENV == 'production') {   //生产环境 
    axios.defaults.baseURL = '';
}
// 请求超时时间
axios.defaults.timeout = 2000;

// post请求头
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';
 
// 添加请求拦截器
axios.interceptors.request.use(    
    config => {
        // 在请求之前做点什么
        // 每次发送请求之前判断是否存在token，如果存在，则统一在http请求的header都加上token，不用每次请求都手动添加了
        // 即使本地存在token，也有可能token是过期的，所以在响应拦截器中要对返回状态进行判断
        const token = localStorage.getItem('token');        
        token && (config.headers.Authorization = token);  
        return config;    
    },    
    error => {    
        // 请求超时
        Toast.fail({
            message:'请求超时'
        })
        //对请求错误做些什么
        return Promise.error(error); 
    }
)
/** 
 * 提示函数 
 * 禁止点击蒙层、显示一秒后关闭
 */
const tip = msg => {    
    Toast({        
        message: msg,        
        duration: 1000,        
        forbidClick: true    
    });
}
/** 
 * 跳转登录页
 * 携带当前页面路由，以期在登录页面完成登录后返回当前页面
 */
const toLogin = () => {
    router.replace({
        path: '/login',        
        query: {
            redirect: router.currentRoute.fullPath
        }
    });
}

// 添加响应拦截器
axios.interceptors.response.use(    
    // 对响应数据做点什么
    response => {      
        // 与后台定义好的token失效的状态码
        if(response.data.code===101){
            Toast("您还未登录，请前往'我的'登录");                
        }else if(response.data.code===102){
            Toast({
                message:"登录过期，请前往'我的'重新登录",
                duration: 1000,  
            });
            toLogin();
            localStorage.removeItem('token');      
        }else if(response.data.code===103){
            Toast({
                message:"登录过期，请前往'我的'重新登录",
                duration: 1000,  
            });
            toLogin();
            localStorage.removeItem('token');       
        }else if(response.data.code===104){
            Toast({
                message:"登录过期，请前往'我的'重新登录",
                duration: 1000,  
            });
            toLogin();
            localStorage.removeItem('token');      
        }else if(response.data.code===105){
            Toast({
                message:"登录过期，请前往'我的'重新登录",
                duration: 1000,  
            });
            toLogin();
            localStorage.removeItem('token');      
        }else if(response.data.code===106){
            Dialog.alert({
                message:response.data.message,
                showCancelButton:false,
            }).then(()=>{
                router.push({
                    path: '/home',        
                    query: {
                        pagenum:1
                    }
                });
            })
        }else if(response.data.code===110){
            Toast(response.data.message);    
        }else if(response.data.code===120){
            Toast.fail("！积分不足");    
        }else if (response.status === 200){  
            return Promise.resolve(response);
        } else {       
            return Promise.reject(response);        
        }         
    },
     // 对响应错误做点什么  
    error => {    
        if (!error.response) {
            //断网处理
            
        }
        if (error.response.status) {            
            switch (error.response.status) {                
                // 401: 未登录                
                // 未登录则跳转登录页面，并携带当前页面的路径                
                // 在登录成功后返回当前页面，这一步需要在登录页操作。                
                case 401:                    
                    break;
                // 403 token过期                
                // 登录过期对用户进行提示                              
                // 跳转登录页面                
                case 403:                     
                    Toast({                        
                        message: '登录过期，请重新登录',                        
                        duration: 1000,                        
                        forbidClick: true                    
                    });                    
                    // 清除token                    
                    // localStorage.removeItem('token');                                       
                    // 跳转登录页面，并将要浏览的页面fullPath传过去，登录成功后跳转需要访问的页面
                    setTimeout(() => {                        
                        toLogin();                  
                    }, 1000);                    
                    break; 
                // 404请求不存在                
                case 404:                
                    Toast({                        
                        message: '网络请求不存在',                        
                        duration: 1500,                        
                        forbidClick: true                    
                    });                    
                break;
                case 500:                
                    Toast({                        
                        message: '加载出错啦,请重试',                        
                        duration: 1500,                        
                        forbidClick: true                    
                    });                    
                break;               
                // 其他错误，直接抛出错误提示                
                default:                    
                    Toast({                        
                        message: '数据加载失败啦' ,                       
                        duration: 1500,                        
                        forbidClick: true                    
                    });            
            }            
            return Promise.reject(error.response);        
        }   
    }
);
/** 
 * get方法，对应get请求 
 *  
 * 
 */
export function get(url, params){    
    return new Promise((resolve, reject) =>{        
        axios.get(url, {            
            params: params        
        })        
        .then(res => {            
            resolve(res.data);        
        })        
        .catch(err => {            
            reject(err.data)        
        })    
    });
}
/** 
 * post方法，对应post请求  **/
//  * @param {String} url [请求的url地址] 
//  * @param {Object} params [请求时携带的参数] 

export function post(url, params) {    
    return new Promise((resolve, reject) => {         
        axios.post(url, QS.stringify(params))        
        .then(res => {            
            resolve(res.data);
        })        
        .catch(err => {            
            reject(err.data);     
        })    
    });
}
```
#同意API管理
```JavaScript
import { get, post } from './http';

// -------------------------云品汇----------------
// 首页头部轮播图
let bannerUrl = '';
// 主题精选轮播图
let topicBannerUrl = '';
// 猜你喜欢
let guessYouLikeUrl = '';
// 足迹
let footPrintUrl = '';
// 删除足迹
let delFootPrintUrl = '';
// 添加足迹
let addFootPrintUrl = '';
// 地址列表信息
let addressListUrl = '';
// 查询地址信息
let readAddressUrl = '';
// 新增地址信息
let addAddressListUrl = '';
// 修改地址信息
let changeAddressListUrl = '';
// 删除地址信息
let delAddressListUrl = '';
// 新增搜索历史
let addHistoryUrl = '';
// 查询历史
let historyUrl = '';
// 删除历史
let delhistoryUrl = '';
// 补全搜索
let completeSearchUrl = '';
// 按名称搜索商品
let searchGoodsListUrl = '';
// 广告展示
let adShowUrl = '';
// 加入购物车
let addShopcarUrl = ''; 
// 修改购物车中商品数量
let goodsNumCarUrl = '';
// 查看购物车列表
let shopCarListUrl = '';
// 在购物车中下单
let shopCarOrdergoodsUrl = '';
// 获取商品运费
let goodsTransUrl = '';
// 单个商品下单
let orderNewUrl = '';
// 确认订单显示页面-订单详情-购买确认页面
let orderSureUrl = '';
// 取消订单
let cancelOrderUrl = '';
// 完成订单
let orderFinishUrl = '';
// 查看订单列表
let AllOrderUrl = '';
// 一级分类tab
let oneClassListUrl = '';
// 二级分类tab
let secondClassListUrl = '';
// 添加分类
let addClassGoodsUrl = '';
// 商品分类列表+所有商品列表
let goodsListUrl = '';
// 修改用户名
let changeUsernameUrl = '';
// 发送验证码
let yanCodeUrl = '';
// 修改手机号
let changembileUrl = '';
// 商品详情
let goodsDetailUrl = '';
// 规格筛选
let goodsSKuCheckUrl = '';
// 登录页登录
let loginUrl = '';
// 退出登录
let loginOutUrl = '';
// 获取初始收藏分类数据
let collectGoodsUrl = '';
// 根据分类id获取商品
let categrayGoodsUrl = '';
// 查看已有订单列表
let orderFinishListUrl = '';
// 我的详情
let ownDetailUrl = '';
// 限时购
let timeGoodsUrl = '';
// 主题精选
let topicGoodsUrl = '';
// 客服中心列表
let customerServerListUrl = '';
// 修改地址订单
let addressChangeUrl = '';
// 支付
let payUrl = '';
// 初始订单列表
let initOrderListUrl = '';
// 分页订单列表
let cancatOrderListUrl = '';

export const server = {
    // 暴露变量出去
    changeUsernameUrl,
    changeaddressUrl,
    chengeIdcardUrl,
    changembileUrl,
    
    // -------------------------云品汇--------------------
    // 首页头部轮播
    bannerAddress: function(bannerobj){
        return post(bannerUrl,bannerobj);
    },
    // 主题精选轮播图
    topicBannerAddress:function(topicBannerobj){
        return post(topicBannerUrl,topicBannerobj)
    },
    // 猜你喜欢
    guessYouLikeAddress:function(guessYouLikeobj){
        return post(guessYouLikeUrl,guessYouLikeobj)
    },
    // 显示足迹
    footPrintAddress:function(footPrintobj){
        return post(footPrintUrl,footPrintobj)
    },
    // 删除足迹
    delFootPrintAddress:function(delFootPrintobj){
        return post(delFootPrintUrl,delFootPrintobj)
    },
    // 添加足迹
    addFootPrintAddress:function(addFootPrintobj){
        return post(addFootPrintUrl,addFootPrintobj)
    },
    // 新增搜索历史
    addHistoryAddress:function(addHistoryobj){
        return post(addHistoryUrl,addHistoryobj)
    },
    // 查询搜索历史
    historyAddress:function(historyobj){
        return post(historyUrl,historyobj)
    },
    // 删除历史
    delhistoryAddress:function(delhistoryobj){
        return post(delhistoryUrl,delhistoryobj)
    },
    // 补全搜索
    completeSearchAddress:function(completeSearchobj){
        return post(completeSearchUrl,completeSearchobj)
    },
    // 按名称搜索商品
    searchGoodsListAddress:function(searchGoodsListobj){
        return post(searchGoodsListUrl,searchGoodsListobj)
    },
    // 一级分类tab
    ClassListAddress:function(ClassListobj){
        return post(oneClassListUrl,ClassListobj)
    },
    // 二级分类tab
    secondClassListAddress:function(secondClassListobj){
        return post(secondClassListUrl,secondClassListobj)
    },
    // 首页商品列表+分类商品
    goodsListAddress:function(goodsListobj){
        return post(goodsListUrl,goodsListobj)
    },
    // 地址列表信息
    addressListAddress:function(addressListobj){
        return post(addressListUrl,addressListobj)
    },
    // 修改地址
    changeAddressListAddress:function(changeAddressListobj){
        return post(changeAddressListUrl,changeAddressListobj)
    },
    // 新增地址
    addAddressListAddress:function(addAddressListobj){
        return post(addAddressListUrl,addAddressListobj)
    },
    // 删除地址
    delAddressListAddress:function(delAddressListobj){
        return post(delAddressListUrl,delAddressListobj)
    },
    // 查询地址
    readAddressSingle:function(readAddressobj){
        return post(readAddressUrl,readAddressobj)
    },
    // 修改用户名
    changeUsernameAddress:function(changeUsernameobj){
        return post(changeUsernameUrl,changeUsernameobj)
    },
    // 发送验证码
    yanCodeAddress:function(yanCodeobj){
        return post(yanCodeUrl,yanCodeobj)
    },
    // 修改手机号
    changembileAddress:function(changembileobj){
        return post(changembileUrl,changembileobj)
    },
    // 商品详情
    goodsDetailAddress:function(goodsDetailobj){
        return post(goodsDetailUrl,goodsDetailobj)
    },
    // 规格筛选
    goodsSKuCheckAddress:function(goodsSKuCheckobj){
        return post(goodsSKuCheckUrl,goodsSKuCheckobj)
    },
    // 单个商品直接下单
    orderNewAddress:function(orderNewobj){
        return post(orderNewUrl,orderNewobj)
    },
    // 加入购物车
    addShopcarAddress:function(addShopcarobj){
        return post(addShopcarUrl,addShopcarobj)
    },
    // 修改购物车商品数量
    goodsNumCarAddress:function(goodsNumCarobj){
        return post(goodsNumCarUrl,goodsNumCarobj)
    },
    // 查看购物车列表
    shopCarListAddress:function(shopCarListobj){
        return post(shopCarListUrl,shopCarListobj)
    },
    // 在购物车下单
    shopCarOrdergoodsAddress:function(shopCarOrdergoodsobj){
        return post(shopCarOrdergoodsUrl,shopCarOrdergoodsobj)
    },
    // 登录
    loginAddress:function(loginobj){
        return post(loginUrl,loginobj);
    },
    // 退出登录
    loginOutAddress:function(loginOutobj){
        return post(loginOutUrl,loginOutobj)
    },
    // 查看订单详情+确认订单
    orderSureAddress:function(orderSureobj){
        return post(orderSureUrl,orderSureobj)
    },
    // 获取初始收藏分类数据
    collectGoodsAddress:function(collectGoodsobj){
        return post(collectGoodsUrl,collectGoodsobj)
    },
    // 根据分类id获取商品
    categrayGoodsAddress:function(categrayGoodsobj){
        return post(categrayGoodsUrl,categrayGoodsobj)
    },
    // 查看积分兑换订单-订单列表
    orderFinishListAddress:function(orderFinishLisobj){
        return post(orderFinishListUrl,orderFinishLisobj)
    },
    // 我的详情
    ownDetailAddress:function(ownDetailobj){
        return post(ownDetailUrl,ownDetailobj)
    },
    // 限时购
    timeGoodsAddress:function(timeGoodsobj){
        return post(timeGoodsUrl,timeGoodsobj)
    },
    // 主题精选
    topicGoodsAddress:function(topicGoodsobj){
        return post(topicGoodsUrl,topicGoodsobj)
    },
    // 客服中心文章列表
    customerServerListAddress:function(customerServerListobj){
        return post(customerServerListUrl,customerServerListobj)
    },
    // 修改订单地址
    addressChangeAddress:function(addressChangeobj){
        return post(addressChangeUrl,addressChangeobj)
    },
    // 支付
    payAddress:function(payobj){
        return post(payUrl,payobj)
    },
    // 初始订单列表
    initOrderListAddress:function(initOrderListobj){
        return post(initOrderListUrl,initOrderListobj)
    },
    // 获取分页订单列表
    cancatOrderListAddress:function(cancatOrderListobj){
        return post(cancatOrderListUrl,cancatOrderListobj)
    },

     
    // --------------------------德合汇-------------------
    // 首页商城文章
    marketArticleAddress:function(marketArticleobj){
        return post(marketArticleUrl,marketArticleobj)
    },
    // 文章详情
    articleDetailAddress:function(articleDetailobj){
        return post(articleDetailUrl,articleDetailobj)
    },
    // 首页文章推荐
    articleRecAddress:function(articleRecobj){
        return post(articleRecUrl,articleRecobj)
    },
    
    
    // 首页消息数量
    newsNumAddress:function(newsNumobj){
        return post(newsNumUrl,newsNumobj)
    },
    // 我的
    myAddress:function(myobj){
        return post(myUrl,myobj)
    },
   
    // 修改用户名  修改地址 修改身份证
    changeUser:function(changeUserUrl,changeUserobj){
        return post(changeUserUrl,changeUserobj)
    },
    
   
    // 新增-取消收藏
    newcollectAddress:function(newcollectobj){
        return post(newcollectUrl,newcollectobj)
    },
    
    
    // 查看积分
    coreAddress:function(coreobj){
        return post(coreUrl,coreobj)
    },
    // 获取用户部分信息
    userPartInfoAddress:function(userPartInfoobj){
        return post(userPartInfoUrl,userPartInfoobj)
    },
    
    
    
    
    
    
    
    // 完成订单
    orderFinishAddress:function(orderFinishobj){
        return post(orderFinishUrl,orderFinishobj)
    },
    // 取消订单
    cancelOrderAddress:function(cancelOrderobj){
        return post(cancelOrderUrl,cancelOrderobj)
    },
    // 查看全部订单
    AllOrderAddress:function(AllOrderobj){
        return post(AllOrderUrl,AllOrderobj)
    },
    
    // 获取商品运费
    goodsTransAddress:function(goodsTransobj){
        return post(goodsTransUrl,goodsTransobj)
    },
    
    
    // 系统提示信息
    investMessageAddress:function(investMessageobj){
        return post(investMessageUrl,investMessageobj)
    },
    
    // 修改地址
    changeaddressAddress:function(changeaddressobj){
        return post(changeaddressUrl,changeaddressobj)
    },
}
```
