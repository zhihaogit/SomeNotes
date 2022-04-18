# 并发编程_原理

## synchronized原理

```java
static final object lock = new Object();
static int counter = 0;

public static void main(String[] args) {
  synchronized(lock) {
    counter++;
  }
}
```

对应的字节码：

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/synchronized_byte_code-2022-04-14-6c10e2b94eea437aa9182a8b7200f3c1-f8f67f-2022-04-15-6c10e2b94eea437aa9182a8b7200f3c1-cbb9dd.png" alt="synchronized_byte_code" style="zoom:25%;" />

## synchronized原理进阶

### 1. 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程访问，但多线程的访问时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化

轻量级锁对使用者是透明的，即语法仍然是 synchronized

假设有两个方法同步块，利用同一个对象加锁

```java
static final Object obj = new Object();
public static void method1() {
  synchronized(obj) {
    // 同步块 A
    method2();
  }
}
public static void method2() {
  synchronized(obj) {
    // 同步块 B
  }
}
```

- 创建锁记录（Lock Record）对象，每个线程的栈帧都户包含一个锁记录的结构，内部可以存储锁定对象的 Mark word

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_init-2022-04-15-65425335a4317401b2a4c57594b04f2b-4d6dbc.png" alt="lock_init" style="zoom:50%;" />

- 让锁记录中 Object referenct指向锁对象，并尝试用 cas替换 Object的 Mark word，将 Mark word的值存入锁记录

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_mark_word_replace-2022-04-15-bada6511aaabb7c0961962f298cf8387-8f5038.png" alt="lock_mark_word_replace" style="zoom:50%;" />

- 如果 cas替换成功，对象头中存储了*锁记录地址和状态*00，表示由该线程给对象加锁，这时图示如下

  <img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_mark_word_repalce1-2022-04-15-0885d29b5bf29e641eadf34c40c6dadf-96d7a8.png" alt="lock_mark_word_repalce1" style="zoom:50%;" />

- 如果 cas失败，有两种情况：

  - 如果是其它线程已经持有了该 Object的轻量级锁，这时表明有竞争，进入锁膨胀过程
  - 如果是自己执行了 synchronized锁重入，那么再添加一条 Lock Record作为重入的计数

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_fail-2022-04-15-532316aa299617fceb690ff16ece4b90-9a5e8f.png" alt="lock_fail" style="zoom:50%;" />

- 当退出 synchronized代码块（解锁时），如果有取值为 null的锁记录，表示有重入，这时重置锁记录，表示重入计数减一

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_mark_word_repalce1-2022-04-15-0885d29b5bf29e641eadf34c40c6dadf-96d7a8.png" alt="lock_mark_word_repalce1" style="zoom:50%;" />

- 当退出 synchronized代码块（解锁时），锁记录的值不为 null，这时使用 cas将 Mark word的值恢复给对象头
  - 成功，则解锁成功
  - 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁流程

### 2. 锁膨胀

如果在尝试加轻量级锁的过程中，cas操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁

```java
static Object obj = new Object();
public static void method1() {
  synchronized(obj) {
    // 同步块
  }
}
```

- 当 Thread-1进行轻量级加锁时，Thread-0已经对该对象加了轻量级锁

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_with_two_threads-2022-04-15-b18ea4d7e7d3dab1535d1e4b6c01bce0-5614b4.png" alt="lock_with_two_threads" style="zoom:50%;" />

- 这时 Thead-1加轻量级锁失败，进入锁膨胀流程
  - 即为 object对象申请 Monitor锁，让 object指向重量级锁地址
  - 然后自己进入 Monitor的 EntryList BLOCKED

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_with_monitor-2022-04-15-61178e610a7ef2eaaced414c51ae7fa6-176cbd.png" alt="lock_with_monitor" style="zoom:50%;" />

- 当 Thread-0退出同步块解锁时，使用 cas将 Mark word的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor地址找到 Monitor对象，设置 Owner为 null，唤醒 EntryList中 BLOCKED线程

### 3. 自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程可以避免阻塞

自旋重试成功的情况

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_monitor_spin_thread-2022-04-15-d47ed9e4ed27e4b7bb9bec81ac51c1b3-3ae962.png" alt="lock_monitor_spin_thread" style="zoom:50%;" />

自旋重试失败的情况

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lock_monitor_spin_thread_fail-2022-04-15-31a56b23e0ba1dacb872574a632de294-5c2d17.png" alt="lock_monitor_spin_thread_fail" style="zoom:50%;" />

- 在 java6之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就会多自旋几次，反之，就少自旋甚至不自旋，总之，比较智能
- 自旋会占用 cpu时间，单核 cpu自旋没有意义，多核 cpu自旋才能发挥优势
- java7之后不能控制是否开启自旋

### 4. 偏向锁

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 cas操作

java6中引入了偏向锁来做进一步优化：只有第一次使用 cas将线程ID设置到对象的 Mark word头，之后发现这个线程 ID是自己的就表示没有竞争，不用重新 cas。以后只要不发生竞争，这个对象就归该线程所有

例如：

```java
static final Object obj = new Object();
public static void m1() {
  synchronized(obj) {
    // 同步块 a
    m2();
  }
}
public static void m2() {
  synchronized(obj) {
    // 同步块 b
    m3();
  }
}
public static void m3() {
  synchronized(obj) {
    // 同步块 c
  }
}
```

轻量级锁流程

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/lightweight_lock_thread-2022-04-15-e4769558a39fbaee7e76fc01af6225db-99c280.png" alt="lightweight_lock_thread" style="zoom:50%;" />

偏向锁流程

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/biasedlocking_thread-2022-04-15-9dc6abdb6b52f891086234a6aa959b83-6774d3.png" alt="biasedlocking_thread" style="zoom:50%;" />

#### 偏向状态

对象头格式：

<img src="https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/mark_word-2022-04-14-5b4009436e20787dda0e6a369358936d-8bd6f6.png" alt="mark_word" style="zoom:50%;" />

一个对象创建时：

- 如果开启了偏向锁（默认开启），那么对象创建后，mark word值为 0x05即最后 3位 101，这时它的 thread、epoch、age都为 0
- 偏向锁是默认延迟的，不会再程序启动后立即生效，如果想避免延迟，可以加 VM参数 `-XX:BiasedLockingStartupDelay=0`来禁用延迟
- 如果没有开启偏向锁，那么对象创建后，mark word值为 0x01即最后 3位为 001，这时它的 hashcode、age都为 0，第一次用到 hashcode时才会赋值

1）测试延迟性

```java
Dog d = new Dog();
synchronized(d) {}
```

2）测试偏向锁

利用 jol第三方工具来查看对象头信息

注意：处于偏向锁的对象解锁后，线程 id仍存储于对象头中

3）测试禁用

添加VM参数 `-XX:-UseBiasedLocking`禁用偏向锁

4）测试hashCode

调用对象的 hashCode方法，会把 hashCode添加到 mark word中，由于偏向锁没有空间存储 hashCode，会被禁用转向正常状态对象

```java
Dog d = new Dog();
d.hashCode();
synchronized(d) {}
```

#### 撤销-调用对象 hashCode

调用了对象的hashcode，但偏向锁的对象 mark word中存储的是线程id，如果调用 hashcode会导致偏向锁被撤销

- 轻量级锁的 hashCode会存在栈帧的锁记录里
- 重量级锁的 hashCode会存在 Monitor对象中

在调用 hashcode后使用偏向锁，需要去掉 `-XX:-UseBiasedLocking`

#### 撤销-其它线程使用对象

当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁

```java
public class TestBiased {
	private static void test2() throws InterruptedException {
    Dog d = new Dog();
    Thread t1 = new Thread(() -> {
      synchronized(d) {
        System.out.println(ClassLayout.parseInstace(d).toPrintableSimple(true));
      }
      System.out.println(ClassLayout.parseInstace(d).toPrintableSimple(true));

      // 防止线程们交错竞争锁，导致生成重量级锁
      synchronized(TestBiased.class) {
        TestBiased.class.notify();
      }
    }, "t1").start();
    
    Thread t2 = new Thread(() -> {
      synchronized(TestBiased.class) {
        try {
        	TestBiased.class.wait();
        } catch(InterruptedException e) {
          e.printStackTrace();
        }
      }

      System.out.println(ClassLayout.parseInstace(d).toPrintableSimple(true));
      synchronized(d) {
        System.out.println(ClassLayout.parseInstace(d).toPrintableSimple(true));
      }
      System.out.println(ClassLayout.parseInstace(d).toPrintableSimple(true));
    }, "t2").start();
  }
}
```

#### 撤销-调用 wait/notify

只有**重量级锁**才有 `wait/notify`，所以调用该方法会将**轻量级锁**和**偏向锁**升级为**重量级锁**

#### 批量重偏向

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID

当撤销偏向锁阈值超过 20次后，jvm会这样觉得，自己是不是偏向错了，于是会在给这些对象加锁时重新偏向至加锁线程

#### 批量撤销

当撤销偏向锁阈值超过40次后，jvm会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变成不可偏向的，新建的对象也是不可偏向的

### 5. 锁消除

锁消除

```java
@Fork(1)
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 3)
@Measurement(iterations = 5)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class MyBenchmark {
  static int x = 0;

  @Benchmark
  public void a() throws Exception {
    x++;
  }

  @Benchmark
  // JIT 即时编译器会分析热点代码，并优化代码
  // 逃逸分析 —> 锁消除
  public void b() throws Exception {
    Object o = new Object();
    synchronized(o) {
      x++;
    }
  }
}
```

```bash
# 默认打开 锁消除优化
java -jar benchmarks.jar

# 关闭 锁消除优化
java -XX:-EliminteLocks -jar benchmarks.jar
```





