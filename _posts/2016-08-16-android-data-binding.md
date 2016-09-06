---
layout: post
title:  "Data binding学习"
date:   2016-08-16 01:56
categories: Android
tags: Android data-binding
---


* content
{:toc}


Google在去年下半年的时候推出了Data binding的support包，当时用起来似乎挺麻烦，不过随着Android Studio 2.0的发展，Google不仅让Data binding支持包变得更易用，也提升了AS对Data binding在IDE上的支持。

随着界面的内容越来越丰富以及项目工程的扩大，在代码里控制UI控件的显示和更新会变得越来越麻烦，你会发现你需要写大量的findViewById函数，获取大量的控件实例，看着都嫌累。

Google推出Data binding就是为了缓解这种问题，它可以让一些基本的UI代码转移到XML资源文件中。Data binding其实就是Android MVVM框架的核心，至于MVVM和MVP有什么不同，看模式图很难看出来，其实MVC，MVP，MVVM这些模式随着Web发展出来的，感觉这些也只是一些理论，随着开发者的使用很容易有不同的实现版本出现，可能MVVM和MVP也只是大同小异罢了。
根据MVVM的定义，分为Model，View和ViewModel（也就是Presenter啦），不过在学习了Data binding后，个人感觉ViewModel和Presenter的差别在于，Presenter更多的是扮演着请求和响应的加工和转交，而ViewModel则更像M和V二者的过渡态，其中一方的改变都可以在VM上看出来，并且在另一方上自动产生相应的变化。

好了不废话了，接下来就开始对Data binding的学习






----------
## 环境配置
正如data binding是一个support library，因此只要在SDK Manager里下载最新的Android Support Repository就能获得。

Data binding支持Android 2.1（API 7）以上版本，并且需要Android Studio 1.3以上版本提供兼容

在对应module的build.gradle中，进行如下配置：

```
android {
	...
	dataBinding {
		enabled true
	}
}
```

这样就配置完毕了。要注意的一点是，如果依赖的库中使用了data binding，那么app module中也需要进行配置


----------


## 小试牛刀
假设有一个TextView，需要显示一个字符串，最一般的做法就是在Activity中用findViewById获得这个控件的实例，然后调用setText方法设置字符串。下面来看看data binding的做法
和普通的布局文件不同的一点是，使用data binding必须用layout标签作为根元素，里面紧接着是data标签，然后才是原来的布局

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="student"
            type="com.example.mahong.test.Student" />
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{student.name}" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{student.number}"/>
    </LinearLayout>
</layout>
```

假设我们需要显示一个学生的名字和学号，正如上面LinearLayout中写的，它也是原来的布局。这里稍微有点看不懂的就是data标签，它里面有个variable标签，顾名思义，表示的就是一个变量，而从name和type属性也可以看得出它们分别表示这个变量的名字和类型。

再看看这两个TextView，它们的text属性设置使用到了特殊的表达式，格式是@{}，中间的语句引用了这个变量的name和number成员

没错，我们需要创建一个POJO类，来提供数据。这个POJO类可以是这样的：

```java
public class Student {
    public final String name;
    public final String number;

    public Student(String name, String number) {
        this.name = name;
        this.number = number;
    }
}
```

也可以是这样的一个java Bean：

```java
public class Student {
    public String name;
    public String number;

    public Student(String name, String number) {
        this.name = name;
        this.number = number;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }
}
```

这取决你对数据的要求，如果你希望提供的数据是一次性的，不容修改的，就使用第一种。总之必须要让你需要用到的域是可访问的（使其为public或者提供get方法）

类也定义好了，那么xml中使用到的这个变量是由谁提供的呢？当然就是Model了，对应的也就是Activity：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

    binding.setStudent(new Student("Mahong", "978"));
}
```

Data binding会自动地产生一个Binding类，类名默认是对应xml文件名地PASCAL命名形式加上“Binding”后缀，位置默认在module所在包中的databinding包

把原先的setContentView方法替换成DataBindingUtil.setContentView方法，需要提供Context对象和对应的layout ID，该方法会返回对应的Binding类，然后调用类中的set方法提供Student实例

当然了，也可以动态地inflate：

```java
LayoutTestBinding binding1 = LayoutTestBinding.inflate(getLayoutInflater());
```
至于在列表适配器中inflate则还要加入viewRoot等参数

就这样，Module，View和ViewModel的设置就完成了


----------
## 事件处理
Data binding允许通过表达式处理视图分发的事件。根据Google官方说明，有两种方式：

 - 方法引用（Method References）
 - 监听器绑定（Listener Bindings）

在方法引用的方式中，事件将直接绑定到一个方法上，这跟我们之前常用的onClick绑定到Activity的一个方法上的做法相似，不过方法引用的一个好处就是它可以在编译时检查方法签名是否符合要求。方法引用的签名必须和监听器方法的签名一致，比如OnClickListener的onClick方法，data binding会根据你提供的方法，自动生成一个监听器

虽然要求签名一致，但我发现方法名不同也是可以的。。。可能这里指的是返回类型和参数列表吧

我尝试了一下：

```
<Button
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="mahong"
	android:onClick="@{listener::onClick}"/>
```

发现AS居然对格式报错

![](http://img.blog.csdn.net/20160816152716708)

虽然报错但编译还是能通过的，而且运行也是没问题

后来在Android Developer的Youtube频道上看到的写法是这样的：

```
android:onClick="@{listener.onClick}"
```

虽然AS没报错了，但方法那边还是警告never used

不知道是库的问题还是IDE的支持没做好，可能后面的版本会修复吧

监听器绑定（Listener Bindings）绑定的是一个Lambda表达式，当事件发生时进行表达式的计算，而方法引用则是数据绑定时就创建了监听器，因此listener bindings的方法要更为灵活

这里要吐槽一下，既然listener bindings是绑定表达式，那为什么还要起这么一个很容易混淆的名字。。。不过这也只是个叫法罢了

可以使用一个回调函数：

```
android:onClick="@{() -> listener.handle(true)}"
```

可以看到，这里并没有给出onClick方法所需要的View参数，这是因为Listener Bindings对监听方法参数提供了两种选择，要么都忽略掉，要么每个参数都起个名字，这取决你是否需要使用到这些参数。所以上面那个语句可以这样写：

```
android:onClick="@{(view) -> listener.handle(true)}"
```

Data binding会尽量避免运行时安全，因此Data binding会对不同的类型提供默认的返回值和初始值，object对应null，int对应0，boolean对应false，等等

注意，有些View的点击监听器不是通过onClick属性设置的：

 - SearchView    android:onSearchClick
 - ZoomControls    android:onZoomIn，android:onZoomOut


----------
## 一些细节
在data标签还可以使用import：

```
<data>
    <import type="com.example.mahong.test.Listener" />
</data>
```

效果相当于java的import，可以直接使用这个类中的静态成员

如果import了两个同名的类，则需要给其中一个起个别名：

```
<data>
    <import type="android.view.View" />
    <import type="com.example.mahong.test.View"
            alias="MyView"/>
</data>
```

java.lang.*是自动import的，所以可以直接使用


----------
## 变量

上面所写的变量都是编译时确定下来的，这些变量发生的变化将不能被观察到，也就是说不能动态地产生视图的变化。如果需要动态的反射机制，则需要使对应的类继承自Observable或者是一个observable collection，后面会讲到

如果不同的configuration有对应的layout，要注意变量名冲突问题

一个特殊的变量context是默认产生的，它表示当前root view的Context，可直接使用


----------
## 自定义Binding类名

布局对应的Binding类的类名，默认是该布局名字的PASCAL命名形式（去掉符号，单词首字母大写），最后加上Binding。当然也可以自定义Binding的类名：

```
<data class="MyCustom">
    ...
</data>
```

注意，这样产生的Binding类名将是MyCustom，而不是MyCustomBinding

所有的Binding类都默认放在app包下的databinding包。可以自定义位置：

```
<data class=".MyCustom">
    ...
</data>
```

这样将直接放在app包中。当然也可以设置完整的路径。


----------
## Includes
如果一个layout中include了另一个layout，可以将自己的变量赋值给它的变量：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:bind="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="text"
            type="String" />
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <include layout="@layout/layout_test1"
            bind:text="@{text}" />
    </LinearLayout>
</layout>
```


----------
## 表达式
可以使用的java元素有：

 - 数学运算符
 - 字符串连接符
 - 逻辑运算符
 - 位运算符
 - 一元符
 - 位移运算符
 - 比较符
 - instance of
 - 小括号
 - 方法调用
 - 域访问
 - 数组访问[]
 - 三元运算符

不能使用的有：

 - this
 - super
 - new
 - 显示泛型调用（啥意思？）

支持空合并运算符??

无论域是否私有，都可以直接用 . 访问，会自动调用其get方法

可以在表达式中直接写字符串，但是字符串的引号不能和整个表达式所在的引号相同，可以使用单引号 ' 或者反引号 `

可以在表达式中直接调用资源


----------
## 数据对象
普通的POJO不能用于更新UI

### Observable对象

实现Observable接口，可以让binding附上一个监听器，来监听约束对象的属性变化

Observable接口包含了添加和溢出监听器的机制，但是通知的任务交给了开发者。因此一般选择继承BaseObservable基类

需要在get方法添加Bindable注解，在set方法中最后进行通知：

```java
public class Textpack extends BaseObservable {
    public String text;

    public Textpack(String text) {
        this.text = text;
    }

    @Bindable
    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
        notifyPropertyChanged(com.example.mahong.test.BR.text);
    }
}
```

BR类是自动生成的

### Observable域

一种更方便的写法是

```java
public class Textpack extends BaseObservable {
    public final ObservableField<String> text = new ObservableField<>();
}
```

这种方法提供了更细粒度的方式，每个域都是独立的observable对象

另外还为各种基本类型提供了对应的类，如ObservableBoolean，ObservableInt等

ObservableField支持自动打包和解包

设置数据时则这样写：

```java
TextPack pack = new TextPack();
pack.text.set("mahong");
String text = pack.text.get();
```

### Observable集合
Data binding只提供了ObservableArrayMap和ObservableArrayList两个已实现了的容器类

这两个容器类分别实现的是ObservableMap和ObservableList接口


----------
## Binding类
这些自动生成的Binding类相当于MVVM结构中的VM，是数据和视图之间的桥梁。这些类都继承自ViewDataBinding

绑定一个布局，一般的写法是静态的inflate：

```java
LayoutTestBinding binding1 = LayoutTestBinding.inflate(inflater);
LayoutTestBinding binding1 = LayoutTestBinding.inflate(inflater, viewGroup, false);
```

有时候Binding类的具体类型不可知，则可以根据layout进行创建：

```java
ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
    parent, attachToParent);
ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
```

### 具有ID的控件
如果一个控件具有ID，那么所在layout的Binding类将会有对应的public final域，对于某些View来说，这种方式会比findViewById更快

### 变量
binding layout中的每个变量都会在对应的Binding类中有访问方法

### ViewStub
ViewStub有助于Android的布局性能优化，其对应的子布局在正式inflate之前不会显示出来，在inflate之后，这个ViewStub引用会被置空

那么问题来了，Binding类中的View都是final的，ViewStub在inflate后又要置空，那怎么办呢

于是Android新添加了类ViewStubProxy，相当于在ViewStub外套了层壳，ViewStub置空并不会影响到它的代理。从Binding对象获得的View是ViewStubProxy类型

如果这个ViewStub对应的布局也是一个binding layout，那么需要给它的ViewStubProxy注册一个OnInflateListener，在inflate发生后进行相应的处理。示例如下：

```java
binding.viewstub.setOnInflateListener(new ViewStub.OnInflateListener() {
    @Override
    public void onInflate(ViewStub viewStub, View view) {
        LayoutTest1Binding binding1 = DataBindingUtil.bind(view);
        binding1.setText("mahong");
    }
});
```

由于我们在binding.viewstub是ViewStubProxy类型，因此要想ViewStub显示出来，就要先调用getViewStub()方法获得对应的ViewStub：

```java
binding.viewstub.getViewStub().inflate();
```

可能是IDE的支持还没做好，ctrl+左查看binding.viewstub的类型时仍然是ViewStub类型，导致上面那条语句在现在最新的AS中还是红的，但是编译是通过的

### 高级Binding
#### 动态变量
有些时候，Binding类的确切类型可能无法获知。比如在RecyclerView的适配器中，因为绑定的layout是任意的，所以无法知道确切的Binding类型，但在onBindViewHolder方法被调用的时候必须对其进行设置

所以需要在ViewHolder中保存其对应的Binding对象

另外，由于通用性，ViewHolder中的Binding引用需要为ViewDataBinding类型，所以在onBindViewHolder方法中设置其中的变量时，需要使用setVariable方法

#### 立即Binding
当一个变量或者observable改变，在下一帧刷新前会进行binding的调度。有时候会需要binding立即执行，这时可以调用executePendingBingings()

#### 后台线程
Data binding在计算时会本地化每个变量或域，因此不用担心同步问题（集合除外）

后面的自定义Setter和Conversion好像应用的空间不大，以后有空再研究吧

就酱

