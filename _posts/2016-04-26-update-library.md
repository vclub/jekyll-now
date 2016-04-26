---
layout: post
title: 最近一些第三方组件更新了
tags: [gitlab]
---

## 1. ButterKnife升级到8.0了。

```
buildscript {
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
  }
}

apply plugin: 'com.neenbedankt.android-apt'

dependencies {
  compile 'com.jakewharton:butterknife:8.0.0'
  apt 'com.jakewharton:butterknife-compiler:8.0.0'
}
```
应该是不用配置Progruad了。【我还没有验证，呵呵】

之前绑定是使用`@Bind`，现在换成`@BindView`了，要自己指定类型了
```
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```

Fragment的取消绑定的规则换了
```
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;
  private Unbinder unbinder;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    unbinder = ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    unbinder.unbind();
  }
}
```

## 2. Fresco 可以更新到0.10.0版本了，支持Okhttp3.0了。

```
dependencies {
  // your project's other dependencies
  compile "com.facebook.fresco:imagepipeline-okhttp3:0.10.0+"
}
```

初始化代码有点小小的变化：
```
Context context;
OkHttpClient okHttpClient; // build on your own
ImagePipelineConfig config = OkHttpImagePipelineConfigFactory
    .newBuilder(context, okHttpClient)
    . // other setters
    . // setNetworkFetcher is already called for you
    .build();
Fresco.initialize(context, config);
```


## 3. 最近发现一个插件，就不用配置rxjava的progruad了, 配置如下：

```
// RxJava itself
compile 'io.reactivex:rxjava:1.1.3'

// And ProGuard rules for RxJava!
compile 'com.artemzin.rxjava:proguard-rules:1.1.3.0'
```
