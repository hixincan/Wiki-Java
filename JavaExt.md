

# 参考

在线api：https://www.oracle.com/cn/java/technologies/java-se-api-doc.html

javase：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html （chrome直接翻译成中文看！再结合阅读插件提升阅读体验）

官方：

https://docs.oracle.com/javase/specs/index.html

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html

官方文档是最好的学习资料



## 学习工具

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



# 更新日志

* JDK 7
* JDK 8

官方：https://www.oracle.com/cn/java/technologies/javase/8-whats-new.html

官方英文：https://www.oracle.com/java/technologies/javase/8-whats-new.html

Stream

* JDK 9
* JDK 10
* JDK 11
* JDK 12



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
