# Java 中的 `::` 使用方式

**Repository:** [Author: GitHub](https://github.com/tangtaoshadow/)

**Introduction:**

**Author:**[杭州电子科技大学](http://www.hdu.edu.cn/) [唐涛](https://www.promiselee.cn/tao)

**CreateTime:**`2020-6-22 03:30:04`

**UpdateTime:**`2020-6-22 04:27:18`

**Copyright:** 唐涛 [HOME | 首页](https://www.promiselee.cn/tao) **©** ***2020*** [![tangtao](https://camo.githubusercontent.com/1ecb34e23d6810047160b4808b8660b1522ec010/68747470733a2f2f7777772e70726f6d6973656c65652e636e2f73686172655f7374617469632f66696c65732f6769746875622f74616f2d6c6f676f2e737667)](https://www.promiselee.cn/tao) [![github](https://camo.githubusercontent.com/fd371f67043f6df54393053102867279bb669670/68747470733a2f2f7777772e70726f6d6973656c65652e636e2f73686172655f7374617469632f66696c65732f6769746875622f6769746875622d6c6f676f2e737667)](https://github.com/tangtaoshadow)

**Email:** [tangtao2099@outlook.com](mailto:tangtao2099@outlook.com)

**Link:** [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities) [GitHub](https://github.com/tangtaoshadow) [语雀](https://www.yuque.com/tangtao-frbji)

---



[toc]

> 使用范围
>
> - 静态方法
> - 实例方法
> - 构造函数

语法：`<Class name>::<method name>`



## 使用总结

### 💨输出流处理中的所有元素

##### Lambda expression:

```java
stream.forEach( s-> System.out.println(s));
```

##### 使用::运算符

```java
stream.forEach( System.out::println(s));
```

##### Code

```java
package tao;

import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonOperator
 * @Note
 * @CreateTime 2020-06-22 03:32:00
 * @UpdateTime 2020-06-22 03:32:00
 */


public class ColonOperator {
    public static void main(String[] args) {

        Supplier<Stream<String>> streamSupplier = () -> Stream.of("TangTao", "HangZhou", "China", "Stream");

        // Lambda
        streamSupplier.get().forEach(s -> System.out.println(s));

        System.out.println("====================");
        // ::
        streamSupplier.get().forEach(System.out::println);
    }

}

```



### 💨静态方法

```java
package tao;

import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonStatic
 * @Note
 * @CreateTime 2020-06-22 03:48:00
 * @UpdateTime 2020-06-22 03:48:00
 */


public class ColonStatic {
    static void func(String s) {
        System.out.println("static func: " + s);
    }

    public static void main(String[] args) {
        Supplier<Stream<String>> streamSupplier = () -> Stream.of("TangTao", "HangZhou", "China", "Stream");
        streamSupplier.get().forEach(ColonStatic::func);
    }
}


```

输出

```java
static func: TangTao
static func: HangZhou
static func: China
static func: Stream
```



### 💨实例方法

```java
package tao;

import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonInstance
 * @Note
 * @CreateTime 2020-6-22 03:54:29
 * @UpdateTime 2020-6-22 03:54:54
 */


public class ColonInstance {
    void func(String s) {
        System.out.println("instance func: " + s);
    }

    public static void main(String[] args) {
        Supplier<Stream<String>> streamSupplier = () -> Stream.of("TangTao", "HangZhou", "China", "Stream");
        streamSupplier.get().forEach((new ColonInstance())::func);
    }
}

```

输出

```java
instance func: TangTao
instance func: HangZhou
instance func: China
instance func: Stream
```



### 💨构造函数

```java
package tao;

import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonConstructor
 * @Note
 * @CreateTime 2020-06-22 04:19:00
 * @UpdateTime 2020-06-22 04:19:00
 */


public class ColonConstructor {

    ColonConstructor(String s) {
        System.out.println("constructor: " + s);
    }

    public static void main(String[] args) {
        Supplier<Stream<String>> streamSupplier = () -> Stream.of("TangTao", "HangZhou", "China", "Stream");

        streamSupplier.get().forEach(ColonConstructor::new);

    }
}

```

输出

```java
constructor: TangTao
constructor: HangZhou
constructor: China
constructor: Stream
```





### 💨Super 方法

语法

```
(super::methodName)
```

```java
package tao;

import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonSuper
 * @Note
 * @CreateTime 2020-06-22 04:02:00
 * @UpdateTime 2020-06-22 04:02:00
 */

class ColonSuperParent {
    String func(String s) {
        return "parent: " + s + "\n";
    }
}

public class ColonSuper extends ColonSuperParent {

    @Override
    String func(String s) {
        Function<String, String>
                func = super::func;
        String s1 = func.apply(s);
        s1 += "son: " + s + "\n";
        System.out.println(s1);
        return s1;
    }

    public static void main(String[] args) {
        Supplier<Stream<String>> streamSupplier = () -> Stream.of("TangTao", "HangZhou", "China", "Stream");

        streamSupplier.get().forEach(new ColonSuper()::func);
    }

}
```

输出

```java
parent: TangTao
son: TangTao

parent: HangZhou
son: HangZhou

parent: China
son: China

parent: Stream
son: Stream

```



### 💨特定类型的任意对象的实例方法

```java
package tao;

import java.util.function.Supplier;
import java.util.stream.Stream;

/**
 * @Author TangTao tangtao2099@outlook.com
 * @GitHub https://github.com/tangtaoshadow
 * @Website https://www.promiselee.cn/tao
 * @ZhiHu https://www.zhihu.com/people/tang-tao-24-36
 * @ClassName ColonArbitrary
 * @Note
 * @CreateTime 2020-06-22 04:11:00
 * @UpdateTime 2020-06-22 04:11:00
 */

class ArbitraryClass {

    private String s;

    ArbitraryClass(String s) {
        this.s = s;
    }

    public void func() {
        System.out.println("ArbitraryClass func: " + this.s);
    }
}

public class ColonArbitrary {

    public static void main(String[] args) {
        Supplier<Stream<ArbitraryClass>> streamSupplier = () -> Stream.of(
                new ArbitraryClass("TangTao"),
                new ArbitraryClass("HangZhou"),
                new ArbitraryClass("China"),
                new ArbitraryClass("Stream"));

        streamSupplier.get().forEach(ArbitraryClass::func);
    }

}

```

输出

```java
ArbitraryClass func: TangTao
ArbitraryClass func: HangZhou
ArbitraryClass func: China
ArbitraryClass func: Stream
```





---

> 原文链接：[GitHub](https://github.com/tangtaoshadow/public_knowledge-tao/blob/master/zhihu/Java%E4%B8%AD%E7%9A%84%E5%8F%8C%E5%86%92%E5%8F%B7%E8%BF%90%E7%AE%97%E7%AC%A6%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93.md) [Web](https://www.promiselee.cn/share_static/files/public_knowledge/zhihu/Java%E4%B8%AD%E5%8F%8C%E5%86%92%E5%8F%B7%E8%BF%90%E7%AE%97%E7%AC%A6%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93.html)
>
> 更新时间：`2020-6-22 05:03:14`