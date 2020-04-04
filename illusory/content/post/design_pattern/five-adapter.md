---
title: "Java设计模式(五)---适配器模式"
description: "适配器模式实现、原理及优缺点分析"
date: 2018-10-15 22:00:00
draft: false
tags: ["Java","设计模式"]
categories: ["设计模式"]
---

本文主要介绍了Java23种设计模式之适配器模式，并结合实例描述了适配器模式的具体实现和优缺点分析。

<!--more-->

> **[Java设计模式系列文章目录](https://www.lixueduan.com/categories/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/)**
>
> [Java设计模式(一)---单例模式](https://www.lixueduan.com/posts/53093.html)
>
> [Java设计模式(二)---工厂模式](https://www.lixueduan.com/posts/34710.html)
>
> [Java设计模式(三)---建造者模式](https://www.lixueduan.com/posts/52453.html)
>
> [Java设计模式(四)---原型模式](https://www.lixueduan.com/posts/24b6c0e4.html)
>
> [Java设计模式(五)---适配器模式](https://www.lixueduan.com/posts/f444ac9.html)
>
> [Java设计模式(六)---装饰者模式](https://www.lixueduan.com/posts/75903408.html)
>
> [Java设计模式(七)---代理模式](https://www.lixueduan.com/posts/ae2a93bd.html)
>
> [Java设计模式(八)---外观模式](https://www.lixueduan.com/posts/22a51705.html)
>
> [Java设计模式(九)---享元模式](https://www.lixueduan.com/posts/34e634e7.html)
>
> [Java设计模式(十)---策略模式](https://www.lixueduan.com/posts/a7982bdc.html)
>
> [Java设计模式(十一)---模板方法模式](https://www.lixueduan.com/posts/57ae709c.html)
>
> [Java设计模式(十二)---观察者模式](https://www.lixueduan.com/posts/48bcf013.html)
>
> [Java设计模式(十三)---组合模式](https://www.lixueduan.com/posts/a340063f.html)
>
> ........
>
> Demo下载--> [Github](https://github.com/illusorycloud/design-pattern)
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)



## 1. 简介

> 适配器模式将一个接口转换成客户希望的另外一个接口。它使得原来由于接口不兼容而不能在一起工作的那些类可以一起工作。
>
> **把一个类的接口变换成客户端所期待的另一种接口**

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/design_pattern/five-adapter.jpeg)

用到的对象

* **Target**
   ---定义Client使用的与特定领域相关的接口。
* **Client**
   ---与符合Target接口的对象协同。
* **Adaptee**
   ---定义一个已经存在的接口，这个接口需要适配。
* **Adapter**
   ---对Adaptee的接口与Target接口进行适配

## 2. 类适配器模式

**原理**：

> 通过继承来实现适配器功能。

当我们要访问的类A中没有我们想要的方法 ，却在另一个接口B中发现了合适的方法，但我们又不能改变类A。

在这种情况下，我们可以定义一个适配器p来进行中转，这个适配器p要继承我们访问类A，这样我们就能继续访问当前类A中的方法（虽然它目前不是我们的菜），然后再实现接口B，这样我们可以在适配器P中访问接口B的方法了。

```java
/**
 * 目标类
 *
 * @author illusoryCloud
 */
public interface Target {
    void Target();
}

/**
 * 被适配类
 * 只有Adaptee方法 但是目标接口要Target方法
 *
 * @author illusoryCloud
 */
public class Adaptee {
    public void Adaptee() {
        System.out.println("这是现有的方法");
    }
}

/**
 * 适配器类
 * 继承Adaptee类 使得此类保留了Adaptee方法
 * 实现Target接口 使得此类同时也拥有Target方法
 * 适配完成
 *
 * @author illusoryCloud
 */
public class Adapter extends Adaptee implements Target {
    @Override
    public void Target() {
        System.out.println("这是目标方法");
    }
}

/**
 * 类适配器模式 测试类
 *
 * @author illusoryCloud
 */
public class ClassAdapterTest {
    @Test
    public void classAdapterTest() {
        //Target类型的对象 同时拥有Target()和Adaptee()方法
        Target target = new Adapter();
        //这是目标方法
        target.Target();
        //这是现有的方法
        ((Adapter) target).Adaptee();
    }
}

```

## 3. 对象适配器模式

原理

> 基本思路和类的适配器模式相同，只是将 Adapter 类作修改，这次不继承 Adaptee 类，而是持有 Adaptee 类的实例，以达到解决兼容性的问题。

```java
//-------Target和Adaptee与上面一样----------

/**
 * 适配器类 持有Adaptee对象来代替继承Adaptee类
 * 传入Adaptee对象 使得此类同样拥有Adaptee方法
 * 实现Target接口 使得此类同时也拥有Target方法
 * 适配完成
 *
 * @author illusoryCloud
 */
public class Adapter implements Target {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void Target() {
        System.out.println("这是目标方法");
    }

    public void Adaptee() {
        adaptee.Adaptee();
    }
}

/**
 * 对象适配器模式 测试类
 *
 * @author illusoryCloud
 */
public class ObjectAdapterTest {
    @Test
    public void objectAdapterTest() {
        Target target = new Adapter(new Adaptee());
        //这是目标方法
        target.Target();
        //这是现有的方法
        ((Adapter) target).Adaptee();

    }
}
```

## 4. 总结

**适配器模式优点**

1、**有更好的复用性**。系统需要使用现有的类，但此类接口不符合系统需要，通过适配器模式让这些功能得到很好的复用

2、**有更好的扩展性**。实现适配器，可以调用自己开发的功能

**缺点**

过多使用适配器会使得系统非常凌乱，明明调用的是A接口，内部却被适配成了B接口。因此除非必要，不推荐使用适配器，而是作为一种补救措施，条件允许的情况下推荐直接对系统重构。

**适配器模式在JDK中的应用**

`InputStreamReader/OutputStreanWriter`

创建InputStreamReader对象的时候必须在构造函数中传入一个InputStream实例，然后InputStreamReader的作用就是将InputStream适配到Reader。很显然，适配器就是InputStreamReader，源角色就是InputStream代表的实例对象，目标接口就是Reader类。

## 5. 参考

`https://www.cnblogs.com/xrq730/p/4906487.html`