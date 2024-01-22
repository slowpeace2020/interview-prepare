# 使用JavaAgent测试Object的大小

作者：马士兵 http://www.mashibing.com

##1对象的创建过程
1. class loading
2. class linking (verification, preparation, resolution)
3. class initializing
4. 申请对象内存
5. 成员变量赋默认值
6. 调用构造方法<init>
    1.成员变量顺序赋初始值
    2.执行构造方法语句


## 对象大小（64位机）

### 观察虚拟机配置

java -XX:+PrintCommandLineFlags -version

### 普通对象
[对象构成](https://www.cnblogs.com/jajian/p/13681781.html)
1. 对象头：markword  8
2. ClassPointer指针：-XX:+UseCompressedClassPointers 为4字节 不开启为8字节
3. 实例数据
   1. 引用类型：-XX:+UseCompressedOops 为4字节 不开启为8字节 
      Oops Ordinary Object Pointers
4. Padding对齐，8的倍数

### 数组对象

1. 对象头：markword 8
2. ClassPointer指针同上
3. 数组长度：4字节
4. 数组数据
5. 对齐 8的倍数

## 实验

1. 新建项目ObjectSize （1.8）

2. 创建文件ObjectSizeAgent

   ```java
   package com.mashibing.jvm.agent;
   
   import java.lang.instrument.Instrumentation;
   
   public class ObjectSizeAgent {
       private static Instrumentation inst;
   
       public static void premain(String agentArgs, Instrumentation _inst) {
           inst = _inst;
       }
   
       public static long sizeOf(Object o) {
           return inst.getObjectSize(o);
       }
   }
   ```

3. src目录下创建META-INF/MANIFEST.MF

   ```java
   Manifest-Version: 1.0
   Created-By: mashibing.com
   Premain-Class: com.mashibing.jvm.agent.ObjectSizeAgent
   ```

   注意Premain-Class这行必须是新的一行（回车 + 换行），确认idea不能有任何错误提示

4. 打包jar文件

5. 在需要使用该Agent Jar的项目中引入该Jar包
   project structure - project settings - library 添加该jar包

6. 运行时需要该Agent Jar的类，加入参数：

   ```java
   -javaagent:C:\work\ijprojects\ObjectSize\out\artifacts\ObjectSize_jar\ObjectSize.jar
   ```

7. 如何使用该类：

   ```java
   ​```java
      package com.mashibing.jvm.c3_jmm;
      
      import com.mashibing.jvm.agent.ObjectSizeAgent;
      
      public class T03_SizeOfAnObject {
          public static void main(String[] args) {
              System.out.println(ObjectSizeAgent.sizeOf(new Object()));
              System.out.println(ObjectSizeAgent.sizeOf(new int[] {}));
              System.out.println(ObjectSizeAgent.sizeOf(new P()));
          }
      
          private static class P {
                              //8 _markword
                              //4 _oop指针
              int id;         //4
              String name;    //4
              int age;        //4
      
              byte b1;        //1
              byte b2;        //1
      
              Object o;       //4
              byte b3;        //1
      
          }
      }
   ```

## Hotspot开启内存压缩的规则（64位机）

1. 4G以下，直接砍掉高32位
2. 4G - 32G，默认开启内存压缩 ClassPointers Oops
3. 32G，压缩无效，使用64位
   内存并不是越大越好（^-^）

## IdentityHashCode的问题

回答白马非马的问题：

当一个对象计算过identityHashCode之后，不能进入偏向锁状态

https://cloud.tencent.com/developer/article/1480590
 https://cloud.tencent.com/developer/article/1484167

https://cloud.tencent.com/developer/article/1485795

https://cloud.tencent.com/developer/article/1482500

## 对象定位

•https://blog.csdn.net/clover_lily/article/details/80095580

1. 句柄池
2. 直接指针

在Java虚拟机（JVM）中，句柄池和直接指针是两种不同的方式，用于管理对Java对象的引用。了解这两种方法的区别有助于理解JVM是如何管理内存和对象引用的。

1. **句柄池 (Handle Pool)**
   - 在句柄池的方法中，Java对象在堆内存中分配，而引用（或句柄）则存储在句柄池中。
   - 每个句柄包含了对象数据、类型数据和同步信息等的指针。
   - 当Java程序引用一个对象时，它实际上持有的是指向句柄池中特定句柄的指针。
   - 这种方法的优点是，当对象被移动（例如，在垃圾回收期间）时，只需要更新句柄池中的句柄，而不需要更新所有引用该对象的地方。
   - 缺点是，访问对象数据需要两次间接引用：首先访问句柄，然后通过句柄访问对象。

2. **直接指针 (Direct Pointer)**
   - 在直接指针方法中，Java引用直接指向堆内存中的对象。
   - 这意味着每个引用包含了对象的实际内存地址。
   - 当访问一个对象时，只需要一次间接引用，这可以提高访问速度。
   - 然而，这种方法的缺点是，在对象移动时（例如，在进行垃圾回收的压缩阶段），所有指向该对象的引用都需要更新，这可能是一个昂贵的操作。

不同的JVM实现可能选择不同的方式来管理对象引用。有些实现可能使用句柄池，而其他实现可能使用直接指针，或者根据特定的场景和优化需求选择不同的策略。例如，HotSpot JVM早期版本使用句柄池，但后来的版本转向使用直接指针，以提高性能。

##对象内存分配
[图文](https://www.cnblogs.com/jianwei-dai/p/15402683.html)