---
title: 深入理解java虚拟机-ch1-修改class文件
author: xxzuo
date: 2022-12-10 21:33:46
tags:
- jvm
categories:
- java虚拟机
---



## 1.安装asmtools.jar



## 2.创建一个java文件并编译

### 2.1创建 Foo.java

```java
public class Foo{
    public static void main(String[] args){
        boolean flag = true;
        if(flag){
            System.out.println("Hello Java!");
        }
        if(flag == true){
            System.out.println("Hello Jvm!");
        }
    }
}

```



### 2.2编译并运行 

```shell
root@LAPTOP:/usr/local/geek-jvm-study/ch1# javac Foo.java
root@LAPTOP:/usr/local/geek-jvm-study/ch1# java Foo
Hello Java!
Hello Jvm!
```

此时可以看出  输出了两条语句 `Hello Java!` 和 `Hello Jvm!`





## 3查看编译后的java文件 class文件

```shell
root@LAPTOP:/usr/local/geek-jvm-study/ch1# javap -verbose Foo
Classfile /usr/local/geek-jvm-study/ch1/Foo.class
  Last modified Dec 10, 2022; size 491 bytes
  MD5 checksum fdecd3da68c7736cfaa7cd44f19b3857
  Compiled from "Foo.java"
public class Foo
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #6                          // Foo
  super_class: #7                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #7.#17         // java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #20            // Hello Java!
   #4 = Methodref          #21.#22        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #23            // Hello Jvm!
   #6 = Class              #24            // Foo
   #7 = Class              #25            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               main
  #13 = Utf8               ([Ljava/lang/String;)V
  #14 = Utf8               StackMapTable
  #15 = Utf8               SourceFile
  #16 = Utf8               Foo.java
  #17 = NameAndType        #8:#9          // "<init>":()V
  #18 = Class              #26            // java/lang/System
  #19 = NameAndType        #27:#28        // out:Ljava/io/PrintStream;
  #20 = Utf8               Hello Java!
  #21 = Class              #29            // java/io/PrintStream
  #22 = NameAndType        #30:#31        // println:(Ljava/lang/String;)V
  #23 = Utf8               Hello Jvm!
  #24 = Utf8               Foo
  #25 = Utf8               java/lang/Object
  #26 = Utf8               java/lang/System
  #27 = Utf8               out
  #28 = Utf8               Ljava/io/PrintStream;
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               println
  #31 = Utf8               (Ljava/lang/String;)V
{
  public Foo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_1
         1: istore_1
         2: iload_1
         3: ifeq          14
         6: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: ldc           #3                  // String Hello Java!
        11: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        14: iload_1
        15: iconst_1
        16: if_icmpne     27
        19: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        22: ldc           #5                  // String Hello Jvm!
        24: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        27: return
      LineNumberTable:
        line 3: 0
        line 4: 2
        line 5: 6
        line 7: 14
        line 8: 19
        line 10: 27
      StackMapTable: number_of_entries = 2
        frame_type = 252 /* append */
          offset_delta = 14
          locals = [ int ]
        frame_type = 12 /* same */
}
SourceFile: "Foo.java"
```





## 4.利用asmtools修改class文件

```shell
root@LAPTOP:/usr/local/geek-jvm-study/ch1# java -cp ../asmtool/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm.1
root@LAPTOP:/usr/local/geek-jvm-study/ch1# awk 'NR==1,/iconst_1/{sub(/iconst_1/,"iconst_2")} 1' Foo.jasm.1 > Foo.jasm
root@LAPTOP:/usr/local/geek-jvm-study/ch1# java Foo
Hello Java!
Hello Jvm!
root@LAPTOP:/usr/local/geek-jvm-study/ch1# java -cp ../asmtool/asmtools.jar org.openjdk.asmtools.jasm.M
ain Foo.jasm
root@LAPTOP-0GUCAKHK:/usr/local/geek-jvm-study/ch1# java Foo
Hello Java!
```

此时 只输出了 `Hello Java!` 没有输出`Hello Jvm!`

> 这里 awk 'NR==1,/iconst_1/{sub(/iconst_1/,"iconst_2")} 1' Foo.jasm.1 是 将 boolean 变量 flag 的值从 1 变成了 2 



## 5.再次查看编译后的文件

```shell
root@LAPTOP:/usr/local/geek-jvm-study/ch1# javap -verbose Foo
Classfile /usr/local/geek-jvm-study/ch1/Foo.class
  Last modified Dec 10, 2022; size 429 bytes
  MD5 checksum 97d020b83c40549a0440ebfa5597a1cd
  Compiled from "Foo.jasm"
public class Foo
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #25                         // Foo
  super_class: #8                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = String             #10            // Hello Jvm!
   #2 = String             #15            // Hello Java!
   #3 = Fieldref           #27.#11        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #8.#17         // java/lang/Object."<init>":()V
   #5 = Methodref          #12.#30        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #6 = Utf8               (Ljava/lang/String;)V
   #7 = Utf8               out
   #8 = Class              #9             // java/lang/Object
   #9 = Utf8               java/lang/Object
  #10 = Utf8               Hello Jvm!
  #11 = NameAndType        #7:#23         // out:Ljava/io/PrintStream;
  #12 = Class              #14            // java/io/PrintStream
  #13 = Utf8               ([Ljava/lang/String;)V
  #14 = Utf8               java/io/PrintStream
  #15 = Utf8               Hello Java!
  #16 = Utf8               main
  #17 = NameAndType        #29:#20        // "<init>":()V
  #18 = Utf8               SourceFile
  #19 = Utf8               println
  #20 = Utf8               ()V
  #21 = Utf8               StackMapTable
  #22 = Utf8               Foo.jasm
  #23 = Utf8               Ljava/io/PrintStream;
  #24 = Utf8               Code
  #25 = Class              #26            // Foo
  #26 = Utf8               Foo
  #27 = Class              #28            // java/lang/System
  #28 = Utf8               java/lang/System
  #29 = Utf8               <init>
  #30 = NameAndType        #19:#6         // println:(Ljava/lang/String;)V
{
  public Foo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #4                  // Method java/lang/Object."<init>":()V
         4: return

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_2
         1: istore_1
         2: iload_1
         3: ifeq          14
         6: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: ldc           #2                  // String Hello Java!
        11: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        14: iload_1
        15: iconst_1
        16: if_icmpne     27
        19: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        22: ldc           #1                  // String Hello Jvm!
        24: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        27: return
      StackMapTable: number_of_entries = 2
        frame_type = 252 /* append */
          offset_delta = 14
          locals = [ int ]
        frame_type = 12 /* same */
}
SourceFile: "Foo.jasm"
```

两次 主要不同的地方在这里

原始的

```shell
	public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_1
         1: istore_1
         2: iload_1
         3: ifeq          14
         6: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: ldc           #3                  // String Hello Java!
        11: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        14: iload_1
        15: iconst_1
        16: if_icmpne     27
        19: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        22: ldc           #5                  // String Hello Jvm!
        24: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        27: return
      LineNumberTable:
        line 3: 0
        line 4: 2
        line 5: 6
        line 7: 14
        line 8: 19
        line 10: 27
      StackMapTable: number_of_entries = 2
        frame_type = 252 /* append */
          offset_delta = 14
          locals = [ int ]
        frame_type = 12 /* same */
```



修改后的

```shell
	public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_2                       //  我们使用 asmtools 修改了这里
         1: istore_1
         2: iload_1
         3: ifeq          14
         6: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: ldc           #2                  // String Hello Java!
        11: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        14: iload_1
        15: iconst_1
        16: if_icmpne     27
        19: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        22: ldc           #1                  // String Hello Jvm!
        24: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        27: return
      StackMapTable: number_of_entries = 2
        frame_type = 252 /* append */
          offset_delta = 14
          locals = [ int ]
        frame_type = 12 /* same */
```

对于 java虚拟机来说 ，boolean类型被映射成了 整数类型

> if(flag == true){} 就是 判断 flag是否等于1
>
> if(flag){} 就是判断 flag是否不为0



















