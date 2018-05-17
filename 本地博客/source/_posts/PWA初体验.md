---
title: PWA初体验
date: 2018-05-17 09:24:21
tags:  # PWA,web
keywords: # PWA,web
---

## 一：什么是PWA
* PWA（ 全称：Progressive Web App ）也就是说这是个渐进式的网页应用程序。它主要有三个技术：
    * Service Worker（ ps：就叫做中间服务商 ）
    * Manifest （应用清单，网页应用程序的图标、是否全屏，启动动画等配置文件）
    * Push Notification（网页应用程序的推送通知）

## 二：Service Worker和Manifest示例代码

![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验1.jpg)
* 一个小demo
    * 代码结构
         * 1、index.hml
         * 2、main.css
         * 3、main1.css
         * 4、swa.js
         * 5、sw-toolbox.js，eat.png 用于缓存的文件
         * 6、manifest.json 应用清单

* index.hml
```python
  <!DOCTYPE html>
  <html>
  	<head>
  	<meta charset="utf-8">
  	<title>hello PWA</title>
  	<meta name="viewport" content="width=device-width,user-scalable=no">
  		<link rel="stylesheet" href="main.css" type="text/css" >
 <link rel="manifest" href="manifest.json">
  	</head>
  	<body>
  		<h3>hello PWA</h3>

  <p>new text</p>
  	</body>
  	<script>

   //检查当前浏览器是否支持 Service Workers
      if ('serviceWorker' in navigator) {  //如果支持，注册一个叫做 'sw.js' 的 Service Worker 文件
      // 注册 service worker
        navigator.serviceWorker.register('swa.js').then(
        	function(registration) {
          // 注册成功
          console.log('ServiceWorker registration successful with scope: ', registration.scope);
        }).catch(function(err) {
          // 注册失败 :(
          console.log('ServiceWorker registration failed: ', err);
        });
      }
  	</script>

  </html>
```
  * 2、main.css
```python
h3{color:#f00;}
```
  * 3、main1.css
```python
h3{color:blue;}
```
 * 4、swa.js
```python
//缓存文件
var cacheName = 'helloWorld';
self.addEventListener('install', function (event) {
    event.waitUntil(
        caches.open(cacheName)
            .then(function (cache) {
                    cache.addAll([
                        'sw-toolbox.js',
                        'eat.png'
                    ]).then(function (data) {
                        console.log('缓存成功',data)
                    })
                }
            ))
});

//中间商对请求进行拦截
self.addEventListener('fetch', function (event) {
    if (/\.css$/.test(event.request.url)) {
        event.respondWith(fetch('/main1.css'))

    }
});
```
 * 5、manifest.json
 ```python
{
	"name":"Li PWA",
	"short_name":"Li PWA",
	"display":"standalone",

	"background_color":"#aaaaff",
	"icons":[{
		"src":"eat.png",
		"sizes":"120x120",
		"type":"image/png"
		}]
}
 ```

## 三：发布网站，运行网站
    我使用的IIS发布
* 缓存文件sw-toolbox.js和eat.png

![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验3.png)
* 中间商拦截修改页面字体颜色，从红色改为蓝色

![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验4.png)

## 四：使用ngrok映射到外网
    ngrok使用教程
    https://hongmaju.github.io/2018/05/13/ngrok%E5%B0%86%E6%9C%AC%E5%9C%B0Web%E6%9C%8D%E5%8A%A1%E6%98%A0%E5%B0%84%E5%88%B0%E5%A4%96%E7%BD%91/#more
## 五：手机浏览器打开映射后的HTTPS地址，添加到主屏幕
![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验5.jpg)
![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验6.jpg)

## 六：一些解释
* Service Worker的安装和触发
    * 安装的代码再index.html中
    * 触发的代码再swa.js中

![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/PWA初体验2.png)

