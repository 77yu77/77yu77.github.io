# ArkTS简介

ArkTS是OpenHarmony优选的应用高级开发语言。ArkTS提供了声明式UI范式、状态管理支持等相应的能力，让开发者可以以更简洁、更自然的方式开发应用。

同时，它在保持TypeScript基本语法风格的基础上，进一步通过规范强化静态检查和分析，使得在程序运行之前的开发期能检测更多错误，提升代码健壮性，并实现更好的运行性能。详见[初识ArkTS语言](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/quick-start/arkts-get-started.md)。

ArkTS提供了标准内置对象，例如`Array`、`Map`、`TypedArray`、`Math`等，供开发者直接使用。另外，ArkTS也提供了语言基础类库，为应用开发者提供常用的基础能力，主要包含能力如下图所示。

**图1** ArkTS语言基础类库能力示意图

![arkts-commonlibrary](https://docs.openharmony.cn/doc_v4.1_1716677174/zh-cn/application-dev/arkts-utils/figures/arkts-commonlibrary.png)

#### 并发能力

提供[异步并发和多线程并发](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/arkts-utils/concurrency-overview.md)的能力。

- 支持Promise和async/await等标准的JS异步并发能力。
- TaskPool为应用程序提供一个多线程的运行环境，降低整体资源的消耗、提高系统的整体性能，开发者无需关心线程实例的生命周期。
- Worker支持多线程并发，支持Worker线程和宿主线程之间进行通信，开发者需要主动创建和关闭Worker线程。

**Promise**

Promise是一种用于处理异步操作的对象，可以将异步操作转换为类似于同步操作的风格，以方便代码编写和维护。Promise提供了一个状态机制来管理异步操作的不同阶段，并提供了一些方法来注册回调函数以处理异步操作的成功或失败的结果。

Promise有三种状态：pending（进行中）、fulfilled（已完成）和rejected（已拒绝）。Promise对象创建后处于pending状态，并在异步操作完成后转换为fulfilled或rejected状态。

最基本的用法是通过构造函数实例化一个Promise对象，同时传入一个带有两个参数的函数，通常称为executor函数。executor函数接收两个参数：resolve和reject，分别表示异步操作成功和失败时的回调函数。例如，以下代码创建了一个Promise对象并模拟了一个异步操作：

```
const promise: Promise<number> = new Promise((resolve: Function, reject: Function) => {
setTimeout(() => {
  const randomNumber: number = Math.random();
  if (randomNumber > 0.5) {
    resolve(randomNumber);
  } else {
    reject(new Error('Random number is too small'));
  }
}, 1000);
})
```

**async/await**

async/await是一种用于处理异步操作的Promise语法糖，使得编写异步代码变得更加简单和易读。通过使用async关键字声明一个函数为异步函数，并使用await关键字等待Promise的解析（完成或拒绝），以同步的方式编写异步操作的代码。

async函数是一个返回Promise对象的函数，用于表示一个异步操作。在async函数内部，可以使用await关键字等待一个Promise对象的解析，并返回其解析值。如果一个async函数抛出异常，那么该函数返回的Promise对象将被拒绝，并且异常信息会被传递给Promise对象的onRejected()方法。

下面是一个使用async/await的例子，其中模拟了一个异步操作，该操作会在3秒钟后返回一个字符串。

```
async function myAsyncFunction(): Promise<void> {
  const result: string = await new Promise((resolve: Function) => {
    setTimeout(() => {
      resolve('Hello, world!');
    }, 3000);
  });
  console.info(result); // 输出： Hello, world!
}

myAsyncFunction();
```

**TaskPool**

任务池（TaskPool）作用是为应用程序提供一个多线程的运行环境，降低整体资源的消耗、提高系统的整体性能，且您无需关心线程实例的生命周期。具体接口信息及使用方法详情请见[TaskPool](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-taskpool.md)。

![TaskPool](https://docs.openharmony.cn/doc_v4.1_1716677174/zh-cn/application-dev/arkts-utils/figures/taskpool.png)

TaskPool支持开发者在主线程封装任务抛给任务队列，系统选择合适的工作线程，进行任务的分发及执行，再将结果返回给主线程。

**Worker**

Worker主要作用是为应用程序提供一个多线程的运行环境，可满足应用程序在执行过程中与主线程分离，在后台线程中运行一个脚本操作耗时操作，极大避免类似于计算密集型或高延迟的任务阻塞主线程的运行。

![worker](https://docs.openharmony.cn/doc_v4.1_1716677174/zh-cn/application-dev/arkts-utils/figures/worker.png)

Worker子线程和宿主线程之间的通信是基于消息传递的，Worker通过序列化机制与宿主线程之间相互通信，完成命令及数据交互。

**TaskPool与Worker对比**

TaskPool和Worker均支持多线程并发能力。由于TaskPool的工作线程会绑定系统的调度优先级，并且支持负载均衡（自动扩缩容），而Worker需要开发者自行创建，存在创建耗时以及不支持设置调度优先级，故在性能方面使用TaskPool会优于Worker，因此大多数场景推荐使用TaskPool。

TaskPool偏向独立任务维度，该任务在线程中执行，无需关注线程的生命周期，超长任务（大于3分钟）会被系统自动回收；而Worker偏向线程的维度，支持长时间占据线程执行，需要主动管理线程生命周期。

常见的一些开发场景及适用具体说明如下：

- 运行时间超过3分钟（不包含Promise和async/await异步调用的耗时，例如网络下载、文件读写等I/O任务的耗时）的任务。例如后台进行1小时的预测算法训练等CPU密集型任务，需要使用Worker。
- 有关联的一系列同步任务。例如在一些需要创建、使用句柄的场景中，句柄创建每次都是不同的，该句柄需永久保存，保证使用该句柄进行操作，需要使用Worker。
- 需要设置优先级的任务。例如图库直方图绘制场景，后台计算的直方图数据会用于前台界面的显示，影响用户体验，需要高优先级处理，需要使用TaskPool。
- 需要频繁取消的任务。例如图库大图浏览场景，为提升体验，会同时缓存当前图片左右侧各2张图片，往一侧滑动跳到下一张图片时，要取消另一侧的一个缓存任务，需要使用TaskPool。
- 大量或者调度点较分散的任务。例如大型应用的多个模块包含多个耗时任务，不方便使用8个Worker去做负载管理，推荐采用TaskPool。

对于CPU密集型任务：

CPU密集型任务是指需要占用系统资源处理大量计算能力的任务，需要长时间运行，这段时间会阻塞线程其它事件的处理，不适宜放在主线程进行。例如图像处理、视频编码、数据分析等。

基于多线程并发机制处理CPU密集型任务可以提高CPU利用率，提升应用程序响应速度。

当任务不需要长时间（3分钟）占据后台线程，而是一个个独立的任务时，推荐使用TaskPool，反之推荐使用Worker

为什么是3分钟，因为**TaskPool**的任务执行时长上限为3分钟

比如：

```
import taskpool from '@ohos.taskpool';

@Concurrent
function imageProcessing(dataSlice: ArrayBuffer): ArrayBuffer {
  // 步骤1: 具体的图像处理操作及其他耗时操作
  return dataSlice;
}

function histogramStatistic(pixelBuffer: ArrayBuffer): void {
  // 步骤2: 分成三段并发调度
  let number: number = pixelBuffer.byteLength / 3;
  let buffer1: ArrayBuffer = pixelBuffer.slice(0, number);
  let buffer2: ArrayBuffer = pixelBuffer.slice(number, number * 2);
  let buffer3: ArrayBuffer = pixelBuffer.slice(number * 2);

  let group: taskpool.TaskGroup = new taskpool.TaskGroup();
  group.addTask(imageProcessing, buffer1);
  group.addTask(imageProcessing, buffer2);
  group.addTask(imageProcessing, buffer3);

  taskpool.execute(group, taskpool.Priority.HIGH).then((ret: Object) => {
    // 步骤3: 结果数组汇总处理
  })
}
```

对于I/O密集型任务：

I/O密集型任务的性能重点通常不在于CPU的处理能力，而在于I/O操作的速度和效率。这种任务通常需要频繁地进行磁盘读写、网络通信等操作。推荐使用taskpool

而对于同步任务：

同步任务是指在多个线程之间协调执行的任务，其目的是确保多个任务按照一定的顺序和规则执行，例如使用锁来防止数据竞争。

同步任务的实现需要考虑多个线程之间的协作和同步，以确保数据的正确性和程序的正确执行。

由于TaskPool偏向于单个独立的任务，因此当各个同步任务之间相对独立时推荐使用TaskPool，例如一系列导入的静态方法，或者单例实现的方法。如果同步任务之间有关联性，则需要使用Worker，例如无法单例创建的类对象实现的方法。

#### 高性能容器类库

容器类库，用于存储各种数据类型的元素，并具备一系列处理数据元素的方法，作为纯数据结构容器来使用具有一定的优势。

容器类采用了类似静态语言的方式来实现，并通过对存储位置以及属性的限制，让每种类型的数据都能在完成自身功能的基础上去除冗余逻辑，保证了数据的高效访问，提升了应用的性能。

当前提供了线性和非线性两类容器，共14种。

**线性容器**

线性容器实现能按顺序访问的数据结构，其底层主要通过数组实现，包括ArrayList、Vector、List、LinkedList、Deque、Queue、Stack七种。

线性容器，充分考虑了数据访问的速度，运行时（Runtime）通过一条字节码指令就可以完成增、删、改、查等操作。

[ArrayList](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-arraylist.md)即动态数组，可用来构造全局的数组对象。 当需要频繁读取集合中的元素时，推荐使用ArrayList，其要求存储位置是一片连续的内存空间，初始容量大小为10，并支持动态扩容，每次扩容大小为原始容量的1.5倍。

[Vector](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-vector.md)是指连续存储结构，可用来构造全局的数组对象。Vector依据泛型定义，要求存储位置是一片连续的内存空间，初始容量大小为10，并支持动态扩容，每次扩容大小为原始容量的2倍。

Vector和[ArrayList](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-arraylist.md)相似，都是基于数组实现，但Vector提供了更多操作数组的接口。Vector在支持操作符访问的基础上，还增加了get/set接口，提供更为完善的校验及容错机制，满足用户不同场景下的需求。

[List](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-list.md)可用来构造一个单向链表对象，即只能通过头结点开始访问到尾节点。List依据泛型定义，在内存中的存储位置可以是不连续的。

[LinkedList](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-linkedlist.md)可用来构造一个双向链表对象，可以在某一节点向前或者向后遍历List。LinkedList依据泛型定义，在内存中的存储位置可以是不连续的。

[Deque](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-deque.md)可用来构造双端队列对象，存储元素遵循先进先出以及先进后出的规则，双端队列可以分别从队头或者队尾进行访问。

[Queue](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-queue.md)可用来构造队列对象，存储元素遵循先进先出的规则。

**非线性容器**

非线性容器实现能快速查找的数据结构，其底层通过hash或者红黑树实现，包括HashMap、HashSet、TreeMap、TreeSet、LightWeightMap、LightWeightSet、PlainArray七种。

#### XML、URL、URI构造和解析的能力

- XML被设计用来传输和存储数据，是一种可扩展标记语言。语言基础类库提供了[XML生成、解析与转换](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/arkts-utils/xml-overview.md)的能力。
- URL、URI构造和解析能力：其中[URI](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-uri.md)是统一资源标识符，可以唯一标识一个资源。[URL](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-url.md)为统一资源定位符，可以提供找到该资源的路径。

#### 常见的字符串和二进制数据处理，以及控制台打印

- 字符串编解码功能。
- 基于Base64的字节编码和解码功能。
- 提供常见的有理数操作支持，包括有理数的比较、获取分子分母等功能。
- 提供Scope接口用于描述一个字段的有效范围。
- 提供二进制数据处理的能力，常见于TCP流或文件系统操作等场景中用于处理二进制数据流。
- Console提供控制台打印的能力。

- 提供[获取进程信息和操作进程](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/reference/apis-arkts/js-apis-process.md)的能力。



### ArkTS与java和ts的对比

#### 并发与多线程方面：

java多线程通过new thread来创建多线程，提供线程类、同步机制、锁等。

锁机制比如synchronized 互斥锁，一次只能允许一个线程进入被锁住的代码块，它也是一种内置锁/监视器锁，Java 中每个对象都有一个内置锁（监视器,也可以理解成锁标记），而 synchronized 就是使用对象的内置锁（监视器）来将代码块/方法锁定的，还有自旋锁等。

而await语法糖用于锁机制中，用于释放当前的锁，：通过signal或者signalAll方法唤醒await挂起的线程。

|                      | await/wait       | Sleep      | Yield          |
| -------------------- | ---------------- | ---------- | -------------- |
| **是否释放持有的锁** | 释放             | 不释放     | 不释放         |
| **调用后何时恢复**   | 唤醒后进入就绪态 | 指定时间后 | 立刻进入就绪态 |
| **谁的方法**         | Condition/Object | Thread     | Thread         |
| **执行环境**         | 同步代码块       | 任意位置   | 任意位置       |

**java提供线程池来管理线程**：

java语言虽然内置了多线程支持，启动一个新线程非常方便，但是，创建线程需要操作系统资源（线程资源，栈空间等），频繁创建和销毁大量线程需要消耗大量时间。

如果可以复用一组线程：

```ascii
┌─────┐ execute  ┌──────────────────┐
│Task1│─────────>│ThreadPool        │
├─────┤          │┌───────┐┌───────┐│
│Task2│          ││Thread1││Thread2││
├─────┤          │└───────┘└───────┘│
│Task3│          │┌───────┐┌───────┐│
├─────┤          ││Thread3││Thread4││
│Task4│          │└───────┘└───────┘│
├─────┤          └──────────────────┘
│Task5│
├─────┤
│Task6│
└─────┘
  ...
```

那么就可以把很多小任务让一组线程来执行，而不是一个任务对应一个新线程。这种能接收大量小任务并进行分发处理的就是线程池。

简单地说，线程池内部维护了若干个线程，没有任务的时候，这些线程都处于等待状态。如果有新任务，就分配一个空闲线程执行。如果所有线程都处于忙碌状态，新任务要么放入队列等待，要么增加一个新线程进行处理。



而typescript是单线程的，因此实现多线程的唯一方法是启动 JS 引擎的多个实例。但是，而实现在这些实例之间进行通信，这就是Web Workers的用武之地。

> Web Workers 使得一个 Web 应用程序可以在与主执行线程分离的后台线程中运行一个脚本操作。这样做的好处是可以在一个单独的线程中执行费时的处理任务，从而允许主（通常是 UI）线程运行而不被阻塞。
>
> 它的作用就是给 JS 创造多线程运行环境，允许主线程创建 worker 线程，分配任务给后者，主线程运行的同时 worker 线程也在运行，相互不干扰，在 worker 线程运行结束后把结果返回给主线程。这样做的好处是主线程可以把计算密集型或高延迟的任务交给 worker 线程执行，这样主线程就会变得轻松，不会被阻塞或拖慢。这并不意味着 JS 语言本身支持了多线程能力，而是浏览器作为宿主环境提供了 JS 一个多线程运行的环境。

Web Workers 使在与 Web 应用程序的主执行线程分离的后台线程中运行脚本操作成为可能在 Web Worker 的帮助下，可以将繁重的计算卸载到单独的线程，从而释放主线程。这些工作线程和主线程使用事件进行通信，一个工作线程可以产生其他工作线程。
无论是 web worker 还是 worker 线程，它们都不像其他语言中的多线程实现那样灵活或简单，并且有很多限制，因此它们大多只在有 CPU 密集型任务或后台任务需要执行以供其他用途时使用使用异步处理的并发情况就足够了。

ts 提供spawn()和expose()以及worker等

```
// master.ts
import { spawn, Thread, Worker } from "threads"
import { Counter } from "./workers/counter"

const counter = await spawn<Counter>(new Worker("./workers/counter"))
console.log(`Initial counter: ${await counter.getCount()}`)

await counter.increment()
console.log(`Updated counter: ${await counter.getCount()}`)

await Thread.terminate(counter)
```

`spawn` 函数用于从 `"./workers/counter"` 文件实例化工作线程，而worker是一个工作线程对象。

`await` 关键字与异步操作结合使用。它允许主线程暂停执行，直到异步操作完成。`await` 关键字与异步操作结合使用。它允许主线程暂停执行，直到异步操作完成。`spawn` 是一个异步函数，它创建一个工作线程并返回一个 Promise，该 Promise 解析为对工作线程公开函数的引用。`await` 关键字确保主线程在继续之前等待 Promise 解析。

#### 容器类库对比

**java容器类库**

Java 提供了丰富的数据结构来处理和组织数据，有：

1. 数组（Arrays）

2. 列表（Lists）

3. 映射（Maps）

4. 栈（Stack）

5. 队列（Queue）

6. 堆（Heap）

7. 树（TreeNode ）

8. 枚举（Enumeration）

9. 位集合（BitSet）

10. 向量（Vector）

11. 哈希表（Hashtable）


![img](https://www.runoob.com/wp-content/uploads/2014/01/2243690-9cd9c896e0d512ed.gif)

从上面的集合框架图可以看到，Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。Collection 接口又有 3 种子类型，List、Set 和 Queue，再下面是一些抽象类，最后是具体实现类，常用的有 [ArrayList](https://www.runoob.com/java/java-arraylist.html)、[LinkedList](https://www.runoob.com/java/java-linkedlist.html)、[HashSet](https://www.runoob.com/java/java-hashset.html)、LinkedHashSet、[HashMap](https://www.runoob.com/java/java-hashmap.html)、LinkedHashMap 等等。

集合框架是一个用来代表和操纵集合的统一架构。所有的集合框架都包含如下内容：

- **接口：**是代表集合的抽象数据类型。例如 Collection、List、Set、Map 等。之所以定义多个接口，是为了以不同的方式操作集合对象
- **实现（类）：**是集合接口的具体实现。从本质上讲，它们是可重复使用的数据结构，例如：ArrayList、LinkedList、HashSet、HashMap。
- **算法：**是实现集合接口的对象里的方法执行的一些有用的计算，例如：搜索和排序，这些算法实现了多态，那是因为相同的方法可以在相似的接口上有着不同的实现。

![img](https://www.runoob.com/wp-content/uploads/2014/01/java-coll-2020-11-16.png)

**ts容器类库**

ts提供Array数组以及Map对象

特殊的，ts提供元组类型，允许存储不同类型的元素，元组可以作为参数传递给函数，比如

```
var mytuple = [10,"Runoob"];
```

ts还提供联合类型（Union Types）可以通过管道(|)将变量设置多种类型，赋值时可以根据设置的类型来赋值。

```
var val:string|number 
val = 12 
console.log("数字为 "+ val) 
val = "Runoob" 
console.log("字符串为 " + val)
```

相对于ArkTS，java提供的容器类库更加丰富，且分类的依据不同，而ts提供的就少得可怜。

