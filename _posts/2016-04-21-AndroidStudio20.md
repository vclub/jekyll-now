---
layout: post
title: Android Studio 2.0 Instant Run 注意事项
tags: [android studio 2.0 Instant Run]
---

* Instant Run 只支持 `debug` 模式的编译，需要使用 Android Plugin for Gradle version 2.0.0 或更高，并且是API 21以上的设备。

下面是网上有人提供的一种建议，通过开发分支，将`minSdkVersion`设置成21.

```
android {
productFlavors {
    // Define separate dev and prod product flavors.
    dev {
        // dev utilizes minSDKVersion = 21 to allow the Android gradle plugin
        // to pre-dex each module and produce an APK that can be tested on
        // Android Lollipop without time consuming dex merging processes.
        minSdkVersion 21
    }
    prod {
        // The actual minSdkVersion for the application.
        minSdkVersion 14
    }
}
      ...
buildTypes {
    release {
        runProguard true
        proguardFiles getDefaultProguardFile('proguard-android.txt'),
                                             'proguard-rules.pro'
    }
}

}
dependencies {
  compile 'com.android.support:multidex:1.0.0'
}
```

* 通过设置 DEX配置 改进编译时间

```
android {
  ...
  dexOptions {
    maxProcessCount 4 // this is the default value
    javaMaxHeapSize "2g"
  }
}
```


* [将你的项目目录从Windows Defender的检测目录中移除]

[将你的项目目录从Windows Defender的检测目录中移除]:http://answers.microsoft.com/en-us/protect/wiki/protect_defender-protect_scanning/how-to-exclude-a-filefolder-from-windows-defender/f32ee18f-a012-4f02-8611-0737570e8eee

* 在debug编译时关闭 Crashlytics

* Instant Run有局限性

InstantRun主要是被设计用来加速构建和部署的。 所以有时候会影响到你的app的行为和兼容性，如果遇到问题，请提交Bug给AS团队。

* 部署到多个设备

如果想同时部署到多个设备，需要关闭InstantRun

* multidex

如果你的项目设置了 multiDexEnabled true 并且 minSkdVersion 20 以下， 你部署的目标设备低于 Android 4.4，
Android Studio 会关闭 Instant Run

如果 `minSdkVersion` 设置了21以上， Instant Run 会自动为你设置multidex，
因为Instant Run 只能工作在debug编译版本，所以你可能需要自己配置你的`release`的编译版本

* 运行instrumented tests和性能分析

在运行Instrumented测试时，不会开启InstantRun

当进行分析你的app时，你应该关闭InstantRun。 因为InstantRun会对性能有一点影响，当使用热交换时影响的更多。
另外热交换这种方式，也会使堆栈的跟踪更加复杂化。


* 使用第三方插件

当使用 Instant Run 时， AS会临时关闭 Java Code Coverage Library （JaCoCo）和 Proguard, 但不会影响你的Release编译

### 关闭 Instant Run

* 打开 Settings 或 Preferences 对话框.
* 找到 Build, Execution, Deployment > Instant Run.
* 取消 Enable Instant Run的选择.
