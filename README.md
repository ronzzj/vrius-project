





### 疫情播报微信小程序 

####介绍

针对2019年01月份爆发的新型管状病毒 2019-nCov, 开发的一个用于获取疫情消息的微信小程序。

包括疫情病例和疫情新闻实时播报、病毒防护知识分享。

#### 小程序截图

<img src="/Users/zhouzejia/Desktop/截屏2020-02-0512.35.19.png" alt="截屏2020-02-0512.35.19" style="zoom: 50%;" />

<img src="/Users/zhouzejia/Desktop/截屏2020-02-0512.35.52.png" alt="截屏2020-02-0512.35.52" style="zoom: 50%;" />

<img src="/Users/zhouzejia/Desktop/截屏2020-02-0516.20.55.png" alt="截屏2020-02-0516.20.55" style="zoom: 50%;" />

#### 数据接口API

- #####请求地址 http://api.tianapi.com/txapi/ncov/index

- #####请求方式 get/post

- #####请求参数

| 请求参数 | 类型   | 必填 | 参数位置 | 示例值           | 备注说明 |
| :------- | :----- | :--- | :------- | :--------------- | :------- |
| key      | string | 是   | urlParam | 用户自己的APIKEY | API密钥  |

- #####返回参数

| ews      | array | 新闻列表数组 | 资讯动态列表                      |
| :------- | :---- | :----------- | :-------------------------------- |
| 返回参数 | 类型  | 示例值       | 备注说明                          |
| case     | array | 病例列表数组 | 全国各省市确诊/疑似病例等具体数据 |
| desc     | array | 概况信息数组 | 传播/地图/疫情概况                |

####UI 框架

- ##### Color UI库

- #####GitHub 地址 https://github.com/weilanwl/ColorUI.git

#### 问题分享

- ##### 语言切换

在微信小程序项目utils文件夹下，创建thrid js文件, 通过module.exports暴露值。

```js
var Chinese = {
  navigationText: '新型冠状病毒',
  tab1: '实时疫情',
  tab2: '新闻播报',
  tab3: '疾病常识',
  tab4: '关于',
}
module.exports = {
  Chinese: Chinese
}
```

引入utils下的thrid js， 通过简单判断当前语言状态，设置相应的内容即可。

```js
/*import utils*/
var utils_CN = require('../../utils/chinese');
var utils_TA = require('../../utils/tibetan');
var app = getApp();
Page({
  /*get content of current language*/
  getContent: function (lastLanuage) {
    var that = this
    if (lastLanuage == 'zh_CN') {
      that.setData({
        content: utils_CN.Chinese,
      })
    } else if (lastLanuage == 'zh_TB') {
      that.setData({
        content: utils_TA.Tibetan
      })
    }
  }
})
```

最终在wxml页面上显示

```js
<text class="cuIcon-new">{{content.tab1}}</text>
```

- #####view组件文字溢出

对于微信小程序的view组件的文字溢出，可以使用下面的样式解决，超过行数的文字会被截断。

```js
.textView{
  display: -webkit-box;  /*设置为弹性盒子*/
  -webkit-line-clamp: 18;  /*最多显示5行*/
  overflow: hidden;  /*超出隐藏*/
  text-overflow: ellipsis;  /*超出显示为省略号*/
  -webkit-box-orient: vertical;
  word-break: break-all;  /*强制英文单词自动换行*/
}
```

- ##### 使用本地缓存

| 方法                            | 参数                    | 说明                              |
| :------------------------------ | :---------------------- | --------------------------------- |
| wx.getStorageSync('key')        | key在本地缓存的唯一标识 | 读取key='key'的本地缓存的内容     |
| wx.setStorageSync('key', value) | Key,value要设置的值     | 写入key='key'的本地缓存,值为value |

场景： 用户设置语言后，需要存储设置好的语言。并在下次打开时，显示用户上一次设置的语言。

方法： 在app.js中设置语言状态全局变量，用户设置语言后写入本地缓存，下次打开时读取缓存即可。

```js
//app.js
App({
  onLaunch: function () {
  },
  globalData: {
    userInfo: null,
    /*language setting*/
    language: wx.getStorageSync('lang'),
    isChecked:wx.getStorageSync('checked')
  }
})
```

本地缓存为空时，设置默认语言，写入本地缓存

```js
/*improt app.js*/
var app = getApp();
Page({
  onLoad: function (options) {
    /*read chack*/
    if (wx.getStorageSync('lang') == '') {
      wx.setStorageSync('lang', 'zh_TB'),
        app.globalData.language = 'zh_TB',
    }
  },
})
```

切换语言

```js
/*switch language*/
  switchLanguage: function (e) {
    if (e.detail.value == true) {
      wx.setStorageSync('lang', 'zh_TB')
      app.globalData.language = "zh_TB"
    } else {
      wx.setStorageSync('lang', 'zh_CN')
      app.globalData.language = "zh_CN"
    }
    /*get content of current language*/
    this.getContent(app.globalData.language)
  },
```

- ##### 列表渲染点击事件

场景：使用微信小程序列表渲染后， 点击某一项，获取点击的内容

方法：数据需要唯一标识的id, 在wxml设置 data-id="{{item.id}}", 事件函数内通过 event.currentTarget.dataset.id获取id。

```js
//数据
	articleList: [{
      id: 0,
      title: 'title',
      content: 'content'
    },
    {
      id: 1,
      title: 'title',
      content: 'content'
    }
```

```html
<view wx:for="{{content.articleList}}" wx:key="id">
  <text>{{item.title}}</text>
  <view class="text-content textView"><text>{{item.content}}</text></view>
  <view class="cu-tag bg-green light sm round" data-id="{{item.id}}" bindtap="viewContent">	  {{content.details}}
</view>
</view>
```

```js
 viewContent: function (event) {
    wx.navigateTo({
      url: '../article/article?id=' + event.currentTarget.dataset.id
    })
  },
```

