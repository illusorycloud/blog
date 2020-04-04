---
title: "Java设计模式(九)---享元模式"
description: "享元模式的实现及其和单例模式的对比"
date: 2018-10-22 22:00:00
draft: false
tags: ["Java","设计模式"]
categories: ["设计模式"]
---

本文主要介绍了Java23种设计模式中的享元模式，并结合实例描述了享元模式的具体实现，具体优缺点和单例模式的对比。

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



## 1. 享元模式介绍

享元模式：“享”就是分享之意，指一物被众人共享，而这也正是该模式的终旨所在。

> 享元模式有点类似于单例模式，都是只生成一个对象来被共享使用。存储这些共享实例对象的地方称为**享元池** 。享元对象能做到共享的关键是区分了**内部状态(Intrinsic State)**和**外部状态(Extrinsic State)**。
>
>  **内部状态是存储在享元对象内部并且不会随环境改变而改变的状态，内部状态可以共享**。如围棋中的的黑棋白棋，不会随外部环境的变化而变化，无论在任何环境下黑棋始终是黑棋。
>
> **外部状态是随环境改变而改变的、不可以共享的状态**。享元对象的外部状态通常由客户端保存，并在享元对象被创建之后，需要使用的时候再传入到享元对象内部。比如每颗棋子的位置是不同的。

围棋中的黑棋和白棋可以是共享的对象，不用每次都创建一个新的对象。这样就只需要创建黑棋和白棋两个对象了。颜色是不会变得，所以是内部状态。落下得位置是随机的，所以作为外部状态。

![pure](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/design_pattern/nine-flyweight-pure.png)

## 2. 单纯享元模式

　　在单纯的享元模式中，所有的享元对象都是可以共享的。

　　单纯享元模式所涉及到的角色如下：

　　●　　**抽象享元(Flyweight)角色 ：**给出一个抽象接口，以规定出所有具体享元角色需要实现的方法。

　　●　　**具体享元(ConcreteFlyweight)角色：**实现抽象享元角色所规定出的接口。如果有内蕴状态的话，必须负责为内蕴状态提供存储空间。

　　**●　　享元工厂(FlyweightFactory)角色** ：本角色负责创建和管理享元角色。本角色必须保证享元对象可以被系统适当地共享。当一个客户端对象调用一个享元对象的时候，享元工厂角色会检查系统中是否已经有一个符合要求的享元对象。如果已经有了，享元工厂角色就应当提供这个已有的享元对象；如果系统中没有一个适当的享元对象的话，享元工厂角色就应当创建一个合适的享元对象。

```java
/**
 * 单纯享元模式 抽象享元角色
 *
 * @author illusoryCloud
 */
public interface Ball {
    /**
     * 简单的show方法
     * 根据传入的参数(外蕴状态)不同而产生不同的表现
     *
     * @param color 外蕴状态
     */
    void show(String color);
}

/**
 * 单纯享元模式 具体享元角色
 * 内蕴状态为type 即球的类型 由构造方法传入
 * 外蕴状态为color 即球的颜色 作为show()方法的参数传入
 *
 * @author illusoryCloud
 */
public class ConcreteBall implements Ball {
    private String type;

    public ConcreteBall(String type) {
        this.type = type;
    }

    @Override
    public void show(String color) {
        System.out.println("这是一个：" + color + "的" + type);
    }
} 

/**
 * 单纯享元模式 享元工厂角色
 *
 * @author illusoryCloud
 */
public class BallFactory {
    /**
     * 将对象存在map中
     */
    private static Map<String, Ball> factory = new HashMap<>();

    /**
     * 获取单纯享元角色
     *
     * @param type 内蕴状态
     * @return 具体享元角色
     */
    public Ball getBall(String type) {
        Ball ball = factory.get(type);
        if (ball == null) {
            //如果对象不存在则创建一个新的对象
            ball = new ConcreteBall(type);
            //把这个新的Flyweight对象添加到缓存中
            factory.put(type, ball);
        }
        return ball;
    }

    /**
     * 静态内部类 单例模式
     */
    private BallFactory() {
    }

    private static class SingletonHolder {
        private static final BallFactory INSTANCE = new BallFactory();
    }

    public  static BallFactory getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
/**
 * 单纯享元模式 测试类
 *
 * @author illusoryCloud
 */
public class PureTest {
    /**
     * 当客户端需要单纯享元对象的时候，需要调用享元工厂的factory()方法，
     * 并传入所需的单纯享元对象的内蕴状态，由工厂方法产生所需要的享元对象。
     */
    @Test
    public void flyWeightTest() {
        BallFactory ballFactory = BallFactory.getInstance();
        Ball basketball = ballFactory.getBall("篮球");
        Ball football = ballFactory.getBall("足球");
        basketball.show("红色");
        basketball.show("黄色");
        football.show("黑色");
        football.show("白色");
        Ball basketball2 = ballFactory.getBall("篮球");
        Ball football2 = ballFactory.getBall("足球");
        //true 都是同一个对象
        System.out.println(basketball.equals(basketball2));
        //true
        System.out.println(football.equals(football2));

    }
}
//输出
这是一个：红色的篮球
这是一个：黄色的篮球
这是一个：黑色的足球
这是一个：白色的足球
true
true
```

## 3. 复合享元模式

　　在单纯享元模式中，所有的享元对象都是单纯享元对象，也就是说都是可以直接共享的。还有一种较为复杂的情况，将一些单纯享元使用合成模式加以复合，形成复合享元对象。这样的复合享元对象本身不能共享，但是它们可以分解成单纯享元对象，而后者则可以共享。

![composite](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/design_pattern/nine-flyweight-composite.png)

　　复合享元角色所涉及到的角色如下：

　　●　　**抽象享元(Flyweight)角色 ：**给出一个抽象接口，以规定出所有具体享元角色需要实现的方法。

　　●　　**具体享元(ConcreteFlyweight)角色：**实现抽象享元角色所规定出的接口。如果有内蕴状态的话，必须负责为内蕴状态提供存储空间。

　　**●　  复合享元(ConcreteCompositeFlyweight)角色** ：复合享元角色所代表的对象是不可以共享的，但是一个复合享元对象可以分解成为多个本身是单纯享元对象的组合。复合享元角色又称作不可共享的享元对象。

　　**●　  享元工厂(FlyweightFactory)角色** ：本角 色负责创建和管理享元角色。本角色必须保证享元对象可以被系统适当地共享。当一个客户端对象调用一个享元对象的时候，享元工厂角色会检查系统中是否已经有 一个符合要求的享元对象。如果已经有了，享元工厂角色就应当提供这个已有的享元对象；如果系统中没有一个适当的享元对象的话，享元工厂角色就应当创建一个 合适的享元对象。

```java
/**
 * 复合享元模式 复合享元角色
 *
 * @author illusoryCloud
 */
public class CompositeBall implements Ball {
    /**
     * 复合享元角色内部包含多个单纯享元角色
     */
    private Map<String, Ball> composite = new HashMap<>();

    /**
     * 增加一个新的单纯享元对象到集合中
     */
    public void add(String type, Ball ball) {
        composite.put(type, ball);
    }

    /**
     * 遍历的方式挨个调用内部单纯享元角色的show方法
     *
     * @param color 外蕴状态
     */
    @Override
    public void show(String color) {
        Set<String> strings = composite.keySet();
        for (String type : strings) {
            Ball ball = composite.get(type);
            ball.show(color);
        }
    }
}

/**
 * 复合享元模式 复合工厂角色
 *
 * @author illusoryCloud
 */
public class CompositeFactory {
    private Map<String, Ball> factory = new HashMap<String, Ball>();

    /**
     * 获取复合享元
     *
     * @param types 类型集合
     * @return 复合享元对象 包含多个单纯享元对象
     */
    public Ball getComposite(List<String> types) {
        CompositeBall composteBall = new CompositeBall();

        for (String type : types) {
            composteBall.add(type, getPure(type));
        }

        return composteBall;
    }

    /**
     * 获取单纯享元角色
     *
     * @param type 内蕴状态
     * @return 具体享元角色
     */
    public Ball getPure(String type) {
        Ball ball = factory.get(type);
        if (ball == null) {
            //如果对象不存在则创建一个新的对象
            ball = new ConcreteBall(type);
            //把这个新的Flyweight对象添加到缓存中
            factory.put(type, ball);
        }
        return ball;
    }

    /**
     * 静态内部类 单例模式
     */
    private CompositeFactory() {
    }

    private static class SingletonHolder {
        private static final CompositeFactory INSTANCE = new CompositeFactory();
    }

    public static CompositeFactory getInstance() {
        return CompositeFactory.SingletonHolder.INSTANCE;
    }
}

/**
 * 单纯享元模式 测试类
 *
 * @author illusoryCloud
 */
public class CompositeTest {
    /**
     * 当客户端需要单纯享元对象的时候，需要调用享元工厂的factory()方法，
     * 并传入所需的单纯享元对象的内蕴状态，由工厂方法产生所需要的享元对象。
     */
    @Test
    public void flyWeightTest() {
        CompositeFactory compositeFactory = CompositeFactory.getInstance();
        Ball pure = compositeFactory.getPure("篮球");
        Ball pure2 = compositeFactory.getPure("篮球");
        pure.show("红色");
        List<String> types = Arrays.asList("篮球", "足球", "排球");
        Ball composite = compositeFactory.getComposite(types);
        Ball composite2 = compositeFactory.getComposite(types);
        composite.show("蓝色");
        //false 复合享元角色不相同
        System.out.println(composite.equals(composite2));
        //true 单纯享元角色相同
        System.out.println(pure.equals(pure2));
    }
}
//输出
这是一个：红色的篮球
这是一个：蓝色的足球
这是一个：蓝色的篮球
这是一个：蓝色的排球
false
true
```

## 4. 总结

**享元模式的核心在于享元工厂类，享元工厂类的作用在于提供一个用于存储享元对象的享元池，用户需要对象时，首先从享元池中获取，如果享元池中不存在，则创建一个新的享元对象返回给用户，并在享元池中保存该新增对象。**

**优点**：

节约系统的开销，可以少创建对象。
外部状态不会影响内部状态，可以在不同环境下进行共享哦。
**缺点**：

享元模式使逻辑变得更加复杂，需要将享元对象分出内部状态和外部状态。

并且为了使对象可以共享，外部状态在很多情况下是必须有的，当读取外部状态时明显会增加运行时间。

**享元模式使用的场景**：

当我们项目中创建很多对象，而且这些对象存在许多相同模块，这时，我们可以将这些相同的模块提取出来采用享元模式生成单一对象，再使用这个对象与之前的诸多对象进行配合使用，这样无疑会节省很多空间。

**与单例模式的区别**：

享元模式的目的是共享，避免多次创建耗费资源，减少不会要额内存消耗 。

单例模式的目的是限制创建多个对象以避免冲突等 。

## 5. 参考

`http://blog.csdn.net/lovelion>`

`https://blog.csdn.net/Hmily_hui/article/details/80917975`