Title: 详解android:layout_weight
Date: 2014-03-28 17:51
Category: Technology
Tags: Android
Slug: android-layout-weight
Author: xymelon

###android:layout_weight

`android：weight`是线性布局`LinearLayout`中非常重要的概念，在创造兼容性APP中有不可替代的功劳。它主要作用为分配剩余空间，剩余空间是在线性布局中把子组件所占空间分配完毕后所剩余的部分。每个子组件占剩余空间的比例等于自身所设置参数除以所有子组件所设置参数之和。

<!-- PELICAN_END_SUMMARY -->

对于线性布局有`horizontal`和`vertical`两个方向，不过剩余空间计算方法都是一样的，以下举例都为` android:orientation="horizontal"`。

剩余空间计算公式：**剩余空间 = 父组件宽度 - 各组件所设置宽度或所占宽度**

**子组件所占空间 = 所设置宽度或所占宽度 + 剩余空间 * 子组件所占比例**

剩余空间和子组件所占空间与`android:layout_width`属性设置参数（`wrap_content`和`match_parent`）有很大关系，以下分别举例。

###设置参数`android:layout_width:"wrap_content"`

首先看看如下布局xml：

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:paddingTop="50dp" >
    
    <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="1"
    android:background="#FFFF00"
    android:text="1" />
    
    <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="2"
    android:background="#FF0000"
    android:text="2" />
    
    <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="3"
    android:background="#000000"
    android:text="3" />
    
    </LinearLayout>


布局效果图1如下：

![布局效果1](/images/layout_weight1.png)

此处各子组件`android:layout_weight`之和为6，TextView1、TextView2、TextView3各占剩余空间比例为1/6、2/6、3/6。各子组件所设置`android:layout_width`属性为`wrap_content`，即依内容而定，此处为各自文本字符串1、2、3的宽度；父组件宽度属性为`android:layout_width="match_parent"`，即撑满整个屏幕宽度。

> 剩余空间 = 屏幕宽度 - 字符串1宽度 - 字符串2宽度 - 字符串3宽度
> 
> 子组件1所占空间 = 字符串1宽度 + 1/6 * 剩余空间 
> 
> 子组件2所占空间 = 字符串2宽度 + 2/6 * 剩余空间
> 
> 字组件3所占空间 = 字符串3宽度 + 3/6 * 剩余空间

因字符串1、2、3宽度比较小，粗略估计为0。那么此处各子组件所占组件比例则近似于1：2：3，符合效果图1。

###设置参数`android:layout_width:"match_parent"`

同样布局文件xml如下，和上文xml文件唯一不同的地方在于各子组件所设置宽度属性改为了`match_parent`:

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:paddingTop="50dp" >
    
    <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_weight="1"
    android:background="#FFFF00"
    android:text="1" />
    
    <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_weight="2"
    android:background="#FF0000"
    android:text="2" />
    
    <TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_weight="3"
    android:background="#000000"
    android:text="3" />
    
    </LinearLayout>


效果图2如下：

![布局效果2](/images/layout_weight2.png)

看到图片你可能非常惊讶，怎么只显示1、2，而3没显示出来。不着急，根据计算公式，你就会知道结果。

同样此处各子组件`android:layout_weight`之和为6，TextView1、TextView2、TextView3各占剩余空间比例为1/6、2/6、3/6。各子组件和父组件宽度属性都为`android:layout_width="match_parent"`，即都撑满整个屏幕宽度。

> 剩余空间 = 屏幕宽度 - TextView1所设置宽度（**即屏幕宽度**） - TextView2所设置宽度（即屏幕宽度） - TextView3所设置宽度（即屏幕宽度） = -2个屏幕宽度（`对的，没错。此处剩余空间为负`）
> 
> 那么组件1所占空间 = TextView1所设置宽度（**即屏幕宽度**） + 1/6 * （-2个屏幕宽度） = 2/3个屏幕宽度
> 
> 组件2所占空间 = TextView2所设置宽度（即屏幕宽度） + 2/6 * （-2个屏幕宽度） = 1/3个屏幕宽度
> 
> 组件3所占空间 = TextView3所设置宽度（即屏幕宽度） + 3/6 * （-2个屏幕宽度） = 0

根据计算，我们可以看到组件3所占空间为0，且组件1和组件2所占空间比为2：1，和效果图一样。

那么如果是属性`wrap_content`和`match_parent`混合使用的话，同样可应用公式算出，本文不在详述。



