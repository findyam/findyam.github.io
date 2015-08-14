---
layout: post
title: "iOS生成静态库"
date: 2015-08-14 10:55:21 +0800
comments: true
categories: 
---

这篇主要是复习一下iOS生成静态库。

为什么写这个主要是因为项目中加入了一个只支持真机的静态库文件，导致模拟器不能用，前阶段很忙，把这个问题放到一边了，今天有时间就把这个问题搞了一下。

解决这个问题思路很简单，就是自己用那个静态库的头文件，自己实现一遍所有接口 *(其实就是每个接口里打个log)*，然后重新生成一个模拟器包，最后真机包和模拟器包合并生成新的包导入工程。

平时开发的时候会遇到蛮多的情况需要打包静态库，下面就简单回顾下静态库如何打包。

------



静态库也就是我们平时看到的很多后缀为*.a*的文件，打包的目的就不多说了，直接开始了。

首先我们需要建一个StaticLibrary的工程 

{% img /images/iOSStaticLibary/create_project.png %}

建好工程之后，就往里搞代码，这里就写的简单点

{% img /images/iOSStaticLibary/code.png %}

然后就开始运行了，这里有一点需要注意，在这里编译的话，需要选择是编译simulator还是device，编译的结果只能是模拟器或真机使用的。

编译成功后会看到

{% img /images/iOSStaticLibary/build_completed.png %}

*在这里需要说一下，只有真机编译的时候才会看到Products文件夹下的.a文件变黑如果需要模拟器的包，可以先编译模拟器版的，再编译真机，这样可以直接右键点击.a文件show in finder 跳转到生成的文件夹目录*

编译好之后右键点击Products 文件夹下的 .a 文件，show in finder 就跳到相应的目录了

{% img /images/iOSStaticLibary/products_dir.png %}

这里需要注意一点就是编译时要记得选择是release 还是debug。

好了，两种文件都生成了，接下来就是合并文件了。

``` bash
  lipo -create Release-iphoneos/libFYStaticLibraryDemo.a Release-iphonesimulator/libFYStaticLibraryDemo.a -output libFyStaticLibraryDemo.a
```



terminal 里执行一下，搞定。

------

好吧，这样搞实在是麻烦，每次修改完代码，要做这么多事情，其实还是有更简单的方法

{% img /images/iOSStaticLibary/new_target.png %}

创建一个新的target

{% img /images/iOSStaticLibary/aggregate.png %}

project 选中当前的project

{% img /images/iOSStaticLibary/name_target.png %}

然后选中新建的target — Build Phases ，点击加号 New Run Script Phase

{% img /images/iOSStaticLibary/add_run_script.png %}

然后在RunScript 中添加如下代码

``` bash
# define output folder environment variable
UNIVERSAL_OUTPUTFOLDER=${BUILD_DIR}/${CONFIGURATION}-universal

# Step 1. Build Device and Simulator versions
xcodebuild -target FYStaticLibraryDemo ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphoneos  BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}"
xcodebuild -target FYStaticLibraryDemo -configuration ${CONFIGURATION} -sdk iphonesimulator -arch i386 BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}"

# make sure the output directory exists
mkdir -p "${UNIVERSAL_OUTPUTFOLDER}"

# Step 2. Create universal binary file using lipo
lipo -create -output "${UNIVERSAL_OUTPUTFOLDER}/lib${PROJECT_NAME}.a" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/lib${PROJECT_NAME}.a" "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/lib${PROJECT_NAME}.a"

# Last touch. copy the header files. Just for convenience
cp -R "${BUILD_DIR}/${CONFIGURATION}-iphoneos/include" "${UNIVERSAL_OUTPUTFOLDER}/"
```

记得把target 改掉。

好了，一切准备就绪，然后选中新建的aggregate target，点击run，然后在products 文件夹里就会看到最终版本了。

{% img /images/iOSStaticLibary/final_result.png %}



静态库打包就简单写到这里，写的不是很详细，以后遇到问题，还是会在这里更新。

------

参考内容：

[http://www.cocoachina.com/industry/20131204/7468.html](http://www.cocoachina.com/industry/20131204/7468.html)

[http://www.raywenderlich.com/41377/creating-a-static-library-in-ios-tutorial](http://www.raywenderlich.com/41377/creating-a-static-library-in-ios-tutorial)



























###  