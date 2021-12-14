---
title: 微信sdk封装调用(适配andriod，IOS)
cover: https://fpr.motaobox.com/407660594768699960
date: 2021-02-25 14:05:17
tags:
categories:
keywords:
description:
---

## 微信分享sdk调用
微信sdk调用参考官网文档就够了，不过有些坑点必须说下，针对SPA的app。
* 安卓必须每次url变化的时候签名
* IOS只能在落地页签名，落地页指的是第一次进入app的url。以后切换路由的时候不需要，也不能重新签名。
* 当前页面只能签名一次，多次签名也是不行的。

SDK 文件
```javascript
/**
 * desc: 微信新jssdk
 * author: Joe
 * date: 2021-02-24
 */
import { Toast } from 'vant';
const jsApiList = [
  'onMenuShareAppMessage',
  'onMenuShareTimeline',
  'updateAppMessageShareData',
  'updateTimelineShareData',
  'openLocation',
  'getLocation',
  'chooseImage',
  'uploadImage'
];

// ready 包装
const withReady = func => async (...args) => {
  const isReady = await ready();
  if (isReady) {
    func.apply(this, args);
  } else {
    Toast('微信配置异常');
  }
};

let instance = null;
// wx.ready
const ready = async () => {
  if (instance !== null) {
    // 已存在ready实例，直接返回对象
    return instance;
  }
  // 创建ready实例
  instance = new Promise(async function (resolve, reject) {
    if (process.env.NODE_ENV !== 'production') {
      resolve(true);
      return;
    }
    // 当前签名url
    const url = location.href;
    // 根据当前url请求后端获取签名
    let { success, data, msg } = await getSign({
      'jsUrl': url
    });
    if (success) {
      wx.config({
        debug: false,
        jsApiList: jsApiList, // 必填，需要使用的JS接口列表
        ...data
      });
      wx.ready(function () {
        resolve(true);
      });
      wx.error(function (res) {
        console.log('wx.error,', res);
        resolve(false);
      });
    } else {
      resolve(false);
      console.log('通过后端获取微信前面接口异常,', msg);
    }
  });
  return instance;
};

const resetReady = () => {
  if (device.isAndroid) {
    console.log('Android 重置ready');
    instance = null;
  }
};

// 获取定位
const getLocation = withReady(async (data) => {
  const { success, fail, complete, cancel, showLoading = true } = data;
  // 默认展开loading效果
  showLoading && Toast.loading({
    duration: 0, // 持续展示 toast
    forbidClick: true, // 禁用背景点击
    loadingType: 'spinner',
    message: '正在获取定位'
  });
  // 非打包环境直接返回结果
  if (process.env.NODE_ENV !== 'production') {
    setTimeout(() => {
      success({
        latitude: 22.51595,
        longitude: 113.3926
      });
      complete();
    }, 1000);
    showLoading && Toast.clear();
    return false;
  }
  wx.getLocation({
    type: 'gcj02',
    success: res => {
      console.log('wx.getLocation success', res);
      success && success(res);
    },
    fail: (error) => {
      console.log('wx.getLocation fail', error);
      fail && fail(error);
    },
    complete() {
      showLoading && Toast.clear();
      console.log('wx.getLocation complete');
      complete && complete();
    },
    cancel() {
      console.log('wx.getLocation cancel');
      cancel && cancel();
    }
  });
});

// 上传图片
const uploadImage = withReady(async callback => {
  wx.chooseImage({
    count: 1, // 默认9
    sizeType: ['original', 'compressed'],
    sourceType: ['album', 'camera'], 
    success: function (res) {
      var localIds = res.localIds; 
      wx.uploadImage({
        localId: localIds[0], 
        isShowProgressTips: 1, 
        success: function (res) {
          let json = {};
          var serverId = res.serverId; 
          json['serverId'] = serverId;
          json['url'] = localIds[0];
          callback(json);
        },
        fail: function (res) {
          console.log(res);
        }
      });
    },
    fail: function (res) {
      console.log(res);
    }
  });
});

// 打开定位
const openLocation = withReady(async data => {
  wx.openLocation(data);
});

const share = withReady(async (data = {}) => {
  const {
    title = '摩萄盒子',
    desc = '专业无人葡萄酒销售',
    link = location.href,
    imgUrl = 'https://sili-static.oss-cn-shenzhen.aliyuncs.com/icon_h5_share.png',
    success,
    cancel
  } = data;
  let shareFriend = wx.updateAppMessageShareData || wx.onMenuShareAppMessage;
  let shareTime = wx.updateTimelineShareData || wx.onMenuShareTimeline;
  let params = {
    title: title, // 分享标题
    desc: desc, // 分享描述
    link: link, // 分享链接 默认以当前链接
    imgUrl: imgUrl, // 分享图标
    success() {
      success && success();
    },
    cancel() {
      cancel && cancel();
    }
  };
  // 分享给朋友
  shareFriend(params);
  // 分享到朋友圈
  shareTime(params);
});

const pay = ({ signParams, success, fail }) => {
  function onBridgeReady() {
    WeixinJSBridge.invoke(
      'getBrandWCPayRequest', signParams,
      function (res) {
        if (res.err_msg === 'get_brand_wcpay_request:ok') {
          success && success(res);
        } else {
          res.msg = '支付失败';
          fail && fail(res);
        }
      }
    );
  }
  if (typeof WeixinJSBridge === 'undefined') {
    if (document.addEventListener) {
      document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
    } else if (document.attachEvent) {
      document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
      document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
    }
  } else {
    onBridgeReady();
  }
};

export {
  share,
  pay,
  getLocation,
  openLocation,
  uploadImage,
  resetReady
};

```

路由文件调用
```javascript
import Vue from 'vue';
import { resetReady, share } from 'utils/wechat-js-sdk';

export default router => {
  router.beforeEach((to, from, next) => {
    //重置签名实例
    resetReady();
    next();
  });

  router.afterEach((to, from, next) => {
    share();
    window.scrollTo(0, 0);
  });
};
```
