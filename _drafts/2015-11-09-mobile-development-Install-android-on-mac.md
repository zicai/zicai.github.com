---
layout: post
category : lessons
title: "Mac上搭建Android开发环境"
tagline: "Supporting tagline"
tags : [meteor]
---


## 与之前版本的不同
从 Meteor 1.2 开始，移除了内置的 Android tools。需要你自己安装 Android SDK。这样的改变使得保持开发工具集的更新变得容易，也避免了一些难以调试的bug。 `meteor install-sdk` 命令不再用来下载安装 Android tools (该命令已被弃用)。

## Android Studio
搭建 Android 开发环境最简单的方式就是安装 [Android Studio](http://developer.android.com/sdk/index.html)，它提供了安装向导，第一次启动的时候会安装 Android SDK Tools，并创建一个默认的虚拟设备。

如果你不打算使用 Android Studio，也可以单独安装。只需确保安装了最新版的 Android SDK Tools ，SDK Platform API 22，并创建一个虚拟设备。

## 关键步骤

- 安装 Java Development Kit(JDK)
- 安装 Android Studio 并完成初始化向导
- 安装 Android SDK Platform API 22
- 可选：将 ANDROID_HOME 和工具目录添加到 PATH

#### 安装 Java Development Kit(JDK)

1. 打开 [Oracle Java website](http://www.oracle.com/technetwork/java/javase/downloads/index.html) ,选择 Java Platform（JDK）

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/1-pick-jdk.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/1-pick-jdk.png)

2. 勾选 accept the license agreement，选择 Mac OS X 镜像文件（jdk-8u**-macosx-x64.dmg）

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/2-download-jdk.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/2-download-jdk.png) 

3. 打开下载的文件

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/3-open-dmg.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/3-open-dmg.png)

4. 启动 Java 安装包

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/4-open-pkg.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/4-open-pkg.png)

5. 点击 Continue 然后 Install

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/5-continue.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/5-continue.png)

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/5-continue.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/5-continue.png)

6. 输入密码，回车

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/7-password.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/7-password.png)

7. 完成安装

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/8-close.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-jdk/8-close.png)

### 安装 Android Studio 并完成初始化向导

1. 打开 [Android Studio website](http://developer.android.com/sdk/index.html)，点击 'Download Android Studio for Mac'
2. 同意 license agreement，点击 'Download Android Studio for Mac'
3. 打开下载的文件，并安装

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-install.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-install.png)

4. 启动 Android Studio 。选择 standard installation，等待向导下载并安装 Android 开发环境所需的组件。

#### 安装 Android SDK Platform API 22

需要注意的是，Android Studio 安装向导只会下载最新的 SDK Platform(API 23)，然而 Cordova 依赖的是 API 22。也就是说你必须手动安装 Android SDK Platform API 22。

1. 点击 'Configure'

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-configure.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-configure.png)

2. 选择 'SDK Manager'

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-launch-sdk-manager.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-studio-launch-sdk-manager.png)

3. Android SDK Manager 被启动，展开 'Android 5.1.1 (API 22)' 文件夹，勾选 'SDK Platform' ，点击安装

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-sdk-platform-22-select.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-sdk-platform-22-select.png)

4. 同意条款，等待安装

    ![https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-sdk-platform-22--accept-license.png](https://github.com/meteor/meteor/wiki/images/screenflows/mac-install-android-studio/android-sdk-platform-22--accept-license.png)

### 可选：将 ANDROID_HOME 和工具目录添加到 PATH

Cordova 会自动在多个位置检测 Android SDK 的安装，包括 Android Studio 默认使用的路径。所以这一步并不是必须的，但是如果你打算在命令行使用 Android tools,还是建议你添加 PATH。

- 将环境变量 `ANDROID_HOME` 设置为 Android SDK 的目录。如果你使用的是 Android Studio 安装向导，默认目录应该是 `~/Library/Android/sdk`
- 将 `$ANDROID_HOME/tools` 和 `$ANDROID_HOME/platform-tools` 添加到 PATH






