---
layout: post
title: 实现Instagram的Material Design概念设计
tags: [android, material design, ui, animations, layout]
---

几个月前（这篇文章的日期是2014 年11月10日），google发布了app和web应用的Material Design设计准则之后，设计师Emmanuel Pacamalan在youtube上发布了一则概念视频，演示了Instagram如果做成Material风格会是什么样子：

<!-- more -->

<iframe width="560" height="315" src="//www.youtube.com/embed/ojwdmgmdR_Q" frameborder="0" allowfullscreen></iframe>

这仅仅是停留在图像上的设计，是美好的愿景，估计很多人都会问，能否使用相对简单的办法将它实现出来呢？答案是：yes，不仅仅能实现，而且无须要求在 Lillipop版本，实际上几年前4.0发布之后我们就可以实现这些效果了。ps 读到这里我们应该反思这几年开发者是不是都吃屎去了。

鉴于这个原因，我决定开始撰写一个新的课题-如何将 *[INSTAGRAM with Material Design]* 视频中的效果转变成现实。当然，我们并不是真的要做一个Instagram应用，只是将界面做出来而已，并且尽量减少一些不必要的细节。

#开始
本文将要实现的是视频中前7秒钟的效果。我觉得对于第一次尝试来说已经足够了。我想要提醒诸位的是，里面的实现方法不仅仅是能实现，也是我个人最喜欢的实现方式。还有，我不是一个美工，因此项目中的所有图片是直接从网上公开的渠道获取的。（主要是从[resources page]).

好了，下面是最终效果的两组截图和视频（很短的视频，就是那7秒钟的效果，可以在上面的视频中看到，这里因为没法直接引用youtube的视频就略了）（分别从Android 4 和5上获得的）：

![Android Lollipop screenshot](/images/2/screen-21.png "Android Lollipop screenshot")
![Android pre-21 screenshot](/images/2/screen-pre21.png "Android pre-21 screenshot")

<iframe width="420" height="315" src="//www.youtube.com/embed/rTucTiIlQDA" frameborder="0" allowfullscreen></iframe>

<iframe width="420" height="315" src="//www.youtube.com/embed/fYhpc1LddHE" frameborder="0" allowfullscreen></iframe>

# 准备
在我们的项目中，将使用一些热门的android开发工具和库。并不是所有这些东西本篇文章都会用到，我只是将它们准备好以备不时之需。

## 初始化项目
首先我们需要创建一个新的android项目。我使用的是Android Studio和gradle的build方式。最低版本的sdk是15（即Android 4.0.4）。

然后我们将添加一些依赖。没什么好讲的，下面是build.gradle以及app/build.gradle文件的代码：

{% gist frogermcs/61b885eb2e44e0d2d61f build.gradle %}

{% gist frogermcs/61b885eb2e44e0d2d61f appbuild.gradle %}

简而言之，我们有如下工具：

* 一些兼容包（CardView, RecyclerView, Palette, AppCompat）-我喜欢使用最新的控件。当然你完全可以使用ListView Actionbar甚至View/FrameView来替代，但是为什么要这么折腾？ 😉
* [ButterKnife] - view注入工具简化我们的代码。（比方说不再需要写findViewById()来引用view，以及一些更强大的功能）.
* [Rebound] - 我们目前还没有用到，但是我以后肯定会用它。这个facebook开发的动画库可以让你的动画效果看起来更自然。
* [Timber] and [Hugo] - 对这个项目而言并不是必须，我仅仅是用他们打印log。

### 图片资源
本项目中将使用到一些Material Design的图标资源。App 图标来自于[INSTAGRAM with Material Design] 视频，这里 [complete bunch of images] 是项目的全套资源.

### v
我们从定义app的默认样式开始。同时为Android 4和5定义Material Desing样式的最简单的方式是直接继承Theme.AppCompat.NoActionBar 或者 Theme.AppCompat.Light.NoActionBar主题。为什么是NoActionBar？因为新的sdk中为我们提供了实现Actionbar功能的新模式。本例中我们将使用Toolbar控件，基于这个原因-Toolbar是ActionBar更好更灵活的解决方案。我们不会深入讲解这个问题，但你可以去阅读android开发者博客 [AppCompat v21].

根据概念视频中的效果，我们在AppTheme中定义了三个基本颜色 `AppTheme`:

{% gist frogermcs/61b885eb2e44e0d2d61f styles.xml %}

{% gist frogermcs/61b885eb2e44e0d2d61f colors1.xml %}

关于这三个颜色的意义，你可以在这里找到[Material Theme Color Palette documentation].

###布局
项目目前主要使用了3个主要的布局元素:

* **Toolbar** - 包含导航图标和applogo的顶部bar
* **RecyclerView** - 用于显示feed
* **Floating Action Button** - 一个实现了Material Design中 [action button pattern] 的ImageButton.

在开始实现布局之前，我们先在 `res/values/dimens.xml`文件中定义一些默认值:

{% gist frogermcs/61b885eb2e44e0d2d61f dimens.xml %}

这些值的大小是基于Material Design设计准则中的介绍。

现在我们来实现`MainActivity`中的layout:

{% gist frogermcs/61b885eb2e44e0d2d61f activity_main.xml %}

以上代码的解释:

* 关于`Toolbar` 重要的特征是他现在是activity layout的一部分，而且继承自ViewGroup，因此我们可以在里面放一些UI元素（它们将利用剩余空间）。本例中，它被用来放置logo图片。同时，因为Toolbar是比Actionbar更灵活的控件，我们可以自定义更多的东西，比如设置背景颜色为colorPrimary（否则`Toolbar`将是透明的）。
* `RecyclerView` 虽然在xml中用起来非常简单，但是如果java代码中没有设置正确，app是不能启动的，会报`java.lang.NullPointerException` (it's connected with not configured LayoutAdapter which is resposible for arranging items inside RecyclerView).
* Elevation 属性不兼容API 21以前的版本。所以如果我们想做到 **Floating Action Button** 的效果需要在Lollipop和Lollipop之前的设备上使用不同的`background`.

#### Floating Action Button
为了简化FAB的使用，我们将用对Lollipop以及Lollipop之前的设备使用不同的样式：

 * **FAB for Android v21**:

 ![FAB Android 21](/images/2/fab-21.png "FAB Android 21")

 * **FAB for Android pre-21**

 ![FAB Android pre-21](/images/2/fab-pre21.png "FAB Android pre-21")

 我们需要创建两个不同的xml文件来设置button的background:
  `/res/drawable-v21/btn_fab_default.xml` 为Android Lollipop版本设备和 `/res/drawable/btn_fab_default.xml` 为Lollipop之前的版本:

{% gist frogermcs/61b885eb2e44e0d2d61f btn_fab_default2.xml %}

{% gist frogermcs/61b885eb2e44e0d2d61f btn_fab_default1.xml %}

我们还要在 `res/values/colors.xml` 文件中添加两个颜色:
{% highlight xml %}
<color name="btn_default_light_normal">#00000000</color>
<color name="btn_default_light_pressed">#40ffffff</color>
{% endhighlight %}

可以看到在 21之前的设备商制造阴影比较复杂。不幸的是在xml中达到真实的阴影效果没有渐变方法。其他的办法是使用图片的方式，或者通过java代码实现 (参见 [creating fab shadow]).

#### Toolbar
现在我们来完成`Toolbar`。我们已经有了background和应用的logo，现在还剩下navigation以及menu菜单图标了。.

非常不幸的是，在xml中`app:navigationIcon=""` 是不起作用的，而`android:navigationIcon=""` 又只能在Lollipop上有用，所以只能使用代码的方式了:

{% highlight java %}
toolbar.setNavigationIcon(R.drawable.ic_menu_white);
{% endhighlight %}

至于menu图标我们使用标准的定义方式就好了 `res/menu/menu_main.xml`:

{% gist frogermcs/61b885eb2e44e0d2d61f menu_main.xml %}

在activity中inflated这个menu:
{% highlight java %}
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main, menu);
    return true;
}
{% endhighlight %}

本应运行的很好，但是正如我在twitter上提到的，Toolbar onClick  selectors有不协调的情况:

 <iframe width="560" height="510" src="//twitframe.com/show?url=https://twitter.com/froger_mcs/status/530995707080359936" frameborder="0" allowfullscreen></iframe>

为了解决这个问题，需要做更多的工作，首先为menu item创建一个自定义的view (`res/layout/menu_item_view.xml`):

{% gist frogermcs/61b885eb2e44e0d2d61f menu_item_view.xml %}

然后为Lollipop和Lollipop之前的设备分别创建onClick的selector，在Lollipop上有ripple效果:

{% gist frogermcs/61b885eb2e44e0d2d61f btn_default_light2.xml %}

{% gist frogermcs/61b885eb2e44e0d2d61f btn_default_light1.xml %}

现在，工程中的所有的color应该是这样子了:

{% gist frogermcs/61b885eb2e44e0d2d61f colors.xml %}

最后我们应该将custom view放到menu item中，在`onCreateOptionsMenu()`中:

{% highlight java %}
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main, menu);
    inboxMenuItem = menu.findItem(R.id.action_inbox);
    inboxMenuItem.setActionView(R.layout.menu_item_view);
    return true;
}
{% endhighlight %}

以上就是toolbar的所有东西。并且onClick的按下效果也达到了预期的效果:

![Toolbar menu item onClick](/images/2/toolbar.png "Toolbar menu item onClick")


#### Feed
Last thing we should implement is feed, built on `RecyclerView`. Right now we have to setup two things: layout manager (RecyclerView has to know how to arrange items) and adapter (to provide items).
最后我们要实现的是feed，基于`RecycleView`.现在我们要设置两个东西：layout manager 和 adapter。

第一件事比较简单－实现一个简单的ListView效果，就用`LinearLayoutManager`就可以了。然后要做些不能偷懒的code.

我们先从Item的布局开始 (`res/layout/item_feed.xml`):

{% gist frogermcs/61b885eb2e44e0d2d61f item_feed.xml %}

在上面的代码中:

* `CardView` - 给列表项边添加圆角边和阴影 (支持 Android 21 and pre-21).
* `ImageViews` 列表项显示图片 (SquaredImageView 用来显示圆形图片).

同样`FeedAdapter` 也比较简单:
{% gist frogermcs/61b885eb2e44e0d2d61f FeedAdapter.java %}

没啥特殊要讲的.

把所有的代码合到一起，然后在 `MainActivity` 类中添加如下方法:
{% highlight java %}
private void setupFeed() {
    LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
    rvFeed.setLayoutManager(linearLayoutManager);
    feedAdapter = new FeedAdapter(this);
    rvFeed.setAdapter(feedAdapter);
}
{% endhighlight %}

下面是完整的 MainActivity 代码:
{% gist frogermcs/61b885eb2e44e0d2d61f MainActivity1.java %}

为了避免遗漏，这里是到目前为止的所有代码 [commit for whole project].

这里的运行结果:

* **Android Lollipop**

![Android Lollipop screenshot](/images/2/screen-21.png "Android Lollipop screenshot")

* **Android pre-21**

![Android pre-21 screenshot](/images/2/screen-pre21.png "Android pre-21 screenshot")

### Animations

最后一件也是最重要的事情就是进入时的动画效果，再浏览一遍概念视频，可以发现在main Activity启动的时候有如下动画，分成两步:

* 显示Toolbar以及其里面的元素
* 在Toolbar动画完成之后显示feed和floating action button.

Toolbar 中元素的动画表现为在较短的时间内一个接一个的进入。实现这个效果的主要问题在于navigation icon的动画，navigation icon是唯一一个不能使用动画的，其他的都好办.

#### Toolbar animation
首先我们只是需要在activity启动的时候才播放动画（在旋转屏幕的时候不播放）. 还要知道menu的动画过程是不能在`onCreate()``中去实现的 (我们在 `onCreateOptionsMenu()`中实现).

创建一个布尔类型的变量 `pendingIntroAnimation` 在 `MainActivity`中. 在`onCreate()` 方法中初始化:
{% highlight java %}
//...
if (savedInstanceState == null) {
    pendingIntroAnimation = true;
}
{% endhighlight %}

并在 `onCreateOptionsMenu()`方法中使用:
{% highlight java %}
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main, menu);
    inboxMenuItem = menu.findItem(R.id.action_inbox);
    inboxMenuItem.setActionView(R.layout.menu_item_view);
    if (pendingIntroAnimation) {
        pendingIntroAnimation = false;
        startIntroAnimation();
    }
    return true;
}
{% endhighlight %}

这样当我们运行我们的应用时，`startIntroAnimation()` 只会被调用一次.

现在准备Toolbar中的各元素动画了。也同样简单。要记住的是，一般来说，动画是由两个步骤组成：

* preparation准备 - 设置动画元素的初始状态. 如果我们要展示动画，首先确保他们先是隐藏的.
* animation动画 - 这一步是设置View的最后状态或位置.

现在来实现Toolbar动画的两个步骤:

{% gist frogermcs/61b885eb2e44e0d2d61f ToolbarAnimation %}

在上面的代码中:

* 首先我们将所有的元素都通过移动到屏幕之外隐藏起来（这一步我们将FAB也隐藏了）。
* 让Toolbar元素一个接一个的开始动画
* 当动画完成，调用了startContentAnimation()开始content的动画（FAB和feed卡片的动画）

简单, 是吧?

#### Content 动画
在这一步中我们将让FAB和feed卡片动起来。

FAB的动画很简单，跟上面的方法类似，但是feed卡片稍微复杂些。

* `startContentAnimation()` 方法:

{% gist frogermcs/61b885eb2e44e0d2d61f FABAnimation %}

* `FeedAdapter` 包含动画和feed项

{% gist frogermcs/61b885eb2e44e0d2d61f FeedAdapter.java %}

本篇文章就结束了.

为了避免遗漏, 这里是代码 [commit for our project with implemented animations].

##Source code
完整的代码在Github [repository].

英文原文：[Instagram with Material Design concept is getting real]

中文翻译出处：［http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0204/2415.html］

*Author: [Miroslaw Stanek]*

[INSTAGRAM with Material Design]:https://www.youtube.com/watch?v=ojwdmgmdR_Q
[resources page]:http://www.google.com/design/spec/resources/sticker-sheets-icons.html
[Butterknife]:http://jakewharton.github.io/butterknife/
[Rebound]:http://facebook.github.io/rebound/
[Timber]:https://github.com/JakeWharton/timber
[Hugo]:https://github.com/JakeWharton/hugo
[complete bunch of images]:https://github.com/frogermcs/frogermcs.github.io/raw/master/files/2/resources.zip
[AppCompat v21]:http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html
[Material Theme Color Palette documentation]:http://developer.android.com/training/material/theme.html#ColorPalette
[action button pattern]:http://www.google.com/design/spec/components/buttons.html#buttons-flat-raised-buttons
[creating fab shadow]:http://stackoverflow.com/questions/24480425/android-l-fab-button-shadow
[commit for whole project]:https://github.com/frogermcs/InstaMaterial/commit/1095d5f199ceb5c04bb2433c8865dcea2cba6f2e
[commit for our project with implemented animations]:https://github.com/frogermcs/InstaMaterial/commit/4e838861c5f858711b1072777cae2325ce12ee21
[repository]:https://github.com/frogermcs/InstaMaterial
[Miroslaw Stanek]:http://about.me/froger_mcs
[Instagram with Material Design concept is getting real]:http://frogermcs.github.io/Instagram-with-Material-Design-concept-is-getting-real
