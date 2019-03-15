# Java 常量池理解与总结

理解常量池，首先要理解方法区，因为常量池是方法区的一部分。方法区是线程共享的内存区域，用于存储已经被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等。

## 常量池的分类

### 运行时常量池

- **运行时常量池是方法区的一部分，**所以也是全局共享的。

- **其作用是存储 Java 类文件常量池中的符号信息。**

- **class 文件中存在常量池(非运行时常量池)，其在编译阶段就已经确定；JVM 规范对 class 文件结构有着严格的规范，必须符合此规范的 class 文件才会被 JVM 认可和装载。**

- **运行时常量池** 中保存着一些 class 文件中描述的符号引用，同时还会将这些符号引用所翻译出来的直接引用存储在 **运行时常量池** 中。

- **运行时常量池相对于 class 常量池一大特征就是其具有动态性，Java 规范并不要求常量只能在运行时才产生，也就是说运行时常量池中的内容并不全部来自 class 常量池，class 常量池并非运行时常量池的唯一数据输入口；在运行时可以通过代码生成常量并将其放入运行时常量池中。**


#### string pool

运行时常量池保存编译期生成的各种字面量和符号引用，这部分在类加载后进入方法区的运行时常量池中存放。运行时常量池未必必须编译期间生成，在运行时也可以装载新的对象。例如String  中 intern() 就可以将String 对象装载在常量池中，`intern` 方法 是个native方法，可以看这个方法的说明 ： 

```tex
When the intern method is invoked, if the pool already contains a string equal to this String object as determined by the equals(Object) method, then the string from the pool is returned. Otherwise, this String object is added to the pool and a reference to this String object is returned.

It follows that for any two strings s and t, s.intern() == t.intern() is true if and only if s.equals(t) is true.

All literal strings and string-valued constant expressions are interned. String literals are defined in section 3.10.5 of the The Java™ Language Specification.

```

两个例子 :

```java

        String str1 = "abcd";
        String str2 = new String("abcd");
        System.out.println(str1==str2);//false
        System.out.println(str1 == str2.intern());  //true
     
```





#### Byte ,Short,Interger .... 的常量池



这5种包装类默认创建了数值[-128，127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。

```java
        Integer a = 110;
        Integer b = 110;
        System.out.println(a == b);  //true
        Integer a1 = 600;
        Integer b1 = 600;
        System.out.println(a1 == b1); // false
```





#### 版本变迁的影响 

**JDK7中符号表被移动到 Native Heap中，字符串常量和类引用被移动到 Java Heap中。**



**JDK7 常量池被移动到 Native Heap(Java Heap)，所以即使设置了持久代大小，也不会对常量池产生影响；不断while循环在当前的代码中，所有int的字符串相加还不至于撑满 Heap 区，所以不会出现异常。**



```java

		 String s1 = new StringBuilder("漠").append("然").toString();
		 System.out.println(s1.intern() == s1);

		 String s2 = new StringBuilder("漠").append("然").toString();
		 System.out.println(s2.intern() == s2);
```

以上代码，在 JDK6 下执行结果为 false、false，在 JDK7 以上执行结果为 true、false。



在 JDK6 下 s1、s2 指向的是新创建的对象，**该对象将在 Java Heap 中创建，所以 s1、s2 指向的是 Java Heap 中的内存地址；**调用 intern 方法后将尝试在常量池中查找该对象，没找到后将其放入常量池并返回，**所以此时 s1/s2.intern() 指向的是常量池中的地址，JDK6常量池在方法区，与堆隔离，；所以 s1.intern()==s1 返回false。**