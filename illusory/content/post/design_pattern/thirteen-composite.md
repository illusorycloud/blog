---
title: "Java设计模式(十三)---组合模式"
description: "组合模式的具体实现和性能分析"
date: 2018-10-26 22:00:00
draft: false
tags: ["Java","设计模式"]
categories: ["设计模式"]
---

本文主要介绍了Java23种设计模式中的组合模式，并结合实例描述了组合模式的具体实现和性能分析测试。

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

**组合模式：将对象组合成树形结构以表示”部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性**。 

> **所谓组合模式，其实说的是对象包含对象的问题，通过组合的方式（在对象内部引用对象）来进行布局**
>
> 以Windows文件系统为例，文件夹下可能有文件，也可能还有一个文件夹。
> 文件夹可以包含文件和文件夹，但文件却没有这些功能。所以实现的时候需要单独实现。
> 如果用组合模式的话，将文件和文件夹看成一个整体。都是文件。当做抽象的文件。

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/design_pattern/thirteen-composite.jpg)

## 2. 代码实现

```java
/**
 * 组合模式
 *
 * @author illusoryCloud
 */
public class Employee {
    private String name;
    /**
     * 职位
     */
    private String dept;
    /**
     * 工资
     */
    private int salary;
    /**
     * 下属 一个Employee集合
     */
    private List<Employee> subordinates;

    public Employee(String name, String dept, int sal) {
        this.name = name;
        this.dept = dept;
        this.salary = sal;
        subordinates = new ArrayList<Employee>();
    }

    public void add(Employee e) {
        subordinates.add(e);
    }

    public void remove(Employee e) {
        subordinates.remove(e);
    }

    public List<Employee> getSubordinates() {
        return subordinates;
    }

    @Override
    public String toString() {
        return ("Employee :[ Name : " + name
                + ", dept : " + dept + ", salary :"
                + salary + " ]");
    }
}

/**
 * 组合模式 测试类
 *
 * @author illusoryCloud
 */
public class CompositeTest {
    @Test
    public void compositeTest(){
        Employee CEO = new Employee("John","CEO", 30000);

        Employee headSales = new Employee("Robert","Head Sales", 20000);

        Employee headMarketing = new Employee("Michel","Head Marketing", 20000);

        Employee clerk1 = new Employee("Laura","Marketing", 10000);
        Employee clerk2 = new Employee("Bob","Marketing", 10000);

        Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
        Employee salesExecutive2 = new Employee("Rob","Sales", 10000);

        CEO.add(headSales);
        CEO.add(headMarketing);

        headSales.add(salesExecutive1);
        headSales.add(salesExecutive2);

        headMarketing.add(clerk1);
        headMarketing.add(clerk2);

        //打印该组织的所有员工
        System.out.println(CEO);
        for (Employee headEmployee : CEO.getSubordinates()) {
            System.out.println(headEmployee);
            for (Employee employee : headEmployee.getSubordinates()) {
                System.out.println(employee);
            }
        }
    }
}
```

## 3. 参考

`https://blog.csdn.net/qq_40709468/article/details/81990084`

`http://www.runoob.com/design-pattern/composite-pattern.html`