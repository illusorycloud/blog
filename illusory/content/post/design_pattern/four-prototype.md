---
title: "Java设计模式(四)---原型模式"
description: "原型模式实现与应用场景、优缺点分析"
date: 2018-10-14 22:00:00
draft: false
tags: ["Java","设计模式"]
categories: ["设计模式"]
---

本文主要介绍了Java23种设计模式中的原型模式，并结合实例描述了原型模式的具体实现和应用场景，优缺点分析等。

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

### 1.1 原型模式

![](https://github.com/lixd/blog/raw/master/illusory/content/post/design_pattern/images/four-prototype.gif)

**原型模式的思想就是将一个对象作为原型，对其进行复制、克隆，产生一个和原对象类似的新对象**。简单地说原型模式就是创建复杂对象的时候使用克隆手段来代替新建一个对象。当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过一个已有实例可以提高新实例的创建效率。

 原型模式主要包含如下三个角色：

* Prototype：抽象原型类。声明克隆自身的接口。 
* ConcretePrototype：具体原型类。实现克隆的具体操作。 
* Client：客户类。让一个原型克隆自身，从而获得一个新的对象。

### 1.2 Java中的克隆

我们需要知道，Java中的对象克隆分为浅克隆和深克隆。

* 浅克隆：将一个对象克隆后，基本数据类型的变量都会重新创建，而引用类型，指向的还是原对象所指向的。

* 深克隆：将一个对象克隆后，不论是基本数据类型还有引用类型，都是重新创建的。

简单来说，就是深克隆进行了完全彻底的克隆，而浅克隆不彻底。



## 2. 实现

```java
//实现Cloneable接口浅克隆，Serializable接口深克隆
/**
 * 构建的消息对象
 * 普通对象
 *
 * @author illusoryCloud
 */
public class Message implements Serializable, Cloneable {
    /**
     * 标题
     */
    private String Title;
    /**
     * 内容
     */
    private String Content;
    /**
     * 发送者
     */
    private User From;
    /**
     * 接收者
     */
    private User To;
    /**
     * 时间
     */
    private Date Time;

    /**
     * 浅克隆
     *
     * @return
     * @throws CloneNotSupportedException
     */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 深克隆
     *
     * @return Message对象
     * @throws IOException
     * @throws ClassNotFoundException
     */
    public Message deepClone() throws IOException, ClassNotFoundException {
        // 写入当前对象的二进制流
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        // 读出二进制流产生的新对象

        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return (Message) ois.readObject();
    }
			//省略Getter、Setter、toString、构造函数等
    }

//User类 Message类中引用 
/**
 * 用户类 被消息类引用
 * 主类实现深克隆 则被引用类也得实现Serializable接口
 * @author illusoryCloud
 */
public class User implements Serializable {
    private String name;
    private int age;
	//省略Getter、Setter、toString、构造函数等
}
//测试
  /**
 * 原型模式测试类
 *
 * @author illusoryCloud
 */
public class PrototypeTest {
    @Test
    public void prototypeTest() {
        User zhangsan = new User("张三", 22);
        User lisi = new User("李四", 23);
        Message message = new Message();
        message.setTitle("hello");
        message.setContent("how are you~");
        message.setFrom(zhangsan);
        message.setTo(lisi);
        message.setTime(new Date());
        Message cloneOne = null;
        Message cloneTwo = null;
        try {
            cloneOne = (Message) message.clone();
            cloneTwo = message.deepClone();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //false  克隆实现的是一个(和原对象相似的)新对象
        System.out.println(message == cloneOne);
        //false
        System.out.println(message == cloneTwo);
        //true 浅克隆 引用对象指向的还是原对象
        System.out.println(message.getFrom()==cloneOne.getFrom());
        //false 深克隆 引用对象也重新创建
        System.out.println(message.getFrom()==cloneTwo.getFrom());

    }

}
```

## 3. 总结

**为什么要用原型模式**

通过复制已有的对象，可以简化对象的创建过程，提高创建对象的效率。

深克隆保存对象状态，实现撤销恢复功能。

**缺点**：

在实现深克隆时需要编写复杂的代码。

需要为每一个类写一个克隆方法，如果要深克隆，则类中的每一层对象的类都得支持深克隆，代码比较复杂。

**应用场景**

创建对象成本高。

当一个系统应该独立于它的产品创建、构成和表示时，要使用原型模式。

当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。

**原型模式在Java中的应用及解读**

只要是实现了Cloneable接口的类都可以算是原型模式的应用，比如ArrayList。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ...
    public Object clone() {
    try {
        ArrayList<E> v = (ArrayList<E>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError();
    }
    }
    ...
}
```

程序中获取到了一个ArrayList的实例arrayList，我们完全可以通过调用arrayList.clone()方法获取到原ArrayList的拷贝。

## 4. 参考

`https://www.cnblogs.com/xrq730/p/4905907.html`

