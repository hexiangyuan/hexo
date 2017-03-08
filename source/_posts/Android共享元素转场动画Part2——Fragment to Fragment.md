---
title: Android共享元素转场动画Part2——Fragment to Fragment
---
继续[Part1部分--Activity to Activity](https://hexiangyuan.github.io/2017/03/02/%E5%85%B1%E4%BA%AB%E5%85%83%E7%B4%A0%E8%BD%AC%E5%9C%BA%E5%8A%A8%E7%94%BBPart1--ActivityToActivity/)，这个部分我们简单介绍下Fragment to Fragment的共享动画实现；

[github源码](https://github.com/hexiangyuan/SharedElementTransitionDemo/tree/fragmentToFragment)

## 先看效果
![fragmentToFragment](http://om6hh53na.bkt.clouddn.com/fragmentTransition.gif)

## Fragment to Fragment

首先我们需要创建一个Activity容器来加载Fragment：
```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Fragment fragment = getSupportFragmentManager().findFragmentByTag(FragmentA.class.getName());
        if (fragment == null) {
            fragment = FragmentA.newInstance();
            getSupportFragmentManager().beginTransaction().add(R.id.activity_main,
                fragment,
                FragmentA.class.getName())
                .commit();
        }
    }
}
```

按照代码所示，我们现在Activity中加载FragmentA 然后由FragmentA跳转到FragmentB，并且实现共享动画。

下面实现FragmentA：

FragmentA中的xml代码实现，一个ImageView 和一个Button ，其中ImageView为FragmentA的共享元素，并且为他设置属性` android:transitionName="simple transition name"`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.github.hexiangyuan.sharedelementtransitionsdemo.MainActivity">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:scaleType="centerCrop"
        android:transitionName="simple transition name"
        android:src="@drawable/image" />

    <Button
        android:id="@+id/btn_click"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me"
        android:textSize="16sp"
        android:layout_below="@+id/imageView" />

</RelativeLayout>
```
FragmentA的java代码

```java
public class FragmentA extends Fragment {
    public static final String TAG = FragmentA.class.getSimpleName();
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_a, container, false);
    }

    public static FragmentA newInstance() {
        return new FragmentA();
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final ImageView imageView = (ImageView) getView().findViewById(R.id.imageView);
        getActivity().findViewById(R.id.btn_click).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Fragment fragmentB = getFragmentManager().findFragmentByTag(TAG);
                if (fragmentB == null) fragmentB = FragmentB.newInstance();
                getFragmentManager()
                        .beginTransaction()
                        .addSharedElement(imageView,
                        ViewCompat.getTransitionName(imageView))
                        .addToBackStack(TAG)
                        .replace(R.id.activity_main, fragmentB)
                        .commit();
            }
        });
    }
}
```
值得注意的是在`addShareElement()`这个方法以及`addToBackStack()`这个方法；

*   addShareElement:设置了作为共享元素的控件以及transitionName;
*   addToBackStack：为了让fragment回栈，如果不设置这个回栈，当跳转到fragmentB的时候，BackClick就会直接退出Activity；

那么在FragmentB就比较容易实现了：

FragmentB：
```xml
 <ImageView
        android:id="@+id/imageView"
        android:layout_width="match_parent"
        android:layout_height="184dp"
        android:scaleType="centerCrop"
        android:transitionName="simple transition name"
        android:src="@drawable/image" />
```

```java
   @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            setSharedElementEnterTransition(
                    TransitionInflater.from(getContext())
                            .inflateTransition(android.R.transition.move));
        }
    }
```
只需要在FragmentB的
  
*   xml 的共享控件里面设置` android:transitionName="simple transition name"`
*   onCreate里面设置动画为android.R.transition.move

## 加载网络图片的元素共享（以Picasso为例）

一般在App中，我们的ImageView都是在网络URL获取的资源，那么网络加载的ImageView也是可以实现共享元素转换的，下面我们就以Picasso为例：

### 添加依赖
```
 compile 'com.squareup.picasso:picasso:2.5.2'
```

### FragmentA中
```java
Picasso.with(getActivity())
                .load("https://s3-us-west-1.amazonaws.com/powr/defaults/image-slider2.jpg")
                .fit()
                .centerCrop()
                .into(imageView);
                 getActivity().findViewById(R.id.btn_click).setOnClickListener(new View.OnClickListener() {
     @Override
public void onClick(View view) {
                Fragment fragmentB = getFragmentManager().findFragmentByTag(TAG);
                if (fragmentB == null) fragmentB = FragmentB.newInstance();
                getFragmentManager()
                        .beginTransaction()
                        .addSharedElement(imageView,
                         ViewCompat.getTransitionName(imageView))
                        .addToBackStack(TAG)
                        .replace(R.id.activity_main, fragmentB)
                        .commit();
            }
        });
```

## FragmentB中添加加载图片的:
```java
  @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        ImageView imageView = (ImageView) getView().findViewById(R.id.imageView);
        Picasso.with(getContext())
                .load("https://s3-us-west-1.amazonaws.com/powr/defaults/image-slider2.jpg")
                .fit()
                .centerCrop()
                .noFade()
                .into(imageView, new Callback() {
                    @Override
                    public void onSuccess() {
                        startPostponedEnterTransition();
                    }

                    @Override
                    public void onError() {

                    }
                });
    }
```

*   添加noFade禁用渐隐的效果促使动画更加的流畅。
*   在CallBack的onSuccess()中设置`startPostponedEnterTransition()`。

[github banch ](https://github.com/hexiangyuan/SharedElementTransitionDemo/tree/picassoFragment)

[Blog](https://sheepyang.cn)

非常感谢，你能耐心读完；
