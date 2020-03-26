---
title: Chapter 2 多线程编程的目标与挑战
date: 2020-03-24 11:05:01
categories:
- 多线程
- Java
- Java多线程编程实战指南
tags:
- 线程
- Java
---

# Chapter 2 多线程编程的目标与挑战

## 2.1 串行、并发与并行

1. 串行(***Sequential***)：按照顺序依次完成多件事情,不会出现插队争抢;

2. 并发(***Concurrent***)：以交替的方式利用等待某件事完成的事件来做其他事情;
3. 并行(***Parallel***)：找与当前事件相同数量的人同时去做这些事情;
4. **<font color=#1A1AE6>并行</font>**可以被看作是一种更为严格，理想的并发；
5. 从计算机的硬件角度考虑：在一个处理器一次只能够运行一个线程的情况下，由于处理器可以使用时间片(Time-slice)分配的技术来实现在同一端时间内运行多个线程，因此一个处理器就可以实现**<font color=#1A1AE6>并发</font>**；而**<font color=#1A1AE6>并行</font>**则需要过个处理器在同一时刻各自运行一个线程来实现。

## 2.2 竞态

1. 状态变量(***State Variable***)：即类的实例变量,静态变量;

2. 共享变量(***Shared Variable***)：可以被多个线程共同访问的变量，需要明确的是：<font color=#1A1AE6>状态变量</font>由于可以被多个线程共享，因此状态变量是共享变量；而**<font color=#1A1AE6>局部变量</font>**是不可以被共享的，因此也不用担心局部变量出现的竞态问题;

3. 竞态的模式和竞态的产生条件

   3.1 竞态的两种模式是：***read-modify-write(读-改-写)***和***check-then-act(检测后行动)***

   3.2 read-modify-write操作可以分为三个步骤：

   - 读取(***read***)一个共享变量的值;

   - 然后根据该值做一些计算(***modify***)，此间便可能会改变该值;

   - 接着更新共享变量的值(***write***)

   - 如：i++这样的操作就是典型的read-modify-write模式

     ```groovy
     load(i,r1) //指令①，从内存中将i的值读取到寄存器r1
     increment(i) //指令②，将寄存器中r1的值加1
     store(i) //指令③，将寄存器r1的内容写入到i所对应的内存空间中
     ```

     *为什么会出现脏数据的问题？*

     ​	当一个线程执行完指令①之后到开始或正在执行指令②的这段时间内，其他线程已经更新了该共享变量值，导致该线程在执行指令②的时候使用的是共享变量的旧值，该线程再把这个“旧值”更新到内存中，其实是对其他线程已更新过的数据进行了覆盖。

   3.3 check-then-act操作可以分为如下步骤

   - 读取某个共享变量的值，根据该变量的值判断下一步的动作是什么

     ```groovy
     if	(i >= 999) { //子操作①：检测共享变量(check)
         i = 0 //子操作②：进行下一步操作(ack)
     } else {
         i++ //子操作②：进行下一步操作(ack)
     }
     ```

     *为什么会出现脏数据？*

     ​	一个线程在执行完子操作①到开始或正在执行子操作②的这段时间内（如第一个线程读取到的i=998，那么他会执行i++），其他线程可能已经更新了该共享变量的值(该线程读取到的也是998,且已经执行过了i++,此时i已经为999了)，那么第一个线程仍旧会去执行i++，但是此时的条件是不满足的。

    3.4 竞态可以被看作访问(读取，更新)同一组共享变量的多个线程所执行的操作相互交      替(Interleave)，比如一个线程读取共享变量并以该变量为基础进行计算的期间另外一个线程更新了该共享变量的值而导致的干扰(读取脏数据)或者冲突(丢失更新)的结果

   3.5 对于局部变量（形式参数和方法体内定义的变量），由于不同的线程访问的是各自的那一份局部变量，所以局部变量不会出现竞态现象;
   
   3.6 消除竞态可以把变量弄为局部变量或者使用***synchronized***关键字修改方法，其原理是在任一时刻该方法只能被一个线程访问，从而避免了这个方法的交替执行而导致的干扰。
   
   4. 线程安全问题概括为**原子性**，**可见性**和**有序性**
   
      4.1  原子性
   
      ​	原子性重在“不可分割”，而该“不可分割”有两层含义：
   
      ​    其一是：访问（读，写）某个共享变量的操作从其执行线程意外的任何线程来看，该操作要么已经执行结束要么尚未发生，即其他线程不会“看到”该操作执行了部分的中间效果
   
      ​    其二是：设O1和O2是访问共享变量V的两个原子操作，这两个操作并非都是读操作，那么一个线程执行O1期间（开始执行且未执行完毕），其他线程无法执行O2，也就是，访问同一组共享变量的原子操作是不能被 交替的。
   
      ​    **Java有两种方式实现原子性**：1）、使用锁机制（Lock/synchronized），它能保障一个共享变量在任意一个时刻只能被一个线程访问，这就排除了多个线程在同一时刻同时访问同一个共享变量而导致的干扰和冲突，即消除了竞态；
   
      ​                                                         2）、使用处理器提供的CAS指令，它是在硬件（处理器和内存）这一层次实现的。
   
       在Java语言中，long和double类型以外的任何类型的变量的***写操作***都是原子操作，即对基本类型（long/double除外,仅包括byte, boolean, short, char, float, int）的变量和引用型变量的写操作都是原子的。
   
      - Java语言规范规定对于long/double类型的变量可以用***volatile***修饰，使其写操作具备原子性。
      - 但是需要注意的是，***volatile***仅仅能够保证变量***写操作***的原子性，不能保证其他操作的原子性（如：read-modify-write和check-then-act）。
      - Java语言中针对任意类型变量的读操作都是原子操作。
   
      4.2 可见性
   
      - 在多线程环境下，一个线程对某个共享变量进行更新后，后续访问该变量的线程可能无法立刻读取到这个更新的结果，甚至永远无法读取到更新后的结果，这就是线程安全的***可见性***。
   
      - 保证可见性的方法是使用：***volatile***关键字，它一个作用是提示JIT编译器被该关键字修饰的变量可能会被多个线程共享，从而阻止JIT编译器对程序作出的优化；另一个作用就是读取一个volatile关键字修饰的变量会使相应的处理器执行刷新处理器缓存的动作。
   
      - JLS（Java语言规范）保证，父亲线程在启动子线程之前，若对共享变量作出了修改，那么该共享变量的值对于子线程来说是可见的。
   
        ```groovy
        class TestVisibility {
        
            //共享变量
            static int data = 0
        
            public static void main(String[] args) {
        
                Thread thread = new Thread() {
                    @Override
                    void run() {
                        Tools.randomPause(R) //使当前线程休眠R毫秒
                        println(data)
                    }
                }
                data = 1 //共享变量值改变了
                thread.start()
            }
        ```
   
        以上代码的最后输出**一定**是：1
   
        ```groovy
        class TestVisibility {
        
            //共享变量
            static int data = 0
        
            public static void main(String[] args) {
        
                Thread thread = new Thread() {
                    @Override
                    void run() {
                        Tools.randomPause(R) //使当前线程休眠R毫秒
                        println(data)
                    }
                }
                data = 1  //在启动子线程前更改共享变量
                thread.start()
                Tools.randomPause(R) //使当前线程休眠R毫秒
                data = 2  //在启动子线程后更改共享变量
            }
        }
        ```
   
        以上代码的输出结果可能是1也可能是2，这将取决于Tools.randomPause(R) 的R的大小，如果主线程休眠时间长，那么子线程读取到的就是1，如果主线程休眠时间短，由于其将data修改为2后，被缓存更新到了内存中，因此子线程可能会读取到2。
   
        - Java语言规范保证一个线程终止后该线程对共享变量的更新对于调用该线程的join方法的线程而言是可见的
   
          ```groovy
          class TestVisibility2 {
              static int data = 0 //共享变量
              public static void main(String[] args) {
                  Thread thread = new Thread() {
                      @Override
                      void run(){
                          Tools.randomPause(50)
                          data = 1 //子线程更新共享变量
                      }
                  }
                  thread.start()
                  try {
                      thread.join() //main线程调用thread.join()
                  } catch(Exception e) {
          
                  }
                  println(data)
              }
          }
          
          class Tools {
             static void randomPause(int millis) {
                 int next = new Random().nextInt(millis)
                 Thread.sleep(next)
             }
          }
          
          ```
   
          以上代码中，线程thread运行时将共享变量data的值更新为1，因此main线程对线程thread的join方法调用结束后，该线程（main）能够保证共享变量data的值为1.
   
        4.3 有序性
   
        - 有序性指的是在一个处理器上运行的一个线程所执行的内存访问操作在另一个处理器上运行的其他线程看来是乱序的；所谓***乱序(Out of Order)***，是指内存访问操作的顺序看起来像是发生了变化。
        
        - 编译器可能改变两个操作的先后顺序，处理器可能不是完全按照程序的目标代码所指定的顺执行指令；另外，一个处理器上执行的多个操作，从其他处理器的角度来看其顺序可能与目标代码所指定的顺序不一致，这种现象叫做***重排序(Reordering)***。
        
     - 重排序是对内存访问有关的操作(读和写)所做的一种优化，它可以在不影响单线程程序正确的情况下提升程序的性能,但是，它可能对多线程程序的正确性产生影响。
       
        - 目前能够对我们代码进行重排序的有编译器，处理器，存储子系统。
        
        - 我们将顺序定义为如下几种：
        
          - 源代码顺序：源代码中指定的内存访问操作顺序（就是程序员编写代码时的顺序）
          - 程序顺序：这里理解为JVM编译出的字节码(Byte Code)
          - 执行顺序：内存访问操作在给定处理器上的实际顺序
          - 感知顺序：给定处理器所感知到的该处理器及其他处理器的内存访问操作发生的顺序
        
          基于上面两种，我们将重排序划分为指令重排序和存储子系统重排序。
        
          | 重排序类型       | 重排序表现                                                   | 重排序来源                    |
          | ---------------- | ------------------------------------------------------------ | ----------------------------- |
          | 指令重排序       | 程序顺序与源码顺序不一致<br />执行顺序与程序顺序不一致       | 编译器<br />JIT编译器，处理器 |
          | 存储子系统重排序 | 源码顺序，程序顺序和执行顺序一致，但是感知顺序和执行顺序不一致 | 高速缓存，写缓冲器            |
          |                  |                                                              |                               |
        
          - 指令重排序(它重排序的是指令,是实实在在对指令的顺序进行了排序)
        
            ==Java平台包含两种编译器：静态编译器(javac)和动态编译器(JIT编译器)。前者的作用是将Java源代码(.java文本文件)编译为字节码(.class字节码文件)，它是在代码编译阶段介于。后者的作用是将字节码动态编译为Java虚拟机机宿主机的本地代码(机器码)，它是在Java程序运行过程中介入的。
        
            在Java平台上，静态编译器(javac)基本上不会执行指令重排序，而是在动态编译时会出现。
            
          - 存储子系统重排序（存储子系统重排序是一种现象而不是一种动作，它并没有真正对指令执行顺序进行调整，只是造成了一种指令的执行顺序像是被调整过一样）
          
          - 保证有序性的措施，volatile关键字，synchronized关键字。
          
          - 需要注意的是：volatile只能够保证写操作的原子性(其实是针对long和double的，因为其他类型的写操作都是原子性的)，但是其他基本类型和引用类型的写操作都是原子性的（其中引用类型的写操作其实就是赋值操作）。但是所有的读操作都是原子性的。
            
          ```java
            package com.thread.chapter2
          
            /**
             * @Author ChangQilong
             * @Date 2020/3/26 23:15
             */
            class TestAtom {
            
                public static void main(String[] args) {
                    AtomicityDemo atomicityDemo = new AtomicityDemo()
                    Thread thread = new Thread() {
                        @Override
                        void run() {
            
                            atomicityDemo.updateHostInfo("123", 12)
            
                        }
                    }
            
                    Thread thread1 = new Thread() {
                        @Override
                        void run() {
                            sleep(100) //重点①
                            atomicityDemo.connect()
                        }
                    }
            
                    thread.start()
                    thread1.start()
                }
            
            }
            
            class AtomicityDemo {
            
                private HostInfo hostInfo
            
                public void updateHostInfo(String ip, int port) {
                    HostInfo hostInfo1 = new HostInfo(ip, port)
                    hostInfo = hostInfo1
                }
            
                public void connect() {
                    println(hostInfo.getIp())
                }
            
            
            }
            
            class HostInfo {
                private String ip
                private int port
            
                HostInfo() {
                }
            
                HostInfo(String ip, int port) {
                    this.ip = ip
                    this.port = port
                }
            
                String getIp() {
                    return ip
                }
            
                void setIp(String ip) {
                    this.ip = ip
                }
            
                int getPort() {
                    return port
                }
            
                void setPort(int port) {
                    this.port = port
                }
            }
            ```
            
            
            
            以上代码，首先两个线程共享了同一个对象的同一份变量,线程1去赋值，线程2去读值，因此要考虑线程1和2的执行先后顺序，不然读值的那个线程读到的就是null。
            
            
            
            ```java
            package com.thread.chapter2
            
            /**
             * @Author ChangQilong* @Date 2020/3/26 23:15
             */
            class TestAtom {
            
                public static void main(String[] args) {
                    AtomicityDemo atomicityDemo = new AtomicityDemo()
            
                    Thread thread = new Thread() {
                        @Override
                        void run() {
                            atomicityDemo.updateHostInfo("123", 12)
                        }
                    }
            
                    Thread thread1 = new Thread() {
                        @Override
                        void run() {
                            //sleep(500) //步骤5
                            atomicityDemo.connect()
                        }
                    }
                    thread.start()
                    thread1.start()
                }
            }
            
            class AtomicityDemo {
            
                private HostInfo hostInfo = new HostInfo()
            
            
                public void updateHostInfo(String ip, int port) {
                    hostInfo.setIp(ip)  //步骤1
                    hostInfo.setPort(port) //步骤2
                }
            
            
                public void connect() {
                    println(hostInfo.getIp()) //步骤3
                    println(hostInfo.getPort()) //步骤4
                }
            
                static class HostInfo {
                    private String ip
                    private int port
                    private static aaa
            
                    HostInfo() {
                    }
            
                    HostInfo(String ip, int port) {
                        this.ip = ip
                        this.port = port
                    }
            
                    String getIp() {
                        return ip
                    }
            
                    void setIp(String ip) {
                        this.ip = ip
                    }
            
                    int getPort() {
                        return port
                    }
            
                    void setPort(int port) {
                        this.port = port
                    }
                }
            
            }
            ```
            
            
            
            由于赋值线程和取值线程会一起运行，因此步骤1和2就不能保证在步骤3和4的前面，因此可能读到的值是null,0；如果让线程2 sleep(X)一定时间(步骤5)，让赋值函数去走完，那么取值函数便能取到正确结果；
            
            ```java
            //取值线程不sleep()的输出
            null
            12
            //取值线程sleep()一定时间
            123
            12
            ```
            
            **PS**这里需要注意的是：第32行，需要手动调用无参构造器，不然HostInfo()是可以调用其静态属性的，但是该类不会初始化，也就是hostinfo的引用是null。
   
