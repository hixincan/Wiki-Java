

# 参考

*官方文档是最好的学习资料，（chrome直接翻译成中文看！再结合阅读插件提升阅读体验）*



JDK：https://www.oracle.com/java/technologies/javase-downloads.html



JVM：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html 



在线api：https://www.oracle.com/cn/java/technologies/java-se-api-doc.html



官方：

https://docs.oracle.com/javase/specs/index.html

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html



# 更新日志

* JDK 5
    * 泛型
    
* JDK 7

* JDK 8

    官方：https://www.oracle.com/cn/java/technologies/javase/8-whats-new.html

    官方英文：https://www.oracle.com/java/technologies/javase/8-whats-new.html

* JDK 9

* JDK 10

* JDK 11

* JDK 12





# 运算符



## 位运算符

![image-20210605121751404](../../../../Inbox/Java.assets/image-20210605121751404.png)

^ 异或运算符



> 使用场景

**1、判断奇偶数**
我们可以利用 & 运算符的特性，来判断二进制数第一位是0还是1。

用if ((a & 1) == 0) 代替 if (a % 2 == 0)来判断a是不是偶数。

**2、交换两个数**
借助临时变量
通常我们交换两个数会使用一个临时变量来帮忙：

```java
int temp = a;
a = b;
b = temp;
```

借助累加和
如果考虑到内存，不希望使用临时变量（其实就是为了炫酷），可以这样实现：

```java
a = a + b;
b = a - b;
a = a - b;
```

从数学角度来分析一下（这个解释很违和，需要在一个频道才能看懂）：

- 第一步：a = a + b
- 第二步：b = a - b = (a + b) - b = a
- 第三步：a = a - b = (a + b) - b = (a + b) - a = b

使用 ^ 位运算符
如果想要更炫酷一点可以使用 ^ 来帮忙实现：
先来了解一下 ^ 的几个特性：

- a ^ a = 0
- a ^ 0 = a
- (a ^ b) ^ c = a ^ (b ^ c)

```java
a ^= b;
b ^= a;
a ^= b;
```

从数学角度来分析一下这个解释也很违和，需要在一个频道才能看懂）：

- 第一步：a = a ^ b
- 第二步：b = a ^ b = (a ^ b) ^ b = a ^ (b ^ b) = a ^ 0 = a
- 第三步：a = a ^ b = (a ^ b) ^ b = (a ^ a) ^ b = b ^ 0 = b


**3、取余**
其实取余算法和上面的判断奇偶数原理是一样的。

比如说我们要让a对16进行取余，那么就可以让 a & 15 得出来的结果就是余数。

可以看出15的二进制表示为：

`0000 0000 0000 0000 0000 0000 0000 1111`
所以 a & 15 返回值就是a二进制的最低四位，也就是 a & 15 = a / 16。

使用 & 来进行取余的算法比使用 / 效率高很多，虽然只能对2^n的数值进行取余计算，但是在JDK源码中也是经常被使用到，比如说HashMap中判断key在Hash桶中的位置。

**4、生成第一个大于a的满足2^n的数**

这个标题可能显得不那么容易理解，下面结合场景来解释一下。

在HashMap中我们需要生成一个Hash桶，用来存储键值对（或者说存储链表）。

当我们查询一个key的时候，会计算出这个key的hashCode，然后根据这个hashCode来计算出这个key在hash桶中的落点，由于上面介绍的使用 & 来取余效率比 / 效率高，所以HashMap中根据hashCode计算落点使用的是 & 来取余。

使用 & 取余有一个局限性就是除数必须是2^n，所以hash桶的size必须是2^n。

由于HashMap的构造器支持传入一个初始的hash桶size，所以HashMap需要对用户传入的size进行处理，生成一个第一个大于size的并且满足2^n的数。

这个算法的使用场景介绍完毕了，那么再来看一下算法实现：

循环判断

```java
public static final int tableSizeFor(int cap) {
    int size = 1;
    while (size < cap) {
        size *= 2;
    }
    return size;
}
```

| 运算符实现

```java
public static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

HashMap就是使用这个算法来修改用户使用构造器传进来的size的，这个算法是使用移位和或结合来实现的，性能上比循环判断要好。

**5、其他简单应用**
求相反数： ~a + 1
求绝对值： a >> 31 == 0 ? a : (~a + 1)













# 数据结构



0x 表示16进制，如 0x00000004

> 何时使用 16 进制？





## 循环

for/in（foreach）是 JDK 5 推出的语法糖

```java
public class Test01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        removeByFor(list);
        //removeByIter(list);
    }
	// 执行异常
    public static void removeByFor(List<String> list) {
        for (String item : list) {
            if ("2".equals(item)) {
                list.remove(item);
            }
        }
        System.out.println(list.toString());
    }
	// 执行正常
    public static void removeByIter(List<String> list) {
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if ("2".equals(item)) {
                iterator.remove();
            }
        }
        System.out.println(list.toString());
    }
}
```

![image-20210608210431965](../../../../Inbox/Java.assets/image-20210608210431965.png)	

==通过反编译来查看真实的代码==（去掉语法糖）

1. `javac TestForEach.java`
2. `javap -verbose TestForEach`

```java
public class Test01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        for (String item : list) {
            if ("2".equals(item)) {
                list.remove(item);
            }
        }
    }
}
```



编译成

```java
public class Test01 {
    public Test01() {
    }

    public static void main(String[] var0) {
        ArrayList var1 = new ArrayList();
        var1.add("1");
        var1.add("2");
        Iterator var2 = var1.iterator();

        while(var2.hasNext()) {
            String var3 = (String)var2.next();
            if ("2".equals(var3)) {
                var1.remove(var3);
            }
        }

    }
}
```

虽然ArrayList的foreach底层用迭代器实现，迭代器也支持在遍历集合的过程中进行删除元素的操作，但是删除的函数必须是迭代器的函数，而不是集合自有的函数。至于上述代码为什么会抛出`ConcurrentModificationException`异常，可以从ArrayList中的迭代器类找到答案。ArrayList中部分源码如下所示

```java
// ArrayList中删除函数源码
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// ArrayList中删除函数源码
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

// ArrayList中迭代器函数源码
public Iterator<E> iterator() {
    return new Itr();
}

// ArrayList中迭代器类源码
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            // 此处重新赋值，避免跳过下一个元素
            cursor = lastRet;
            lastRet = -1;
            // 此处重新赋值，避免下次调用next()函数时校验不通过抛出异常
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

当执行`for (Integer num : nums)`语句时，会先调用ArrayList中的iterator()接口生成迭代器，而在初始化`Itr`类时会先将ArrayList对象中的`modCount`变量赋给Itr对象中的`expectedModCount`变量，在调用迭代器的`next`函数时会先调用`checkForComodification`函数进行校验，如果`expectedModCount`和`modCount`不相等则会抛出`ConcurrentModificationException`异常。在正常的集合遍历中，一般情况下，我们只使用迭代器中`hasNext`和`next`函数，并不会改变`expectedModCount`或者`modCount`的值，所以不会有问题，但是如果在遍历中调用了集合中自有的删除函数操作，则会改变`modCount`的值，从而导致`expectedModCount`与`modCount`不相等，进而在调用迭代器的`next`函数时进行校验不通过产生`ConcurrentModificationException`异常。而在遍历中调用迭代器的删除函数操作，由于其内部会在删除元素后对`expectedModCount`重新赋值，使其与`modCount`值相等，所以在遍历集合的过程中使用迭代器的删除函数操作不会有问题。





# Java 体系

![image-20210616093224512](Java.assets/image-20210616093224512.png)









 



# 算法

目标是内存优化：减少计算





O(1)

O(n)



## 递归









# RPC

RPC 全称 Remote Procedure Call——远程过程调用，应用在分布式系统中。

参考：https://www.jianshu.com/p/0385f95250e7





# 安全

## CSRF

跨站请求伪造  Cross-site request forgery

攻击者利用后台有规律的接口



解决

* 客户端：对于数据库的修改请求，全部使用POST提交，禁止使用GET请求。
* 服务端：在表单里面添加一段隐藏的唯一的token(请求令牌)





# 重构

final





# 学习工具

查看字节码 `javap -c`

反编译工具推荐 jar， 命令 `jad -sjava`

==注意==在 powerShell 下执行命令

JD-GUI 显示的是最原始的 Java 源代码，而 ==JAD 显示的是更贴近事实的源代码==：+ 号操作符在编译的时候其实是会转成 StringBuilder 的。

这特别的关键，如果你想知道编译器的工作内容，就可以使用 JAD。就像 javap 一样，只不过更加的清晰明了，javap 一般人看不太懂，如下：

```java
// 源代码
public class StringTest {
    public static void main(String[] args) {
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += UUID.randomUUID().toString().substring(0, 1);
        }
        System.out.println(str);
    }
}
```



```java
// jad 反编译
public class StringTest
{

    public StringTest()
    {
    }

    public static void main(String args[])
    {
        String str = "";
        for(int i = 0; i < 10; i++)
            str = (new StringBuilder()).append(str).append(UUID.randomUUID().toString().substring(0, 1)).toString();

        System.out.println(str);
    }
}
```



```
PS D:\Repos\Study\JavaEx\out\production\java\com\string> javap -c .\StringTest.class
Compiled from "StringTest.java"
public class com.string.StringTest {
  public com.string.StringTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String
       2: astore_1
       3: iconst_0
       4: istore_2
       5: iload_2
       6: bipush        10
       8: if_icmpge     46
      11: new           #3                  // class java/lang/StringBuilder
      14: dup
      15: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      18: aload_1
      19: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: invokestatic  #6                  // Method java/util/UUID.randomUUID:()Ljava/util/UUID;
      25: invokevirtual #7                  // Method java/util/UUID.toString:()Ljava/lang/String;
      28: iconst_0
      29: iconst_1
      30: invokevirtual #8                  // Method java/lang/String.substring:(II)Ljava/lang/String;
      33: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      36: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      39: astore_1
      40: iinc          2, 1
      43: goto          5
      46: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
      49: aload_1
      50: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      53: return
}
```



