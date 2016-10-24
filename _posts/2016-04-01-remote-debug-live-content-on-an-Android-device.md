---
layout: post
category : lessons
title: "Chrome 远程调试 Android 设备网页【译】"
tagline: "Supporting tagline"
tags : [chrome]
---

### 目录

- 前提条件
- 开启 Android 设备的 USB 调试
- 连接并发现 Android 设备
- 在开发机上调试 Android 设备上的内容
- 映射 Android 设备内容到开发机

### 在这之前
之前远程调试需要借助 Android SDK 和 adb，步骤很繁琐。参见：https://developer.chrome.com/devtools/docs/remote-debugging-legacy

## 前提条件

- 开发机安装 chrome 32 或更新版本
- 如果开发机是 windows，则需安装 USB drivers
- 连接开发机和 Android 设备的 USB 线
- Android 4.0 或更新版本
- Android 设备安装 chrome 浏览器

## 开启 Android 设备的 USB 调试
设置 --> 开发者选项 --> USB 调试

Android 4.2 及更新版本开启方式，参见：http://developer.android.com/tools/device.html#device-developer-options

## 连接并发现 Android 设备

- 开发机上，打开 chrome 并登陆。在游客模式下，远程调试不可用
- 打开 DevTools，选择 More tools --> Inspect devices，打开 Inspect Devices 对话框，启用 Discover USB devices。
- 连接开发机和 Android 设备。
- 如果是第一次连接，你会在 Inspect Devices 对话框看到 unknown device that is pending authorization。这时 Android 设备应该会弹出 Allow USB debugging，点击确认。

	tips: 如果在发现设备过程中遇到任何问题，可以到 Android 设备的开发者选项，选择 Revoke USB debugging authorizations

- 一切顺利的话，就可以在 Inspect Devices 对话框看到连接的设备。

## 在开发机上调试 Android 设备上的内容

- 在 Inspect Devices 对话框左侧菜单选择要调试的设备。右侧就会出现 Android 设备的一些相关信息。
- 点击 inspect 就可以调试了

## 映射 Android 设备内容到开发机
DevTools 右上角启用 screencast

原文地址：

- https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging?hl=en
- [调试webview](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/webviews?hl=en)




















