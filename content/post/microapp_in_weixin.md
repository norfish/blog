---
title: "微信小程序踩坑"
date: 2017-03-20T13:39:53+08:00
draft: false
---


日常开发中遇到的一些坑，写的比较简略，有些bug可能微信后续的版本已经修复，会有过时的风险，仅供参考

- 不支持important
- color不支持父级覆盖
- 如果视图滚动，则上层的vew背景色是半透明的，没法遮挡
- 事件绑定不支持delagate吧
- wx.login 不支持断网设置，会很慢，只能超时
- 获取网络状态，不管有无网络都是会触发success，只是networkType不同。没网的时候ios和adr返回值不同，分别为fail和none
- view 是 display：block。text是inline。只有text的可以选中
- input 如果value是通过data赋值的，同时又绑定了bindinput，有可能会触发循环，导致文字闪烁
- 按钮点击多次，对多次执行，比如页面跳转
- template的data传值,
- import src路径计算有bug ./item.wxml 和 item.wxml 是不一样的，用后者，否则就是../item.wxml
- setData 单次设置不能超过1024kb
- storage存储大小限制是每个10m
- 小程序中的组件样式是可以覆盖的，比如button，使用普通的class可以覆盖
- 小程序不支持webfont字体的外部引用，但是如果转成base64的话，是可以正常使用的- 