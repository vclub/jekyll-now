---
layout: post
title: InstaMaterial concept (part 2) - Comments window transition
tags: [android, material design, ui, animations, layout]
---

这是如何实现 *[INSTAGRAM with Material Design]* 视频中的效果系列课程的一部分。今天我们将要实现的是feed和评论界切换以及相关的动画效果。（视频中9-13秒那部分）。我将忽略涉及到的button效果（波纹、发送完成的动画等，在下篇文章专门讨论），将重点放在comment Acitvity进入的效果上面。

<!-- more -->

这是今天这篇文章的最终效果 (包括 Android Lollipop 和 21之前的版本):

<iframe width="420" height="315" src="//www.youtube.com/embed/6C7ZA5cWdPE" frameborder="0" allowfullscreen></iframe>

<iframe width="420" height="315" src="//www.youtube.com/embed/b8OOaluag-w" frameborder="0" allowfullscreen></iframe>

#初始化设置

我们首先将一些不太重要的东西加入前面文章所创建的项目中 [previously created project]. 我们需要添加如下库:

* [Picasso] 一个图片异步加载库（用于评论列表中显示作者的头像）,
* 在`AndroidManifest.xml` 文件中添加评论的新Activity，并设置样式.

当然我们还要创建新activity的布局文件。除了底部的评论输入框之外基本上和上篇文章是一致的。我们再次用到了`Toolbar`，`RecyclerView`等。没有什么好值得讲的。

{% gist frogermcs/827dfb8077af625bf2d4 activity_comments.xml %}

![CommentsActivity layout preview](/images/3/comments_activity_preview.png "CommentsActivity layout preview")

下一步，创建评论列表item的布局：:

{% gist frogermcs/827dfb8077af625bf2d4 item_comment.xml %}

![Comments list item](/images/3/comments_list_item.png "Comments list item")

评论作者圆角头像部分的代码片段:

{% gist frogermcs/827dfb8077af625bf2d4 bg_comment_avatar.xml %}

最后一件事情是处理feed卡片底部评论按钮的的onClick事件，这个事件将打开显示当前照片评论的`CommentsActivity`。我们暂时将卡片的整个底部区域作为点击按钮，但是以后将会替换为真正的评论按钮。我们在RecyclerView adapter中添加onClick的listener，将为每个卡片的底部设置这个listener。

`FeedAdapter` class (只包含改动部分):

{% gist frogermcs/827dfb8077af625bf2d4 FeedAdapter.java %}

`MainActivity` class (只包含改动部分):

{% gist frogermcs/827dfb8077af625bf2d4 MainActivity.java %}

为了防止遗漏，我们将这一部分的代码提交在了这里 [onClick on item in RecyclerView adapter].

...上面就是初始化设置要做的了.

## 进入动画

先我们将创建进入的过度动画，根据概念视频上的显示，以下是我们需要实现的效果:

* Static Toolbar - 新的activity打开的时候Actionbar不能有过渡与切换效果（我们想让用户以为他们仍然在同一个窗口中）
* 评论列表界面要根据用户点击的位置展开（不管列表滚动到何处）
* 在展开效果结束之后，评论列表中的每一项要依次展示.

### Static Toolbar
这是本文最简单的部分。因为在MainActivity与CommentsActivity中`Toolbar`是非常相似的，所以我们只需要将Acitvity默认的切换效果屏蔽了就可以了，那样新的窗口不会有任何移动仅仅是绘制在上个窗口的上面。这样就实现了Toolbar的静态显示。下面是代码:

{% gist frogermcs/827dfb8077af625bf2d4 MainActivity_disable_transition.java %}

通过调用 `overridePendingTransition(0, 0);` 我们屏蔽了`MainActivity`退出效果和`CommentsActivity`的进入效果.

### 从被点击的地方展开 `CommentsActivity`

现在我们创建可以从任何点击的地方开始的扩展动画。这个动画分为:

* 展开背景
* 显示内容

在我们开始动画部分的代码之前，先将`CommentsActivity`设置成半透明。不然的话扩展动画将显示这默认的窗口背景之上，而不是`MainAcitvity`的view之上。这是因为每个activity的`windowBackground`都是定义在它所采用的主题中了的。如果我们想让Activity变半透明，我们需要修改样式:

{% gist frogermcs/827dfb8077af625bf2d4 styles.xml %}

下面是改变背景与不改变背景的区别:

![Expand animation with translucent background](/images/3/expand_translucent.gif "Expand animation with translucent background")
![Expand animation without translucent background](/images/3/expand_not_translucent.gif "Expand animation without translucent background")

现在我们可以开始制造展开的效果了。首先，我们需要得到动画的Y轴初始值。我们的例子中其实完全不需要知道点击的精确位置（动画很快，用户不会注意到几个像素的差异的），我使用被点击view的Y值来替代，并且将这个Y值传递给`CommentsActivity`:

{% gist frogermcs/827dfb8077af625bf2d4 MainAcitity_view_location.java %}

下一步，在 `CommentsActivity` 中实现background的展开动画。为了简便起见，我们使用Scale动画（因为在此刻还没有任何内容，因此没人知道到底是缩放开来的还是展开的），别忘了使用 `setPivotY()` 方法设置正确的初始位置.

{% gist frogermcs/827dfb8077af625bf2d4 CommentsActivity_enter_transition.java %}

通过上面的代码，当我们打开`CommentsActivity`时，动画只会开始一次，多亏了`onPreDrawListener`，我们才可以在view树完成测量并且分配空间,而绘制过程还没有开始的时候播放动画。

上面的代码中我们已经实现了展开背景与显示内容的动画，下面是运行的效果:

![Expand animation](/images/3/expand_animation.gif "Expand animation")

是不是感觉还是少了点什么东西?

还需要准备评论列表中每个评论项的动画。很简单，但是需要注意几件重要的事情:

* 每个item的动画需要有一定延时。否则所有的动画将在瞬间结束用户只能感受到一个动画。
* Adapter需要有锁定动画的功能，因为在用户滚动列表的时候动画是不需要的。
* 同样的我们还要让每个单独的item能锁定与解锁动画（比如添加一个评论）

目前CommentsAdapter 的代码如下:

{% gist frogermcs/827dfb8077af625bf2d4 CommentsAdapter.java %}

为了展示头像我们使用带 [CircleTransformation] 的 [Picasso] 库 . 我们利用 `RecyclerView` 的 `notifyItemInserted()` 方法实现了添加一个item的动画效果，其余的代码都很简单.

下面是在 `CommentsActivity` 中使用的代码:

{% gist frogermcs/827dfb8077af625bf2d4 CommentsActivity_items_animations.java %}

Items 的动画在用户滚动 `RecyclerView` 的时候被锁定了 (当用户添加新条目时，它们会临时解锁).

到这里. `CommentsActivity` 的进入动画就全部完成了.

## 退出动画
后一件事是实现退出动画，没有非常特别的技巧，我们只需创建一个transition 动画来滑出activity就可以了，记住Toolbar必须是静止的，因此我们再次使用overridePendingTransition(0, 0);并且播放内容部分的动画:

{% gist frogermcs/827dfb8077af625bf2d4 CommentsActivity_exit_transition.java %}

以上就是我们实现概念app的第二阶段的所有内容。下一篇我们将讨论这章遗漏的内容（按钮的动画效果）.

##Source code
完整的代码在Github [repository].

中文参考[http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0206/2422.html]

*Author: [Miroslaw Stanek]*

[INSTAGRAM with Material Design concept]:https://www.youtube.com/watch?v=ojwdmgmdR_Q
[previously created project]:https://github.com/frogermcs/InstaMaterial
[Picasso]:http://square.github.io/picasso/
[onClick on item in RecyclerView adapter]:https://github.com/frogermcs/InstaMaterial/commit/ec3d47bd546f4bdcb7ba1a2a5afe58112972ea0a
[CircleTransformation]:https://gist.github.com/julianshen/5829333
[Miroslaw Stanek]:http://about.me/froger_mcs
[repository]:https://github.com/frogermcs/InstaMaterial
