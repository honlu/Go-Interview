# go并发编程

### Mutex几种状态

- mutexLocked 表示互斥锁的锁定状态
- mutexWoken 唤醒锁
- mutexStarving 当前互斥锁进入饥饿状态
- mutexWaiterShift 统计阻塞在这个互斥锁上的goroutine的数目

互斥锁无冲突是最简单的情况了，有冲突时，首先进行自旋，因为Mutex保护的代码段都很短

经过短暂的自旋就可以获得，如果自旋等待无果，就只好通过信号量让当前goroutine进入Gwaitting状态

### Mutex的正常模式和饥饿模式

- 正常模式（非公平锁）
  1. 正常模式下，所有等待的goroutine按照FIFO（先进先出）的顺序等待
  2. 唤醒的goroutine不会直接拥有锁而是会和新进来的请求goroutine竞争锁
  3. 新请求的goroutine更容易抢占，因为它正在CPU上执行，所以刚刚唤醒的goroutine很大可能会竞争失败
  4. 在这种情况下这个被唤醒的goroutine会被加入到等待队列的前面
- 饥饿模式（公平锁）
  1. 为了解决等待goroutine队列的长尾问题，饥饿模式下
  2. 直接由unlock把锁交给等待队列中第一个goroutine(队头),同时，饥饿模式下
  3. 新进来的goroutine不会进行抢锁，也不会进入自旋状态，会直接进入等待队列的尾部
  4. 这样很好的解决了老的goroutine一直抢不到锁的情况
- 饥饿模式的触发条件
  1. 当一个goroutine等待时间超过1毫秒时，或者当前队列剩下一个goroutine的时候，Mutex切换到饥饿模式
- 总结
  1. 对于两种模式，正常模式下性能是最好的，因为goroutine可以连续多次获取锁
  2. 饥饿模式解决了取锁公平的问题，但是性能会下降，这其实是性能和公平的一个平衡模式

### Mutex允许自旋的条件

1. 锁已被占用，且锁不处于饥饿模式
2. 积累的自旋次数小于最大自旋次数
3. CPU核数>1
4. 有空闲的P
5. 当前的goroutine所挂载的P下，本地待运行队列为空

### RWMutex实现

1. 通过记录readerCount读锁的数量来进行限制，当有一个写锁的时候，会将读锁的数量设置为负数1<<30
2. 目的是让新进来的读锁等待之前的写锁释放通知读锁
3. 同样当有写锁进行抢占时，也会等待之前的读锁都释放完毕，才进行后续的操作
4. 而等写锁释放完成之后，会将值重新加上1<<30，并通知刚才新进入的读锁（rw.readerSem）两者互相限制

### RWMutex注意事项

1. RWMutex是个单写多读锁，可以加多个读锁或者一个写锁
2. 读锁占用的情况下会阻止写，不会阻止读，多个goroutine可以同时获取读锁
3. 写锁会阻止其它goroutine（无论读写锁）进来，整个锁由该goroutine独占
4. 适用于读多写少的场景
5. RWMutex的零值是一个未锁定状态的互斥锁
6. RWMutex在首次使用之后就不能再被拷贝
7. RWMutex的读锁或写锁在未锁定的状态下进行解锁都会引发panic
8. RWMutex的一个写锁去锁定临界区的共享资源，如果临界区的资源已被（读锁或写锁）锁定，这个写锁的goroutine会被阻塞直到解锁
9. RWMutex的读锁不要用于递归调用，容易产生死锁
10. RWMutex的锁定状态与特定的goroutine没有关联，一个goroutine可以RLock(Lock),另外一个goroutine可以RUnlock(Unlock)
11. 写锁被解锁后，所有因操作锁定读锁的goroutine会被唤醒，并都可以成功锁定读锁
12. 读锁被解锁后，在没有其它读锁锁定的情况下，所有因操作锁定写锁而被阻塞的goroutine中，其中等待时间最长的goroutine会被唤醒

### cond是什么

1. Cond实现了一种条件变量，可以使用在多个reader等待共享资源ready的场景，（如果只有一读一写，一个锁或者channel就搞定了）
2. 每个Cond都会关联一个Lock(*sync.Mutex or *sync.RWMutex), 当修改条件或者调用Wait方法时，必须加锁以保护condition
3. 案例：

```go
var (
	c = sync.Cond{L: &sync.Mutex{}}  // 条件变量，多个reader等待共享资源ready的场景
	maxNum = 15
)
func main(){
	for i := 0; i < maxNum; i++{
		go func(i int) {
			c.L.Lock()
			defer c.L.Unlock()
			// Wait会释放c.L锁，并挂起调用者的goroutine，之后恢复执行
			c.Wait()  // Wait会在返回时对c.L加锁
			// 除非被Broadcast或Signal唤醒，否则Wait不会返回
			fmt.Println("goroutine:", i)
			time.Sleep(time.Millisecond * 100)
		}(i)
	}
	c.L.Lock()
	maxNum = 16
	c.L.Unlock()
	c.Broadcast()  // 广播，所有等待的goroutine都会执行
	//c.Signal()  // 单播，从所有等待的goroutine中随机找一个去执行
	time.Sleep(time.Second * 2)
}
```

### Broadcast和Signal的区别

- Broadcast会唤醒所有等待c的goroutine,调用Broadcast的时候，可以加锁也可以不加锁
- Signal只唤醒一个等待c的goroutine，调用Signal的时候，可以加锁也可以不加锁

### Cond中Wait使用

1. Wait会自动释放c.L锁，并挂起调用者的goroutine，之后恢复执行
2. Wait会在返回时对c.L加锁
3. 除非被Broadcast或Signal唤醒，否则Wait不会返回
4. 由于Wait第一次恢复是，c.L并没有加锁，所以当Wait返回时，调用者通常不能假设条件为真
5. 简单来说，只要想使用condition就必须加锁

### WaitGroup用法

- 一个WaitGroup对象可以等待一组协程结束，使用方法是

1. main协程通过调用 wg.add(delta int) 来设置worker协程的个数，然后创建worker协程
2. worker协程结束以后，都要调用wg.Done()
3. main协程调用wg.Wait()而被block, 知道所有worker协程全部执行结束后返回
4. 如果不确定要创建的worker协程数量，就不要一次性wg.add(),而是在每个创建worker协程之前调用一次wg.add(1)
5. 首次使用后不得复制wg

### WaitGroup实现原理

1. WaitGroup主要维护了两个计数器，一个请求计数器v,一个等待计数器w
2. 二者组成了一个64bit的值，请求计数器占高32bit,等待计数器占低32bit
3. 每次Add执行，请求计数器 v+1, 每次Done，等待计数器 w-1
4. 当v为0时，通过信号量唤醒Wait

### 什么是sync.Once

1. Once可以用来执行且仅仅执行一次的动作，常常用于单例对象的初始化场景
2. Once常常用来初始化单例资源，或者并发访问只需要初始化一次的共享资源
3. 或者在测试的时候初始化一次测试资源
4. sync.Once只暴露了一个方法Do,你可以多次调用Do方法
5. 但是只有第一次调用Do方法时f参数才会执行，这里的f是一个无参数无返回值的函数

### 什么操作叫原子操作

1. 原子操作即是进程过程中不能被中断的操作，针对某个值的原子操作在被进行的过程中
2. CPU绝不会再去进行其它针对值的操作，为了实现这样的严谨性，原子操作仅会由一个独立的CPU指令代表和完成
3. 原子操作是无锁的，常常直接通过CPU指令直接实现，
4. 事实上，其它同步技术的实现常常依赖于原子操作

### 原子操作和锁的区别

1. 原子操作由底层硬件支持，而锁由操作系统调度器实现
2. 锁应当用来保护一段逻辑，对于一个变量更新的保护
3. 原子操作通常执行上会更有效率，并且更能利用计算机多核资源
4. 如果要更新的是一个复合对象，则应当使用automic.Value封装好的实现

### sync.Pool有什么用

1. 对于很多需要重复分配、回收内存的地方，sync.Pool是一个很好的选择
2. 频繁的分配回收内存会给GC带来一定负担，严重的时候会引起CPU的毛刺
3. 而sync.Pool可以将暂时不使用的对象缓存起来，待下次需要的时候直接使用
4. 不用再次经过内存分配，复用对象的内存，减轻GC的压力，提升系统性能
5. 案例

```go
func main(){
	var p sync.Pool
	var a = "jdfakljfdalfaskdj阿加加加金灯送福卡萨发几块爱神的箭快疯了金阿奎"
	p.Put(a)
	ret := p.Get()
	fmt.Println(ret)
```

# Go Runtime

### 1. goroutine定义

```x86asm
golang在语言级别支持协程，称之为goroutine; golang标准库提供的所有系统调用操作（包括所有同步I/O操作）
都会让出CPU给其它goroutine, 这让goroutine的切换管理不依赖于系统的线程和进程，也不依赖于CPU的核心数量
而是交给Golang的运行时统一调度
```

### 2. GMP指的是什么

```markdown
* G（goroutine)，我们所说的协程，用户级的轻量级线程，每个goroutine对象中的sched保存着其上下文信息
* M（machine），对内核级线程的封装，数量对应真实的CPU数量（真正干活的对象）
* P（processor），即为G和M的调度对象，用来调度G和M之间的关联关系，其数量可通过GOMAXPROCS来设置，默认是CPU核心数量
```

### 3. 1.0之前GM调度模型

```css
调度器把不同的G分配到M上，不同的G在不同的M并发运行时，都需要向系统申请资源，比如堆栈内存，因为资源是全局的
就会因为资源竞争造成很多性能损耗，为了解决这一问题go从1.1版本引入，在运行时系统的时候加入p对象，
让P去管理G对象，M想要运行G，必须先绑定P，然后才能运行P下面的G对象

GM调度存在的问题：
    1. 单一全局互斥锁（Sched.Lock）和集中状态存储
    2. groutine传递问题（M经常在M之间传递可运行的goroutine）
    3. 每个M做内存缓存，导致内存占用过高，数据局部性较差
    4. 频繁的syscall调用, 导致严重的线程阻塞/解锁，家具额外的性能损耗_
```

### 4. GMP调度流程

```css
1. 每个p有个局部队列，局部队列放的是待执行的goroutine，当M绑定的P的局部队列满了以后就会把G放到全局队列
2. 每个P和一个M绑定，M是真正执行P中goroutine的实体，M从绑定的P的局部队列中获取G来执行
3. 当M绑定的P的局部队列为空时，M就会从全局队列获取到本地队列来执行，当全局队列也为空时，
   M就会从其它P队列偷取G来执行，这种从其它P偷的方式称之为 work stealing
4. 当G因系统调用(syscall)阻塞时会阻塞M，此时P会和M解绑即hand off，并寻找新的M，如果没有空闲的M就会新建一个M
5. 当G因channel或者network I/O操作阻塞时，不会阻塞M，M会寻找其它runable的G，当阻塞的G恢复后重新进入runable进入P队列等待执行
```

### 5. GMP中的work stealing机制

```css
先获取p本地队列，如果为空时，去全局队列里取G运行，如果全局队列为空时从netpoll和事件池里拿
如果在拿不到从其它P队列里偷
```

### 6. GMP中的handoff机制

```css
当本地线程M因为G进行的系统调用阻塞时，会释放绑定的P，把P转移给其它空闲的M执行
```

### 7. 协作式的抢占式调度

```markdown
在1.14之前，程序只能依靠goroutine主动让出CPU资源才能触发调用，这种方式存在问题有：
1. goroutine长时间占用线程，造成其它goroutine的饥饿
2. 垃圾回收需要暂停整个程序，最长可能几分钟，导致整个程序无法工作
```

### 8. 基于信号的抢占式调度

```css
1. 在任何情况下，go运行时并行执行的goroutine数量要小于等于p的数量
2. 为了提高性能，p的数量肯定不是越小越好，官方给出默认是CPU的核心数量
3. 如果设置过小的，当M绑定的P执行的G执行系统调用阻塞，导致M也阻塞时，GO的调度器是迟钝的，他有可能什么都不做
   它有可能什么都不做，知道M阻塞了相当长时间以后，才会发现一个P/M被syscall阻塞了，然后才会用空闲的M来抢这个P
所以P也不建议设置太小,通过sysmon监控实现的抢占式调度，最快在20us,最慢在10-20ms才会发现一个M持有P并阻塞了，
而操作系统1ms可以完成几十次系统调度
所以P适当的比CPU核心数多一些最好
```

### 9. GMP调度过程中存在哪些阻塞

```markdown
1. I/O, select
2. block on syscall
3. channel
4. 等待所
5. runtime.Gosched()
```

### 10. sysmon有什么用

```markdown
sysmon也叫监控线程，变动的周期性检查，好处
1. 释放超过5分钟的span物理内存
2. 如果超过两分钟没有垃圾回收，强制执行
3. 将长时间未处理的netpoll加入到全局队列
4. 向长时间运行的g任务发出抢占调度(超过10ms的g，会进行retake)
5. 收回因syscall阻塞的P
```