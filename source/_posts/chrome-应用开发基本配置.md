---
title: chrome 应用开发基本配置
date: 2019-07-12 16:22:59
categories: 
- web前端
tags:
- chrome
---

## chrome应用简介

### 什么是chome应用？

Chrome 应用提供了与原生应用能力相同的体验，但是与网页一样安全。就像网上应用一样，Chrome 应用使用 HTML5、JavaScript 和 CSS 编写，但是 Chrome 应用从外观上与行为上都与原生应用类似，它们也具有类似于原生应用的能力，比网上应用可用的更强大。

### chrome 应用下载

打开`chrome`浏览器后，点击
![chrome商店](/images/chrome-extension/1.jpeg)
后，再点击应用商店进入[chrome 网上应用店](https://chrome.google.com/webstore/category/extensions)，就可以下载自己喜欢的应用了。

### 常用的chrome 应用

* [Adblock Plus 拦截广告](https://chrome.google.com/webstore/detail/adblock-plus-free-ad-bloc/cfhdojbkjhnklbpkdaibdccddilifddb?utm_source=chrome-ntp-icon) 
* [Postman Interceptor](https://chrome.google.com/webstore/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo?utm_source=chrome-ntp-icon)
* [web 前端助手](https://chrome.google.com/webstore/detail/web%E5%89%8D%E7%AB%AF%E5%8A%A9%E6%89%8Bfehelper/pkgccpejnmalmdinmhkkfafefagiiiad?utm_source=chrome-ntp-icon)
* [划词翻译](https://chrome.google.com/webstore/detail/%E5%88%92%E8%AF%8D%E7%BF%BB%E8%AF%91/ikhdkkncnoglghljlkmcimlnlhkeamad?utm_source=chrome-ntp-**icon**)
![fehelper](/images/chrome-extension/2.jpeg)

## 开发与调试

直接输入 [chrome://extensions/](chrome://extensions/)，开启开发者模式
![开发者模式](/images/chrome-extension/3.jpg)，右键选择你要调试的应用点击
![审查元素](/images/chrome-extension/4.jpg) 后面的程序就和普通页面调试一模一样了。

## 核心文件 manifest.json

```json
{
// 必须的字段
  // 应用名称
  "name": "My Extension",
  // 应用版本
  "version": "1.0.0",
  // manifest_version
  "manifest_version": 2,

// 建议提供的字段
  // 应用描述
  "description": "A plain text description",
  // 图标
  "icons": { 
    "16": "icon16.png",             
    "48": "icon48.png",            
    "128": "icon128.png" 
   },
   // 语言
  "default_locale": "en",
  
  // 浏览器右上角图标设置，browser_action、page_action、app三选一或者都不选
  "browser_action": {
    "default_icon": {                    // optional
      "16": "images/icon16.png",           // optional
      "24": "images/icon24.png",           // optional
      "32": "images/icon32.png"            // optional
    },
    "default_title": "Google Mail",      // optional; shown in tooltip
    "default_popup": "popup.html"        // optional
  },
  // 当某些特定页面打开才显示的图标
  "page_action": {
    "default_icon": "icons/foo.png", // optional 
    "default_title": "Do action",    // optional; shown in tooltip 
    "default_popup": "popup.html"    // optional 
  },
  
  
// 根据需要提供
  // 从启动浏览器开始，整个生命周期都存在
  // 一般仅仅需要js文件,浏览器的扩展系统会自动根据上面scripts字段指定的所有js文件自动生成背景页，也可指定背景页。
  "background": {
    "scripts": ["background.js"],
    "persistent": false
    // "page": "background.html"
  },
  // 可以将Chrome默认的一些特定页面替换掉 (书签管理器chrome://bookmarks 、历史记录 chrome://history、空白标签页chrome://newtab)
  "chrome_url_overrides": {
    "newtab": "myPage.html"
  },
  // 嵌入到页面内运行的javascript脚本,只可访问页面的dom和修改。
  "content_scripts": [
    {
       // "<all_urls>" 表示匹配所有地址
      "matches": ["http://*/*", "https://*/*"],
      // "matches": ["<all_urls>"],
      "css": ["mystyles.css"],
      "js": ["jquery.js", "myscript.js"],
      "run_at": "document_start",  // document_start || document_end || document_idle 默认（document_end和发出window.onload事件之间的某个时机注入）
    }
  ],
  // 安全策略
  "content_security_policy": "policyString",
  // 上传（需要加权限 permissions: fileBrowserHandler）
  "file_browser_handlers": [
    {
      "id": "upload",
      "default_title": "保存至照片库", // 按钮上显示的文字
      "file_filters": [
        "filesystem:*.jpg", // 要匹配所有文件，请使用 "filesystem:*.*"
        "filesystem:*.jpeg",
        "filesystem:*.png"
      ]
    }
  ],
  // devtools页面入口，只能是html
    "devtools_page": "devtools.html",
  // 扩展的主页 url
  "homepage_url": "http://path/to/homepage",
  // 可选值："spanning"和"split"，指定当扩展在允许隐身模式下运行时如何响应。
  "incognito": "spanning" or "split",
  // 开发时为扩展指定的唯一标识值(不需要设置)
  "key": "publicKey",
  // 最低版本
  "minimum_chrome_version": "versionString",
  // 指定本扩展或app是否支持脱机运行。
  "offline_enabled": true,
  // 关键词搜索匹配
  "omnibox": { "keyword": "aaron" },
  // 选项页
  "options_page": "aFile.html",
  // 权限
  "permissions": [
    "tabs", // 标签
    "bookmarks", // 书签
    "http://www.blogger.com/",
    "http://*.google.com/",
    "contextMenus", // 右键菜单
    "notifications", // 通知
    "webRequest", // web请求
    "cookies",
    "background",
    "history",
    "storage", // 插件本地存储
    "http://*/*", // 可以通过executeScript或者insertCSS访问的网站
    "https://*/*" // 可以通过executeScript或者insertCSS访问的网站
  ],
  // 自动升级url
  "update_url": "http://path/to/updateInfo.xml",
  // 指定本扩展在注入的目标页面上所需使用的资源的路径
  "web_accessible_resources": [
    "images/my-awesome-image1.png",
    "images/my-amazing-icon1.png",
    "style/double-rainbow.css",
    "script/double-rainbow.js"
  ]
}
```

## 各js权限

| 类型 | 可访问的api | 是否可访问DOM | 是否可访问JS | 是否可跨域 |
| :--- | :---------- || :-----| :----- | :----- || :-----|
| content_script | 只能访问 extension、runtime等部分API  | Y | N | N |
| popup.js | 可访问绝大部分API，除了devtools系列  | N | N | Y |
| background.js | 可访问绝大部分API，除了devtools系列  | N | N | Y |
| devtools.js | 只能访问 devtools、extension、runtime等部分AP  | Y | Y | N |


## 通信能力

|      | injected-script | content-script | popup-js | background-js |
| :--- | :-------------- || :-----| :----- | :----- || :-----|
| injected-script | - | window.postMessage | - | - |
| content-script | window.postMessage | - | chrome.runtime.sendMessage  chrome.runtime.connect | chrome.runtime.sendMessage  chrome.runtime.connect |
| popup-js | - | chrome.tabs.sendMessage chrome.tabs.connect | - | chrome.extension. getBackgroundPage() |
| background-js | - | chrome.tabs.sendMessage chrome.tabs.connect | chrome.extension.getViews | - |
| devtools-js | chrome.devtools. inspectedWindow.eval | - | chrome.runtime.sendMessage | chrome.runtime.sendMessage |

## 参考链接
* [chrome 应用开发官网](https://developer.chrome.com/extensions)
* [360浏览器 应用开发官网](http://open.chrome.360.cn/extension_dev/permissions.html)
* [chrome 开发指南](https://crxdoc-zh.appspot.com/extensions/devguide)
* [【干货】Chrome插件(扩展)开发全攻略](https://www.cnblogs.com/liuxianan/p/chrome-plugin-develop.html)