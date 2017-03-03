---
title: 共享元素转场动画Part1————Activity to Activity
---
Share Element Transition（共享元素变换）这一概念是在android 5.0 Material Design中提出的新的页面转场的方式。
那么本文将教你一步一步的做出炫酷的MD Style的转场动画。


## 什么是共享元素变换？

元素共享变换的定义：共享的View元素从一个Activity/Fragment到另一个Activity/Fragment的切换中是如何变化的；
在Google Play,Google Music 等众多Google嫡系APP中就得到了很多的运用。例如下图是Google Music的变化效果：

![Google Music](http://om6hh53na.bkt.clouddn.com/google_music.gif)

上图中的Activity A 中Grid列表的CardView中ImageView和Acitivty B 的ImageView拥有共享的元素图片,于是就形成了
无缝的动画切换效果。

## 开始准备工作

首先，我们要开启windowContentTransitions.
如果你的targetSdk < 21，那么你需要在你的res文件夹下创建一个value-v21的资源文件夹，添加以下一行代码
```xml
 <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        //add this line to open transitions
        <item name="android:windowContentTransitions">true</item>
    </style>
```

## Activity to Activity

然后，我们搭建两个Activity ActivityA 和ActivityB,如下图 ActivityA中有一个图片ImageView和一个按钮Button，ActivityB中有一个大图ImageView和一个描述的TextView。

![ActivityA](http://om6hh53na.bkt.clouddn.com/activity_a.png)
![ActivityB](http://om6hh53na.bkt.clouddn.com/activity_b.png)

ActivityA layout 布局文件如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="io.github.hexiangyuan.sharedelementtransitionsdemo.ActivityA">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="148dp"
        android:layout_height="148dp"
        android:scaleType="centerCrop"
        android:transitionName="@string/transitions_name"
        android:src="@drawable/image" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_below="@+id/imageView"
        android:onClick="imageClick"
        android:text="Click Me" />
</RelativeLayout>
```

特别注意的是`android:transitionName="@string/transitions_name`这一个属性，需要加在共享元素的View下，细心的朋友会发现Android Studio会变黄的tips提醒

> This check finds attributes set in XML files that were introduced in a version newer than the oldest version targeted by your application (with the minSdkVersion attribute). 
> This is not an error;
> the application will simply ignore the attribute. However, if the attribute is important to the appearance of functionality of your application, you should consider finding an alternative way to achieve the same result with only available attributes, and then you can optionally create a copy of > > the layout in a layout-vNN folder which will be used on API NN or higher where you can take advantage of the newer attribute.

意思是：这不是个错误，当你的target < 21时，你会收到lint的警告，你可以忽略或者设置不提醒。

AcitivityA java代码文件如下：

```java
public class ActivityA extends AppCompatActivity {

    private ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.acitivit_a);
        imageView = (ImageView) findViewById(R.id.imageView);
    }

    public void imageClick(View v) {
        Intent intent = new Intent(this, SimpleActivityB.class);
        ActivityOptionsCompat optionsCompat = ActivityOptionsCompat.makeSceneTransitionAnimation(
                this, imageView,
                getString(R.string.transitions_name)
        );
        startActivity(intent, optionsCompat.toBundle());
    }
}
```

需要留意的是

```java
Intent intent = new Intent(this, SimpleActivityB.class);
        ActivityOptionsCompat optionsCompat = ActivityOptionsCompat.makeSceneTransitionAnimation(
                this, imageView,
                getString(R.string.transitions_name)
        );
        startActivity(intent, optionsCompat.toBundle());
```

通过这个`ActivityOptionsCompat.makeSceneTransitionAnimation`方法，传入（context,view,transitionName)这些参数，然后跳转时候会将optiont通过bundle的方式传递过去；

然后，在ActivityB中：

layout布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.github.hexiangyuan.sharedelementtransitionsdemo.ActivityA">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="match_parent"
        android:layout_height="184dp"
        android:scaleType="centerCrop"
        android:transitionName="@string/transitions_name"
        android:src="@drawable/image" />

    <TextView
        android:id="@+id/textView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentEnd="true"
        android:layout_alignParentRight="true"
        android:layout_below="@+id/imageView"
        android:layout_marginEnd="16dp"
        android:layout_marginRight="16dp"
        android:text="这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容这是内容"
        android:textSize="16sp" />

</RelativeLayout>
```

java

```java
public class ActivityB extends AppCompatActivity{

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_b);
    }
}
```

在ActivityB中就只需要在共享元素上添加一个属性
`android:transitionName="@string/transitions_name"`就OK了；

那么，Acitity to Activity 的共享动画就完成了，赶快Run 一下看看成果吧，是不是很简单？So easy！

![finished](http://om6hh53na.bkt.clouddn.com/share_part1.gif)

未完待续.......

下期预告 part2 -- Fragment to Fragment