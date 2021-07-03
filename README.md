## 前言  
对协程库不怎么熟悉，分析下代码。  

## 分析  

asyncio_pool 兼容3.4以上的版本。  
asyncio_pool下的init文件.由于小于3.6的python版本在工作中不常用，于是只分析3.7及其以上的协程池的实现  
AioPool继承MxAsyncGenPool, BaseAioPool两个类。  
### BaseAioPool  
#### init
loop:事件循环类，_get_loop 可以返回兼容版本的eventloop。  
size:协程数量大小。  
_executed:是否在执行。  
_joined:集合中是future对象,用于存储执行协程返回的result。  
_waiting:未执行的任务。  
_active:正在执行的协程任务。  
semaphore:信号量和当前协程数相一致,同时又控制这协程的数量不能超过这个值。  
#### aenter  
当类被创建的时候返回自身。  
#### aexit  
当类被退出调用join方法。  
#### join  
如果没有等待或者和正在执行的协程就会返回真，否则创建future对象等待协程执行完成返回结果。  
#### release_joined
如果还有等待或者执行的协程，那么不会被释放抛出异常，否则人为的给这些future对象设置为true.  
#### spawn  
等待协程池被完全释放之后才会放入新的执行对象。  
创建一个furture对象，然后在等waiting中创建一个future对象(作为一个占位符)，coro指代的是要被执行的任务  
cb是回调方法，ctx是上下文对象,然后调用_spawn，首先获得信号量的锁，然后等待未来对象返回(如果返回了就证明协程池里面的结果被执行完了)，。  
##### 1.首先创建future对象  
##### 2.给类的等待一个future对象  

##### 3.调用spawn  
首先获取锁，锁限制了协程的数量，以至于同时执行的协程数不会超过这个值。  
获得锁以后证明可以被执行了，那么从等待中移除掉这个future。  
加一个判断，如果future 被设置值（也就是被releadse_join）后就会被释放不执行。  
如果没有future被设置值那么就会赋值一个wrapped（装饰器函数），这个装饰器函数执行的是把coro执行掉  
然后给future对象设置值，并且给executed被执行量+1，然后从正在活跃的_active中进行删除  
但是这里_active[future]  =task,其实也就是 future 执行完之后就把active的当前future对象删除了。  

#### spawn_n
原理和spawn相似，唯一的不同就是 future返回的快慢。这个返回的比较慢，但是执行的比较快，



