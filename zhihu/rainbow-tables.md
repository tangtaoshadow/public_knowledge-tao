---
rainbow tables
唐涛 杭州电子科技大学 管理学院 2016级

---





[TOC]








# rainbow tables

**Repository:** [Author: GitHub](https://github.com/tangtaoshadow/)  [Gitee](https://gitee.com/tangtao2099/ant-design-tao)

**Introduction:**  

**Author:**[杭州电子科技大学](http://www.hdu.edu.cn/)  2016级管理学院 工商管理 唐涛 [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)

**CreateTime:**`2020-2-6 18:13:54`

**UpdateTime:**`2020-2-7 16:22:29`

**Copyright:**  唐涛 [HOME | 首页](https://www.promiselee.cn/tao) **©**  ***2020***  [<img alt="tangtao" style="width:35px;display:inline;" src="https://www.promiselee.cn/share_static/files/github/tao-logo.svg"/>](https://www.promiselee.cn/tao)  [<img style="width:25px;display:inline;margin-bottom:5px;" alt="github" src="https://www.promiselee.cn/share_static/files/github/github-logo.svg"/>](https://github.com/tangtaoshadow/spring-core)

**Email:**  <tangtao2099@outlook.com>

**Link:**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)   [GitHub](https://github.com/tangtaoshadow)  [Spring官网文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html)







# 彩虹表背后的概念(Concept)



## hash

在计算机系统中的密码不是直接存储为纯文本，而是使用加密进行哈希处理。`hash`函数是单向（`1-way`）函数，这意味着您无法解密哈希以查看密码的明文内容，无法解密❌。



当我们尝试登录系统时，输入的密码会使用加密“散列 `hashed`”，因此实际密码永远不会以明文形式通过通信线路发送，这样可以防止窃听者拦截密码。密码的散列通常看起来像一堆垃圾，并且长度通常与原始密码不同。您的密码可能是`tangtao`，但密码的哈希值看起来像是`C5F9781BED703A03AB07BA0A3924855C`。





## MD5 

**MD5**的全称是**Message-Digest Algorithm 5**（信息摘要算法），MD5的典型应用是对一段信息（**Message**）产生信息摘要（**Message-Digest**），以防止被篡改。下表是字符串 `tangtao` 经过`md5`的加密结果：

| 原始字符串    | `tangtao`                          |
| ------------- | ---------------------------------- |
| **16位 小写** | `ed703a03ab07ba0a`                 |
| **16位 大写** | `ED703A03AB07BA0A`                 |
| **32位 小写** | `c5f9781bed703a03ab07ba0a3924855c` |
| **32位 大写** | `C5F9781BED703A03AB07BA0A3924855C` |





## 如何破解密码？

为了验证用户，系统采用客户端计算机上密码哈希功能创建的哈希值，并将其与服务器上表中存储的哈希值进行比较。如果哈希匹配，则对用户进行身份验证并授予访问权限。



密码破解程序的工作方式与上述的登录过程类似，

第一种方式：破解程序首先获取纯文本密码（例如 **字典**），然后通过哈希算法（例如`MD5`）运行它们，然后将哈希输出结果与最初存储的哈希进行比较。如果找到匹配项，则程序已经破解了密码。这也就是常说的暴力破解💫，此过程可能需要很长时间。

第二种种方式：事先计算出所有字典(**明文**)的`hash`结果(**密文**)，然后将其存入数据库中，通过建立**索引**等方式，当我们拿到一个**密文**(`hash`)时，我们去数据库通过索引将其快速查出对应的**明文**，如果数据库中不存在，则破解失败。

第三种方式：第一种方式破解时间很长很长，第二种方式需要巨大的存储空间，而且我们无法穷举所有可能的结果，因此我们就结合了上述的**暴力破解**和**查表破解**两种方式诞生了**彩虹表**。黑客可以购买预先计算的`Rainbow Tables`，以破解易受攻击的操作系统（例如**Windows XP**，**Vista**，**Windows 7**和使用`MD5`和`SHA1`作为其密码哈希机制的应用程序）的密码（许多`Web`应用程序开发人员仍在使用这些哈希算法❌）。



## 破解密码



### 生成哈希链

我们先构建一张这样的链表：

-   第一步：先根据明文计算出其哈希值，此过程称为**`H`**函数，

-   第二步：取计算出的`hash`值的前`n`(比如 `n`为 8)位得到新的字符串，将新的字符串作为明文，此过程称为**`R`**函数

-   第三步：再次重复上面两步。





>   🛑  当面对要破解的哈希函数`H`，首先要定义一个函数`R`(**此函数自己定义，只要满足该函数的定义域和值域与哈希函数相反**)，通过该函数可以将哈希值映射为一个与原文相同格式的值。需要强调的是，由于哈希函数是不可逆的，所以对于密文进行`R`运算几乎不可能得到明文原文。



#### 举例

首先，我们获取字符串并将其通过`MD5`哈希函数传递

```js
MD5(tangtao) = c5f9781bed703a03ab07ba0a3924855c
```



刚才说了**`R`**函数自己定义，那么我们自定义一个`R`函数，它就是截取`H`函数值的前**8**个字符，这样我们通过R运算，等到了一个新的结果，然后我们重新哈希它

```js
MD5(c5f9781b) = 39856f5c517195b517b44ff6f19f5305
```



重复此过程，直到输出链中有**足够**的散列为止。比如现在得到了下面这条链



![tangtao](https://www.promiselee.cn/share_static/files/github/private-knowledge/java/rainbow-tables-tangtao-hash-2020-2-6-191623.png)



这表示一个链，该链从第一个纯文本开始，到最后一个哈希结束。



#### <a name="保存链表_哈希链集_20200207151620">保存链表:哈希链集</a>

>   🛑  上述 `H`,`R`重复次数记为`k`，`k`值应该适当，`k`太大出现[重复](#哈希链重复_202002062038)子链(下面解释)的可能性更大；而太小会让我们生成更多的链条，使得存储空间变得很大。

我们只存储链的开始位置🔹和结束位置🔸，通过这种方式我们可以生成很多很多链条，这些哈希链条就构成**哈希链集**。

获得足够的链后，我们将它们存储在表中，供下面破解使用😼。





### 开始破解



#### 第一次查询哈希链

现在我们得到了一个密文，比如是`X`，那么密文就是`hash`值，所以我们第一次使用`R`函数对其进行运算，得到了`Y=R(X)`；

还记得上面的哈希链表吗，它每条链只存储了开始和结尾位置，开始位置是明文，结尾位置是最后一次`R`函数运算的结果；

因此选择我们把刚才得到的`Y`与哈希链中的结尾位置进行比对，接下来就会有出现比对成功和比对失败。



#### 比对成功

如果发现匹配成功了某条链的结尾位置，由于不可逆，所以我们就开始从这条链的开始位置开始重复**H**，**R**运算，

也就是 **开始位置**→**H1**→**R1**→**H2**→**R2**→**H3**→**R3**→...💦

这样在一直计算下去，就会存在 `Rn-1→Hn=X→Rn=Y=R(X)`，也就是`Rn-1`经过**H**运算得到的`Hn`就是`X`，`X`是什么😮**(+_+)?**，它是我们已知的密文，现在我们得到了一个这样的等式

**`Rn-1→Hn=X→Rn=Y=R(X)`**

也就是**`Rn-1`**经过了**H**运算，得到了**`Hn`**，而这个**`Hn`**就是**`X`**，这就是我们要找的明文😨，因此破解成功👀。



#### 比对失败

这个时候我们就要转换思路❓，

如果上面比对成功意味着什么💞，也就是 **`Rn-1→Hn=X→Rn=Y=R(X)`**，这不就是说明了**`Rn-1`**就是明文吗？

那**`Rn-1`**代表了什么❗ 

它代表了它是从开始位置一直计算了**`n-1`**次**H**运算和**R**运算，也就是说我们要找的答案(**明文**)藏在了表的**`Rn-1`**这个位置，也就是说我们刚才没有比对成功只说明了明文不在**`Rn-1`**上，

那么我们继续假设明文在**`Rn-1`**的前一个位置**`Rn-2`**上，这个时候我们需要将第一次R运算的结果**`Y=R(X)`**进行一次**H**运算，再将运算的结果再进行一次**R**运算，得到了**`Y2`**，再将**`Y2`**与所有哈希链的结尾位置进行比对，又会出现比对成功和比对失败；

如果这次比对失败💤，意味着明文不出现在 **`Rn-2`**上，因此我们在重复**H**运算和**R**运算，再比对，一直比对下去；



#### 最终结果

重复上述过程；

如果最终出现了和某条哈希链比对成功，就说明明文藏在了这条哈希链中，

如果一直假设它的位置到**`R1`**还是比对失败，说明我们查找了现在存储的所有哈希链但还是未找到结果，也就是要找的明文不在我们存储的哈希链集中，最终结果比对失败💥。



#### 举例

以下面这张表为例：

![tangtao](https://www.promiselee.cn/share_static/files/github/private-knowledge/java/rainbow-tables-tangtao-hash-2020-2-6-191623.png)



假设我们现在有密文 **`39856f5c...`**

我们对其进行一次**`R`**运算，得到了 **`39856f5c`**

发现其比对了上面这条链的结尾位置，这说明了明文存在**链的倒数第三个位置**，

因此我们就从这条链的开始位置开始计算，

就会得到 `c5f9781b`→`39856f5c...`，那么明文就找到了，它就是 `c5f9781b`



假设现在我们有密文 **`c5f9781b...`**

我们对其进行一次**R**运算，得到了 `c5f9781b` ，

没有比对中链的结尾位置，这说明了明文不存在**链的倒数第三个位置**

再对 `c5f9781b`进行一次**H**运算和一次**R**运算，得到了 `39856f5c`，

发现比对成功了上面的这条链的结尾位置，这说明了明文不存在**链的倒数第五个位置**，因此我们就从上面这条链的开始位置开始计算，

就会得到 `tangtao`→`c5f9781b...`，那么明文就找到了，他就是 `tangtao`



#### 总结

>   🟢 其实我们的明文只会存在经过**R**运算后的结果上，且最后一次R运算的结果不算。
>
>   上面所做的工作实际上可以理解成我们假设明文最先存在链表倒数第三个位置上，如果不对，再假设它存在倒数第五个位置上，一直下去，最后到假设到链表的第二个位置上，如果还是没有比对成功，那只能说明明文不在我们存储的哈希链集中。



对于一个长度为`k`的预计算的哈希链集，我们破解的次数不超过`k`，哈希链集中的每条链只保存开始节点和结束节点，这样我们就既比暴力破解的用时快得多，同时我们又比查表需要的存储空间少得多。







# 彩虹表



## <a name="哈希链重复_202002062038">哈希链重复</a>

针对上面生成的哈希链表，会因为**R**函数生成重复链的问题。

🔆重复链的生成与**R**函数的选取有关，比如我们上面定义了一个**R**函数：取**H**函数运算结果的前**8**位，我们假设`aaaabbbb`的`hash`值的前8位刚好也是 `c5f9781b`，那么就会出现重复链的问题，



![tangtao](https://www.promiselee.cn/share_static/files/github/private-knowledge/java/rainbow-tables-tangtao-hash-repeat-2020-2-6-204303.png)



如果链出现重复，那么我们两条链可以破解的明文数量将会大打折扣。而且我们通常的哈希链会非常多非常多，因此重复的情况会变得非常严重🤡。



## 解决方式

针对上面的重复问题，我们想了一个办法，对于长度为k的哈希链，第一次**R**运算使用**R1**函数运算，第二次**R**运算使用**R2**函数运算，...🤖，

这样我们定义并使用`k`个**R**函数，**R1**，**R2**，**R3**,...，**Rk**，其中每个**R**函数都不相同，

这样做能彻底解决吗🦴？并不能，我们使用了更多不同的**R**函数，将重复的可能性降低，





比如有两条链重复了，如果这种重复出现在完全相同的链位置，假设他们第一次重复是出现都是出现在第2次**R**运算后，那么后面的链就会完全重复，也就是



`aaaa` **--H-->** `gdgf` **--R1-->** `gtsa` **--H-->** `mkba` **--R2-->** `grws` 💢 **--H-->** `gtwq` **--R3-->** `rtva`



`bbbb` **--H-->** `btga` **--R1-->** `plcd` **--H-->** `mwqp` **--R2-->** `grws` 💢 **--H-->** `gtwq` **--R3-->** `rtva`



>   🟢 如果真的发生了链重复的情况，那么链的结尾位置会使你相同的，因此我们可以及时发现。



如果重复的不是在链的同一个位置，假设是一条链的**R2**的运算结果与另一条链的**R1**运算结果相同了，那么下一个**H**运算就会是相同的，但是这两条链因为重复位置不同，所以对相同的**H**运算结果是进行不同的**R**运算，所以就很可能避免了下一个**R3**与另一条链的**R2**重复，将可能是下面的情况



`aaaa` **--H-->** `gdgf` **--R1-->** `gtsa`       **--H-->** `mkba` **--R2-->** `grws` 💢 **--H-->** `gtwq` **--R3-->** `rtva`



`bbbb` **--H-->** `btga` **--R1-->** `grws` 💢 **--H-->** `gtwq` **--R2-->** `ppnn`       **--H-->** `bgre` **--R3-->** `abcd`



上面所用的不同的**R**函数，在不同的运算位置使用不同的**R**函数，就像彩虹由内而外的不同位置上显示出不同的颜色一样💙💚💜🤎🧡💗

![tangtao](https://www.promiselee.cn/share_static/files/github/private-knowledge/java/rainbow-tables-tangtao-rainbow-tao-2020-2-6-214404.png)





## 彩虹表概念

**彩虹表**(`rainbow table`)是一个用于[加密散列函数](https://zh.wikipedia.org/wiki/加密散列函数)逆运算的预先计算好的[表](https://zh.wikipedia.org/wiki/查找表)，常用于破解加密过的密码散列。 查找表常常用于包含有限字符固定长度[纯文本](https://zh.wikipedia.org/wiki/纯文本)[密码](https://zh.wikipedia.org/wiki/密码)的加密。



这是[以空间换时间](https://zh.wikipedia.org/wiki/以空間換時間)的典型实践，在每一次尝试都计算的暴力破解中使用更少的计算能力和更多的储存空间，但却比简单的每个输入一条散列的翻查表使用更少的储存空间和更多的计算性能。使用[加盐](https://zh.wikipedia.org/wiki/盐_(密码学))的[KDF函数](https://zh.wikipedia.org/w/index.php?title=KDF函数&action=edit&redlink=1)可以使这种攻击难以实现。



>   🛑 由于一个以上的文本可以产生相同的哈希值，因此只要能产生相同的哈希值，知道原始密码的真实性就无关紧要。







## 总结

### <a name="彩虹表计算思路_202002071508">彩虹表计算思路</a>

彩虹表计算和哈希链集计算类似，

-   假设要明文位于彩虹表中的某一链条的 `R(k-1)`位置，然后对其进行`Rk`运算，查找彩虹表所有链表的结束位置是否有匹配的结果，如果某条链匹配，则从该链开始位置进行计算；
-   如果未匹配，说明明文不在 `R(k-1)` 位置，继续假设它在 `R(k-2)` 位置，因为使用了`k`个不同的`R`函数运算，所以需要对密文重新进行  `R(k-1)`，`H`，`Rk` 三次运算，然后将计算结果继续在哈希链结束位置进行查找匹配。
-   如果一直假设到链的最开始位置，称为`R0`，还是没有找到结果，说明明文不在此彩虹表中❗



### 彩虹表时间&空间

根据[彩虹表计算计算思路](#彩虹表计算思路_202002071508)，最多计算次数为 

**`R `运算**

```js
1+2+3+...+(k-1)=k(k-1)/2
```

**`H` 运算**

```js
1+2+3+...+(k-1)=k(k-2)/2
```

由于使用了不同的`R`函数，因此计算彩虹表的代价大于[哈希链集](#保存链表_哈希链集_20200207151620)。

**`K`的意义**

如果`k`越大，可以解释的明文数量越多，这样存储的空间就会更少，但是计算彩虹表时间就会增加。







## 获取彩虹表

[project-rainbowcrack.com](https://project-rainbowcrack.com/table.htm)

[freerainbowtables.com](https://freerainbowtables.com/)





## 防御彩虹表攻击



我们注意到彩虹表使用了一个相同的`H`函数，如果我们使用不同的`H`函数，这样它的彩虹表将必须使用相同的`H`函数才能有效，这种方法称为加盐。

通过加盐可以轻松地防止彩虹表攻击，该技术是将随机数据与纯文本一起传递到哈希函数中，这样可以确保每个密码都有一个唯一的生成的哈希，因此可以防止彩虹表攻击，该攻击基于一个原则，即多个文本可以具有相同的哈希值。



### 举例

在`spring`中，我们通常使用 `BCryptPasswordEncoder` 类对密钥进行加密，

这里我们对同一个明文 `tangtao`，对其进行**5**次加密，

```java
package com.tao.bluefairy;

import com.tao.bluefairy.utils.JwtTokenUtil;
import java.util.ArrayList;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@SpringBootTest
class BlueFairyApplicationTests {

  @Test
  void contextLoads() {

    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    //  存储密钥数组
    ArrayList<String> passwordArray = new ArrayList<String>();
    for (int i = 0; i < 5; i++) {
      // 将循环生成的密钥添加到数组中
      passwordArray.add(passwordEncoder.encode("tangtao"));
    }
    System.out.println("======  加密的密钥  =====");
    passwordArray.stream().forEach(System.out::println);
    System.out.println("======  验证密钥  =====");
    passwordArray.stream()
        .forEach(
            p -> {
              //  验证不同的密文是否匹配相同的明文
              boolean res = passwordEncoder.matches("tangtao", p);
              String s = res ? " 匹配成功" : " 匹配失败";
              System.out.println("当前密钥: " + p + s);
            });
    System.out.println("=======  结束  =======");
  }

}

```



通过`debug`，我们可以看到生成了五组密文都不相同，

![tangtao](https://www.promiselee.cn/share_static/files/github/private-knowledge/java/rainbow-tables-spring-bcryptpasswordencoder-debuger-20200207160505.png)



然后我们在对系统的明文与生成的五个不同的密文进行匹配，查看每次是否匹配，下面是控制台输出

```sh
======  加密的密钥  =====
$2a$10$LNukPI1pXuINtCoHgcOII.7HSPfLLDtCPV4aC33Q4m1OIQHkeTdLO
$2a$10$qWd/4otrjjFAKHMqadgwqetJuJua8cBVHGhAt7zlAVtG9xp4WcbdS
$2a$10$2VoYIaAUi9J/BEmbXnz9euAgU0uPKvynBSDGscLg3UL/wXe6Ro8BW
$2a$10$lQxAKZvUarY6ODNUUwfn8OXMIGtkJYfiDD8YFp6pck8d2tkioCRSO
$2a$10$AuavMiVStvbDRzyXe4uBWOg7DApSLwk9ClatULle.VrDrPwX1rLU6
======  验证密钥  =====
当前密钥: $2a$10$LNukPI1pXuINtCoHgcOII.7HSPfLLDtCPV4aC33Q4m1OIQHkeTdLO 匹配成功
当前密钥: $2a$10$qWd/4otrjjFAKHMqadgwqetJuJua8cBVHGhAt7zlAVtG9xp4WcbdS 匹配成功
当前密钥: $2a$10$2VoYIaAUi9J/BEmbXnz9euAgU0uPKvynBSDGscLg3UL/wXe6Ro8BW 匹配成功
当前密钥: $2a$10$lQxAKZvUarY6ODNUUwfn8OXMIGtkJYfiDD8YFp6pck8d2tkioCRSO 匹配成功
当前密钥: $2a$10$AuavMiVStvbDRzyXe4uBWOg7DApSLwk9ClatULle.VrDrPwX1rLU6 匹配成功

```



这样我们发现对于相同的明文，每次加密的结果都不相同，但是相同的明文与上面的密文都匹配，因此我们可以舍弃明文，将密文存入数据库中将是一种更安全的方式，

通常我们的密码是在前端输入的，因此我们可以将用户输入的敏感数据加密传输到后端，后端将密码与数据库中的结果进行比对，实现用户验证。





### 建议

在密码哈希函数中尽量不使用`MD5`或`SHA1`。`MD5`和`SHA1`是**过时**的密码哈希算法，大多数用于破解密码的`Rainbow`表都是使用这些哈希方法针对应用程序和系统而构建的，应该考虑使用更现代的哈希方法，例如`SHA2`。







------

**原文链接**：[rainbow-tables](https://www.promiselee.cn/share_static/files/public_knowledge/zhihu/rainbow-tables.html)

**Author**：[TangTao](https://www.promiselee.cn/tao)  [HOME | 首页](https://www.promiselee.cn/tao) **©**  ***2020***  [<img alt="tangtao" style="width:35px;display:inline;" src="https://www.promiselee.cn/share_static/files/github/tao-logo.svg"/>](https://www.promiselee.cn/tao)

**CreateTime:**`2020-2-6 18:13:54`

**UpdateTime:**`2020-2-7 16:22:29`









[toc]

