# Java 中的 `::` 使用方式

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



原文链接：[GitHub](https://github.com/tangtaoshadow/public_knowledge-tao/blob/master/zhihu/%E5%AE%89%E8%A3%85Kubernetes%E9%9B%86%E7%BE%A4.md) Web