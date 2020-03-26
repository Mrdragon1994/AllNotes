---
title: Chapter 1 走进JAVA世界中的多线程
date: 2020-03-23 23:04:26
tags: 
- 线程
- Java
categories: 
- 多线程
- Java多线程编程实战指南
---

# Chapter 1 走进Java世界中的多线程

## 1.1 进程、线程与任务

1. ***进程(Process)***是程序的运行实例,一个运行的Java程序就是一个Java虚拟机进程；
2. 进程是程序向操作系统申请资源的基本单位,***线程(Thread)***是进程中可独立执行的最小单位；
3. 一个进程可以包括多个线程,***同一个进程中的所有线程共享该进程中的资源***,如内存空间;
4. 线程所要完成的计算就被称为***任务(Task)***;

## 1.2 多线程简介

1. Java标准类库java.lang.Thread就是Java平台对线程的实现,Thread类或其子类的一个实例就是一个线程;也就是说在Java平台创建一个线程就是创建一个Thread类(或其子类)的实例;

2. Thread的***run()***方法是线程处理任务逻辑的入口方法,它是由JVM在运行相应的线程时直接调用的，而不是由应用代码进行调用;

3. 运行一个线程实际上就是让JVM执行该线程的***run()***方法,从而使相应线程的任务处理逻辑代码得以执行；

4. Thread类的***start()***方法的作用是启动相应的线程,启动一个线程的实质就是请求JVM执行相应的线程的***run()***方法中的逻辑代码,但是，调用完start()后，并不意味着该线程已经启动了，而是需要线程调度器负责调度执行;

5. Thread类常用的两个构造器是:***Thread()***和***Thread(Runnable target)***，使用第一种方式时，需要继承***Thread()***这个抽象类，然后重写***run()***方法；使用第二种方式就是实现***Runnable()***接口，然后重写***run()***方法；

   ```java
   //方式1
   Thread thread = new WelcomeThread();
   
   calss WelcomeThread extends Thread{
       @Overrice
       public void run() {
           //do something
       }
   }
   
   //方式2
   Thread thread = new Thread(new WelcomeTask());
   
   calss WelcomeTask implements Runnable {
    	@Override
       public void run() {
           //do something
       }
   }
   ```

   6. 需要注意的是：***run()***方法如果是通过***start()***方法调用的，那么将会由JVM启动新的线程去运行，而如果我们手动去调用***run()***，那么将会由当前线程去直接执行***run()***的逻辑而不会由JVM创建新的线程去运行;

      ```groovy
      //调用start()方法
      class WelcomeApp {
          public static void main(String[] args) {
      
              Thread thread = new WelcomeThread()
              thread.start()
              println("1. welcome ${Thread.currentThread().getName()}")
          }
      }
      
      class WelcomeThread extends Thread {
          @Override
          void run() {
              println("2. welcome ${currentThread().getName()}")
          }
      }
      //输出：
      2. welcome Thread-1
      1. welcome main
      
      //调用run()方法
      class WelcomeApp {
          public static void main(String[] args) {
      
              Thread thread = new WelcomeThread()
              thread.run()
              println("1. welcome ${Thread.currentThread().getName()}")
          }
      }
      
      class WelcomeThread extends Thread {
          @Override
          void run() {
              println("2. welcome ${currentThread().getName()}")
          }
      }
      //输出：
      2. welcome main
      1. welcome main
      ```

      

   7. 线程的***start()***方法只能被调用一次,若多次调用**同一个Thread实例**的***start()***方法，则会抛出IllegalThreadStateException异常;

   8. 线程的属性：线程的编号(ID)，名称(Name)，线程类别(Daemon)，线程优先级(Priority)

      | 属性               | 属性类型及用途                                         | 是否是只读 | 注意事项                                                     |
      | ------------------ | ------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
      | 编号(ID)           | 类型：long，标识不同的线程                             | 是         | 当一次运行中，每个线程都有自己唯一的ID，但是当JVM平台重启后，编号可能会被复用。 |
      | 名称（Name）       | 类型：String，默认格式是：Thread-线程编号              | 否         | 可以自行给不同的线程起名字,方便调试。                        |
      | 线程类型（Daemon） | 类型：boolean，true表示是守护线程,false表示是用户线程. | 否         | setDaemon()必须用在start()方法前，即线程被启动前就必须设置好改属性。 |
      | 优先级（Priority） | 类型：int，Java提供了1-10，默认是5                     | 否         | 一般使用默认优先级即可。                                     |

   9. 用户线程用阻止JVM的正常停止,即一个Java虚拟机只有在所有的用户线程都结束运行后才能正常停止;而守护线程则不会；但是如果虚拟机是被强制停止的，比如kill -9或System.exit()，那么即便是用户线程也无法阻止JVM虚拟机的停止;

   10. Thread类常用的方法

       | 方法                           | 功能                               | 备注                                                         |
       | ------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
       | static Thread currentThread()  | 返回当前线程                       | 同一段代码的Thread.currentThread()可能返回不同的线程对象     |
       | void run()                     | 用于实现线程任务处理逻辑           | 该方法是由JVM直接调用，不建议手动调用。                      |
       | void satrt()                   | 启动相应的线程                     | 该方法返回并不代表该线程已经被启动，而是处于Runnbale状态，多次调用start会抛出异常。 |
       | void join()                    | 等待相应线程的运行结束             | 若线程A调用线程B的join方法,那么线程A的运行会被暂停，直到线程B运行结束。 |
       | static void yield()            | 使当前线程主动放弃其对处理器的占用 | 该方法不可靠                                                 |
       | static void sleep(long millis) | 使当前线程休眠(暂停运行)指定的时间 |                                                              |

   11. 线程的生命周期状态：Java线程的状态可以通过监控工具查看，也可通过***Thread.getState()***调用获取;***Thread.getState()***的返回值类型是一个枚举类型(Enum)。该枚举有以下几个状态：

       - New：一个已经创建但为启动的线程处于该状态,由于一个线程实例只能启动一次,因此一个线程只能处于一次该状态;
       - RUNNABLE：该状态可以看成是一个复合状态。它包括两个状态：Ready和RUNNING，执行***Thread.yield()***的线程，其状态可能由RUNNING转换为Ready;
       - BLOCKED:当一个线程发起文件读写或阻塞式Socket时，或者申请一个由其他线程持有的独占资源时，相应的线程会被***Blocked***,处于***BLOCKED***状态的线程**不会占用处理器资源**，当阻塞式I/O操作完成后，或者线程获得了其申请的资源，该线程的状态又可以转换为***RUNNABLE***；
       - WAITING：一个线程执行了某些特定方法后就会处于这种等待其他线程执行另外一些特定操作的状态。能够使线程变更为***WAITING***状态的方法包括：***Object.wait()***,***Thread.join()***,***LockSupport.park(Object)***,能够使相应线程从***WAITING***变更为***RUNNABLE***的方法包括：***Object.notify()***/***notifyAll()***,***LockSupport.unpark(Object)***;
       - TIME_WAITING：该状态和WAITING()类似，但是该线程并非无限期等待其他线程执行特定操作，而是在有限时间等待后，当其他线程没有在指定时间内执行所期望的待定操作后，该线程的状态自动切换为***RUNNABLE***。
       - TERMINATED：已经执行结束的线程处于该状态，由于一个线程实例只能够被启动一次，因此一个线程也只能有一次处于该状态。***Thread.run()***正常返回后者抛出异常而提前终止都会导致相应的线程处于该状态。