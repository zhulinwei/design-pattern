# Singletion Pattern
单例模式
- [Singletion Pattern](#singletion-pattern)
  - [概念](#概念)
  - [使用场景](#使用场景)
  - [实现方式](#实现方式)
    - [饿汉式](#饿汉式)
      - [示例](#示例)
        - [Java Sample](#java-sample)
        - [Golang Samle](#golang-samle)
      - [总结反思](#总结反思)
    - [懒汉式](#懒汉式)
      - [示例](#示例-1)
        - [Java Sample](#java-sample-1)
        - [Golang Sample](#golang-sample)
      - [总结反思](#总结反思-1)
    - [双重检测](#双重检测)
      - [示例](#示例-2)
        - [Java Sample](#java-sample-2)
        - [Golang Sample](#golang-sample-1)
      - [总结反思](#总结反思-2)
  - [类图](#类图)
  - [总结反思](#总结反思-3)

## 概念
一个类只允许创建唯一一个对象/实例

唯一：这里指的是进程内唯一（意味着线程内和线程间都唯一），但进程间不是唯一的；

## 使用场景
适合表示全局唯一类的场景，如系统的配置信息，以及处理资源访问冲突等（只有一个实例在工作，避免冲突问题）；

## 实现方式

### 饿汉式
在类加载期间即完成实例的初始化，优点是线程安全，缺点是不支持延迟加载；

#### 示例

##### Java Sample

<details>

```java
public class Singletion {
    pirvate static Singletion single = new Singletion();
    private Singletion() {}
    public static Singletion getSingle () {
        return single;
    }
}
```

</details>

##### Golang Samle

<details>

```golang
type Singletion struct {}

var single *Singletion

func GetSingle() *Singletion {
    return single
}

func init() {
    single = new(Singletion)
}
```
</details>

#### 总结反思

反方观点：不支持延迟加载，如果实例占用资源多（比如内存）或者耗时长（需要加载各种配置文件呢），提前初始化是一种资源的浪费，最好的做法是等到要用的时候再去初始化；

正方观点：如果占用资源多，最好在初始化时暴露问题，以免在运行时发生导致系统奔溃，影响系统的可用性；如果初始化耗时长，最要不要等到真正要用的时候才去执行这个初始化过程，以免影响到系统的性能；

### 懒汉式
就是在创建对象时比较懒，只有在需要时才会创建对象。相比饿汉式支持延迟加载，但他的实现方式会导致频繁地加锁、解锁，从而因并发度低产生性能问题；

#### 示例

##### Java Sample
<details>

```java
public class Singletion {
    private static Singletion single = null
    private Singletion () {}
    public static synchronized Singletion getSingle () {
        if (single == null) {
            single = new Singletion();
        }
        return single;
    }
}
```

</details>

##### Golang Sample

<details>

```golang
type Singletion struct {}

var mutex sync.Mutex
var single *Singletion

func GetSingle() *Singletion {
    mutex.Lock() 
    defer mutex.Unlock()
    if single == nil {
        single = new(Singletion)
    }
    return single
}
```

</details>

#### 总结反思

缺点：在Java Sample中我们可以看到synchronzed这把大锁，它的存在会导致函数的并发度很低，如果这个函数被频繁使用，那么频繁的加锁、解锁就会导致性能问题；

如果不加锁会存在什么问题呢？
在多线程的情况下，这样写可能会导致single有多个实例。比如下面这种情况，考虑有两个线程同时调用GetSingle函数：

| Time | Thread A | Thread B |
| - | - | - |
| T1 | 检查到single为空||
| T2 | | 检查到single为空 |
| T3 | | 初始化对象A |
| T4 | | 返回对象A |
| T5 | 初始化对象B | |
| T6 |  返回对象B | |

可以看到，single被实例化了两次并且被不同对象持有。完全违背了单例的初衷。

### 双重检测
即常说的Doublue checked，饿汉式不支持延迟加载，懒汉式有性能问题，不支持高并发，而双重检测是对他们一种优化：先判断对象是否已经被初始化，再决定要不要加锁。

#### 示例
##### Java Sample

<details>

```java
public class Singletion {
    private volatile static Singletion single = null
    private Singletion () {}
    public static synchronized Singletion getSingle () {
        if (single == null) {
            synchronized(Singletion.class) {
                if (single == null) {
                    single = new Singletion();
                }
            }
        }
        return single;
    }
}
```
</details>

##### Golang Sample

<details>

```golang
type Singletion struct {}

var mutex sync.Mutex
var single *Singletion

func GetSingle() *Singletion {
    if single == nil {
        mutex.Lock() 
        defer mutex.Unlock()
        if single == nil {
            single = new(Singletion)
        }    
    }
    return single
}
```
</details>

#### 总结反思
第一个检查是为了避免频繁加锁问题：前面的请求创建好实例后，后面的请求就不需要再加锁处理了；

第二个检查是为了处理锁竞争的情况：如果来个两个线程同时发现single为空，于是开始抢锁，由于前面抢锁成功的线程已经初始化完成示例，所以后面获得锁的线程在判断single不为空后就可以退出了；

补充：在Golang中，sync模块的Once方法已经实现了双重检测的机制，因此我们可以直接调用它：

<details>

```goalng
type Singletion struct {}

var once sync.Once
var single *Singletion

func GetSingle() *Singletion {
    once.Do(func() {
        single = new(Singletion)
    })
    return single
}
```

</details>

我们来看一下Once的源码：

<details>

```golang
type Once struct {
    m    Mutex
    done uint32
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 1 { // <-- Check
        return
    }
    // Slow-path.
    o.m.Lock()                           // <-- Lock
     defer o.m.Unlock()
    if o.done == 0 {                     // <-- Check
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}
```

</details>

可以发现Once的做法和语义和双重检测一致。

## 类图
单例模式一般可以用以下类图表示：

![singletion](singletion.png)

## 总结反思
反思单元模式存在的问题：
1. 单例模式不需要显示的创建，在函数中直接调用就可以使用，容易隐藏类之间的依赖关系，从可读性的角度来讲，我们通常是希望能一眼就看出类与类之间的依赖关系；
2. 对代码的可测行不好，单例通常以硬编码的形式使用，如果单例依赖比较重的外部资源时，导致不易被mock掉（可以通过依赖注入等方式解决这个问题）；
3. 对参数传递的支持不友好，比如我们创建一个连接池时，没法很好地通过参数来指定连接池大小（有三种思路：1是增加init函数，通过init传参再初始化single对象；2是在获取single对象时传递参数；3是设置全部变量，然后single直接读取）；
