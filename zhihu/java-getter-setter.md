## Java  Getter和Setter

**Author：**[杭州电子科技大学](https://link.zhihu.com/?target=http%3A//www.hdu.edu.cn/) 2016级管理学院 工商管理 唐涛 [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)

**CreateTime：**`2019-10-5 15:49:43`

**UpdateTime：**  `2019-10-5 22:12:57`

**Copyright:**  *唐涛* *2019* © [HOME](https://link.zhihu.com/?target=https%3A//www.promiselee.cn/tao)

**Email：**[tangtao2099@outlook.com](mailto:tangtao2099@outlook.com)

**Link：** [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)  [GitHub](https://link.zhihu.com/?target=https%3A//github.com/tangtaoshadow)  [Gitee](https://gitee.com/tangtao_2099)



**`Getter`和`setter`在Java中被广泛使用。这看似简单，但并非每个程序员都正确理解和实现这种方法。因此，在本文中，我想深入讨论Java中的`getter`和`setter`方法.**



## 1.什么是`Getter`和`Setter`？

在Java中，getter和setter是两种常规方法，用于检索和更新变量的值。

以下代码是带有私有变量和几个getter / setter方法的简单类的示例：



```java
public class SimpleGetterAndSetter {
    private int number;
    public int getNumber() {
        return this.number;
    }
    public void setNumber(int num) {
        this.number = num;
    }
}
```



该类声明一个私有变量number。由于number是私有的，因此此类外部的代码无法直接访问该变量，如下所示：

```java
SimpleGetterAndSetter obj = new SimpleGetterAndSetter();
obj.number = 10;    // compile error, since number is private
int num = obj.number; // same as above
```



相反，外部代码必须调用getter   `getNumber()`和setter，   `setNumber()`以读取或更新变量，例如：

```java
SimpleGetterAndSetter obj = new SimpleGetterAndSetter();
obj.setNumber(10);  // OK
int num = obj.getNumber();  // fine
```



因此，`Setter`是一种更新变量值的方法。`Getter`是一种读取变量值的方法。`Getter`和`setter` 在Java 中也称为**访问器**和**更改器**。



## 2.为什么我们需要`Getter`和`Setter`？

通过使用getter和setter，程序员可以控制如何以适当的方式访问和更新其重要变量，例如在指定范围内更改变量的值。考虑以下setter方法的代码：

```java

public void setNumber(int num) {
    if (num < 10 || num > 100) {
        throw new IllegalArgumentException();
    }
    this.number = num;
}
```



这样可以确保将数字的值始终设置在10到100之间。假定可以直接更新变量号，则调用者可以为其设置任意值：

```java
obj.number = 3;
```



这就违反了该变量从10到100范围内的值的约束。当然，我们不希望这种情况发生。因此，将变量号隐藏为私有，然后使用设置器即可解决。

另一方面，getter方法是外界读取变量值的唯一方法：



```java
public int getNumber() {
    return this.number;
}
```



下图说明了这种情况：

![](http://cdn.promiselee.cn/share_static/files/java/java-getter-and-setter-2019-10-5%20155539.png)

setter和getter方法可保护变量的值免受外界（调用方代码）的意外更改。

当变量由private修饰符隐藏  并且只能通过getter和setter进行访问时，将被封装。封装是面向对象编程（OOP）的基本原理之一，因此实现getter和setter是在程序代码中强制执行封装的方法之一。

诸如Hibernate，Spring和  Struts之类的某些框架 可以检查信息或通过getter和setter注入其实用程序代码。因此，在将代码与此类框架集成时，必须提供getter和setter方法。



## 3. Getter和Setter的命名约定

setter和getter的命名方案应遵循  *Java Bean命名约定，*如   `getXxx()` 和   `setXxx()`，其中  `Xxx` 变量的名称。例如，使用以下变量名：

```java
private String name;
```



合适的setter和getter将是：

```java
public void setName(String name) { }

public String getName() { }
```



如果变量的类型为`boolean`，则`getter`的名称可以为  `isXXX()` 或   `getXXX()`，但首选使用前者命名。例如：

```java
private boolean single;

public String isSingle() { }
```



下表显示了符合命名约定的getter和setter的一些示例：



| **变量声明**     | **Getter method**                       | **Setter method**                 |
| ---------------- | --------------------------------------- | --------------------------------- |
| int quantity     | `int getQuantity()`                     | `void setQuantity(int qty)`       |
| string firstName | `String getFirstName()`                 | `void setFirstName(String fname)` |
| Date birthday    | `Date getBirthday()`                    | `void setBirthday(Date bornDate)` |
| boolean rich     | `boolean isRich()`  `boolean getRich()` | `void setRich(Boolean rich)`      |





## 4. 使用Getter和Setter时的常见错误

### **错误＃1：您同时拥有setter和getter，但在限制较少的范围内声明了变量。**

分析以下代码片段：



```java

public String firstName;
public void setFirstName(String fname) {
    this.firstName = fname;
}
public String getFirstName() {
    return this.firstName;
}
```



将该变量  firstName 声明为public，因此可以直接使用点（.）运算符对其进行访问，从而使setter和getter无效。这种情况的解决方法是使用更多受限制的访问修饰符，例如protected和private：

```java

private String firstName;
```



### **错误2：直接在`Setter`中分配对象引用**

```java

private int[] scores;
public void setScores(int[] scr) {
    this.scores = scr;
}
```



下面的代码演示了此问题：

```java
int[] myScores = {5, 5, 4, 3, 2, 4};
setScores(myScores);
displayScores();   
myScores[1] = 1;
displayScores();
```

整数数组  `myScores`初始化为6个值（第1行），并将该数组传递给  `setScores()` 方法（第2行）。该方法  `displayScores()` 只是从数组中打印出所有分数：



第3行将产生以下输出：

```bash
5 5 4 3 2 4
```



这些都是`myScores` 数组的所有元素  。现在，在第4行中，我们可以 按如下所示修改数组中第二个元素  的值`myScores`：

```java
myScores[1] = 1;
```



由于第4行的赋值，第二个元素的值从5更改为1。为什么重要？因为这意味着可以在setter方法范围之外修改数据，这破坏了setter的封装目的。为什么会这样呢？让我们`setScores()`  再次看一下该 方法：

```java

public void setScores(int[] scr) {
    this.scores = scr;
}
```



成员变量分数直接分配给方法的参数变量  `scr` 。这意味着两个变量都引用内存中的同一对象  `myScores` 数组对象。因此，对`scores` 或  `myScores` 变量所做的更改  实际上是在同一对象上进行的。

这种情况的一种解决方法是将元素从`scr` 数组复制  到  `scores` 数组中。setter的修改后的版本如下所示：

```java

public void setScores(int[] scr) {
    this.scores = new int[scr.length];
    System.arraycopy(scr, 0, this.scores, 0, scr.length);
}
```



有什么不同？好吧，成员变量`scores` 不再引用变量  所引用的对象  `scr` 。而是将数组  `scores` 初始化为大小等于array大小的新数组  `scr`。然后，使用   方法将所有元素从数组复制  `scr` 到array  。`scores``System.arraycopy()`

再次运行以下示例，它将为我们提供以下输出：



```java

5  5  4  3  2  4
5  5  4  3  2  4
```



现在，两次调用  `displayScores()` 产生相同的输出。这意味着该数组  `scores` 是独立的，并且与`scr` 传递给setter 的数组不同  ，因此我们具有以下分配：

```java
myScores[1] = 1;
```



这不会影响数组  `scores`。

因此，经验法则是：如果将对象引用传递给setter方法，则不要将该引用直接复制到内部变量中。相反，您应该找到一些将传递的对象的值复制到内部对象的方法，例如，使用该`System.arraycopy()`  方法将元素从一个数组复制到另一个数组 。



### **错误3：直接在`getter`中返回对象引用**

```java
private int[] scores;
public int[] getScores() {
    return this.scores;
}
```



然后看下面的代码片段：

```java
int[] myScores = {5, 5, 4, 3, 2, 4};
setScores(myScores);
displayScores();
int[] copyScores = getScores();
copyScores[1] = 1;
displayScores();
```



输出

```bash
5  5  4  3  2  4
5  1  4  3  2  4
```



如您所见，该数组的第二个元素  scores 在setter的第5行中进行了修改。由于getter方法直接返回内部变量score的引用，因此外部代码可以获得该引用并更改内部对象。

这种情况的解决方法是，我们应该返回对象的副本，而不是直接在getter中返回引用。这样，外部代码只能获取副本，而不能获取内部对象。因此，我们对上述getter进行如下修改：

```java
public int[] getScores() {
    int[] copy = new int[this.scores.length];
    System.arraycopy(this.scores, 0, copy, 0, copy.length);
    return copy;
}

```



因此，经验法则是：不要在getter方法中返回原始对象的引用。相反，它应该返回原始对象的副本。



## 5.实现原始类型的获取器和设置器

随着原始类型（`int`，  `float`，  `double`，  `boolean`，  `char`...），你可以自由地直接在assign/返回值/getter,因为Java拷贝一个原始的另一个而不是复制对象引用的值。因此，可以轻松避免错误＃2和＃3

例如，以下代码是安全的，因为setter和getter涉及以下原始类型  `float`：

```java
private float amount;
public void setAmount(float amount) {
    this.amount = amount;
}
public float getAmount() {
    return this.amount;
}
```



因此，对于原始类型，没有正确实现getter和setter的特殊技巧。



## 6.实现常见对象类型的Getter和setter

### **字符串对象的getter和setter：**

`String`是一种对象类型，但是是不可变的，这意味着一旦创建了String对象，就无法更改其String文字。换句话说，对该String对象的每次更改都会导致新创建一个String对象。因此，像原始类型一样，您可以安全地为String变量实现getter和setter，如下所示：

```java

private String address;
public void setAddress(String addr) {
    this.address = addr;
}
public String getAddress() {
    return this.address;
}
```



### **日期对象的获取器和设置器：**

的  `java.util.Date` 类实现  `clone()` 从方法  `Object` 类。该方法  `clone()` 返回对象的副本，因此我们可以将其用于getter和setter，如以下示例所示：

```java
private Date birthDate;
public void setBirthDate(Date date) {
    this.birthDate = (Date) date.clone();
}
public Date getBirthDate() {
    return (Date) this.birthDate.clone();
}
```



该  clone() 方法返回一个  Object，因此我们必须将其Date 强制转换为 Date 类型。





## 7.实现集合类型的`getter`和`setter`

如错误2和错误3所述，使用这样的setter和getter方法是不好的：

```java
private List<String> listTitles;
public void setListTitles(List<String> titles) {
    this.listTitles = titles;
}
public List<String> getListTitles() {
    return this.listTitles;
}
```



分析一下程序

```java
import java.util.*;
public class CollectionGetterSetter {
    private List<String> listTitles;
    public void setListTitles(List<String> titles) {
this.listTitles = titles;
    }
    public List<String> getListTitles() {
        return this.listTitles;
    }
    public static void main(String[] args) {
        CollectionGetterSetter app = new CollectionGetterSetter();
        List<String> titles1 = new ArrayList();
        titles1.add("Name");
        titles1.add("Address");
        titles1.add("Email");
        titles1.add("Job");
        app.setListTitles(titles1);
        System.out.println("Titles 1: " + titles1);
        titles1.set(2, "Habilitation");
        List<String> titles2 = app.getListTitles();
        System.out.println("Titles 2: " + titles2);
        titles2.set(0, "Full name");
        List<String> titles3 = app.getListTitles();
        System.out.println("Titles 3: " + titles3);
    }
}
```



根据实现getter和setter的规则，这三个  `System.out.println()` 语句应产生相同的结果。但是，运行上述程序时，它将产生以下输出：

```bash
Titles 1: [Name, Address, Email, Job]
Titles 2: [Name, Address, Habilitation, Job]
Titles 3: [Full name, Address, Habilitation, Job]
```



对于字符串集合，一种解决方案是使用将另一个集合作为参数的构造函数。例如，我们可以如下更改上述getter和setter的代码：

```java
public void setListTitles(List<String> titles) {
    this.listTitles = new ArrayList<String>(titles);
}

public List<String> getListTitles() {
    return new ArrayList<String>(this.listTitles);   
}
```



重新编译并运行  `CollectionGetterSetter` 程序；它将产生所需的输出：

```bash
Titles 1: [Name, Address, Email, Job]
Titles 2: [Name, Address, Email, Job]
Titles 3: [Name, Address, Email, Job]
```



**注意**：

上面的构造方法仅适用于**字符串的**集合，但不适用于Collections **对象**。考虑下面的`Person` 对象集合示例  ：

```java
import java.util.*; 
public class CollectionGetterSetterObject { 
    private List<Person> listPeople; 
    public void setListPeople(List<Person> list) { 
        this.listPeople = new ArrayList<Person>(list); 
    } 
    public List<Person> getListPeople() { 
        return new ArrayList<Person>(this.listPeople); 
    } 
    public static void main(String[] args) { 
        CollectionGetterSetterObject app = new CollectionGetterSetterObject(); 
        List<Person> list1 = new ArrayList<Person>(); 
        list1.add(new Person("Peter")); 
        list1.add(new Person("Alice")); 
        list1.add(new Person("Mary")); 
        app.setListPeople(list1); 
        System.out.println("List 1: " + list1); 
        list1.get(2).setName("Maryland"); 
        List<Person> list2 = app.getListPeople(); 
        System.out.println("List 2: " + list2); 
        list1.get(0).setName("Peter Crouch"); 
        List<Person> list3 = app.getListPeople(); 
        System.out.println("List 3: " + list3); 
    } 
} 
class Person { 
    private String name; 
    public Person(String name) { 
        this.name = name; 
    } 
    public String getName() { 
        return this.name; 
    } 
    public void setName(String name) { 
        this.name = name; 
    } 
    public String toString() { 
        return this.name; 
    } 
}
```



运行时将产生以下输出：

```bash
List 1: [Peter, Alice, Mary]
List 2: [Peter, Alice, Maryland]
List 3: [Peter Crouch, Alice, Maryland]
```



因为与String不同，每当复制String对象时都将为其创建新对象，所以其他  `Object` 类型则不会。仅引用被复制，因此这就是两个Collection不同但它们包含相同对象的原因。换句话说，这是因为我们没有提供任何复制对象的方法。



查看Collection API；我们发现  `ArrayList`，  `HashMap`，  `HashSet`，等实现自己的  `clone()` 方法。这些方法返回浅表副本，这些浅表副本不会将元素从源Collection复制到目标。根据 类`clone()` 方法  的Javadoc  `ArrayList`：

`public Object clone()`

公共对象clone（）返回此ArrayList实例的浅表副本。（元素本身不会被复制）



因此，我们不能使用`clone()` 这些Collection类的 方法。

解决方案是`clone()` 为我们自己定义的对象（`Person` 上例中的类）实现该 方法 ，使用`clone()` 在`Person` 类中实现该方法， 如下所示：

```java
public Object clone() {
    Person aClone = new Person(this.name);
    return aClone;
}
```



 `listPeople` 的Setter修改如下：

```java
public void setListPeople(List<Person> list) {
    for (Person aPerson : list) {
        this.listPeople.add((Person) aPerson.clone());
    }
}
```



相应的`getter`被修改，如下所示：

```java
public List<Person> getListPeople() {
    List<Person> listReturn = new ArrayList<Person>();
    for (Person aPerson : this.listPeople) {
        listReturn.add((Person) aPerson.clone());
    }
    return listReturn;
}
```



结果是该类的新版本  `CollectionGetterSetterObject,` 如下所示：

```java
import java.util.*;
public class CollectionGetterSetterObject {
    private List<Person> listPeople = new ArrayList<Person>();
    public void setListPeople(List<Person> list) {
        for (Person aPerson : list) {
            this.listPeople.add((Person) aPerson.clone());
        }
    }
    public List<Person> getListPeople() {
        List<Person> listReturn = new ArrayList<Person>();
        for (Person aPerson : this.listPeople) {
            listReturn.add((Person) aPerson.clone());
        }
        return listReturn;
    }
    public static void main(String[] args) {
        CollectionGetterSetterObject app = new CollectionGetterSetterObject();
        List<Person> list1 = new ArrayList<Person>();
        list1.add(new Person("Peter"));
        list1.add(new Person("Alice"));
        list1.add(new Person("Mary"));
        app.setListPeople(list1);
        System.out.println("List 1: " + list1);
        list1.get(2).setName("Maryland");
        List<Person> list2 = app.getListPeople();
        System.out.println("List 2: " + list2);
        list1.get(0).setName("Peter Crouch");
        List<Person> list3 = app.getListPeople();
        System.out.println("List 3: " + list3);
    }
}

```



编译并运行新版本的  `CollectionGetterSetterObject`；它将产生所需的输出：

```bash
List 1: [Peter, Alice, Mary] 
List 2: [Peter, Alice, Mary] 
List 3: [Peter, Alice, Mary]
```



因此，实现Collection类型的getter和setter的关键点是：

- 对于String对象的Collection，由于String对象是不可变的，因此不需要任何特殊的调整。
- 对于对象的自定义类型的集合：
    - 实现`clone()` 自定义类型的 方法。
    - 对于setter，将克隆的项目从源集合添加到目标集合。
    - 对于getter，创建一个新的Collection，并将其返回。将原始集合中的克隆项添加到新集合中。





## 8.为自己的类型实现getter和setter

如果定义对象的自定义类型，则应`clone()` 为自己的类型实现该  方法。例如：

```java
class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    }
    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String toString() {
        return this.name;
    }
    public Object clone() {
        Person aClone = new Person(this.name);
        return aClone;
    }
}
```



如我们所见，该类  `Person` 实现其   `clone()` 方法以返回其自身的克隆版本。然后，setter方法应实现如下：

```java
public void setFriend(Person person) {
    this.friend = (Person) person.clone();
}
```



对于getter方法：

```java
public Person getFriend() {
    return (Person) this.friend.clone();
}
```



因此，为自定义对象类型实现getter和setter的规则是：

- 实现`clone()` 自定义类型的  方法。
- 从getter返回一个克隆的对象。
- 在设置器中分配一个克隆的对象。






## 结论

`Java` 中的 `getter` 和 `setter` 看起来很简单，但是如果天真地实现，可能会变得很危险。它甚至可能是导致您的代码行为异常的问题的根源。或更糟糕的是，可以通过隐式操纵获取器和设置器的参数并从中获取对象来轻易地利用您的程序。因此，请小心并考虑实施上述最佳实践。





## [原文链接](https://github.com/taoshadow/public_knowledge-tao/blob/master/zhihu/java-getter-setter.md)



