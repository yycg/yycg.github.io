---
layout:     post
title:      "Effective Java"
subtitle:   ""
date:       2020-03-05 12:00:00
author:     "盈盈冲哥"
header-img: "img/yyy-11.jpg"
mathjax: true
catalog: true
tags:
    - 学习
---

- 覆盖equals时请遵守通用约定

  实现高质量equals方法的诀窍：

  1. 使用==操作符检查“参数是否为这个对象的引用”
  1. 使用instanceof操作符检查“参数是否为正确的类型”
  1. 把参数转换成正确的类型
  1. 对于该类中的每个“关键”域，检查参数中的域是否域该对象中对应的域相匹配（域的比较顺序可能会影响到equals方法的性能。为了获得最佳的性能，应该先比较最优可能不一致的域，或者是开销最低的域）
  1. 当你完成了equals方法之后，应该问自己三个问题：它是否是对称的、传递的、一致的？（**自反性、对称性、传递性、一致性、非空性**）

  告诫：

  - 覆盖equals时总要覆盖hashCode
  - 不要将equals声明中的Object对象替换为其他的类型

- 覆盖equals时总要覆盖hashCode

  如果equals时不覆盖hashCode：equals时hashCode不同，散列到不同的散列桶中找不到key

  简单的解决方法：

  1. 把某个非零的常数值，比如17，保存在一个名为result的int类型的变量中。

  2. 对于对象中每个关键域f（指equals方法中涉及的每个域），完成以下步骤：

    a. 为该域计算int类型的散列码c：

    i.boolean类型: `f?1:0`

    ii.byte, char, short, int: `(int)f`

    iii. long: `(int)(f^(f>>>32))`

    iv. float: `Float.floatToIntBits(f)`

    v. double: `Double.doubleToLongBits(f)`，然后按照步骤2.a.iii为得到的long类型值计算散列值

    vi. 对象引用：递归调用hashCode

    vii. 数组：把每一个元素当作单独的域来处理

    b. 按照公式把步骤2.a中计算得到的散列码c合并到result中：
    
    `result=31*result+c`

    注意：
    
    - 如果一个类是不可变的，并且计算散列码的开销也比较大，就应该开率把散列码缓存在对象内部。
    - 不要试图从散列码计算中排除掉一个对象的关键部分来提高性能。
