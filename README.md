##前言  
对协程库不怎么熟悉，分析下代码。  

##分析  

asyncio_pool 兼容3.4以上的版本。  
asyncio_pool下的init文件.由于小于3.6的python版本在工作中不常用，于是只分析3.7及其以上的协程池的实现  
AioPool继承MxAsyncGenPool, BaseAioPool两个类。  
### BaseAioPool  
####init
loop:事件循环类，_get_loop 可以返回兼容版本的eventloop。  
size:协程数量大小。  
_executed:是否在执行。  
_joined:集合中是future对象,用于存储执行协程返回的result。  
_waiting:未执行的任务。  
_active:正在执行的协程任务。  
semaphore:信号量和当前协程数相一致,同时又控制这协程的数量不能超过这个值。  
####aenter  
当类被创建的时候返回自身。  
####aexit  
当类被退出调用join方法。  
####join  
如果没有等待或者和正在执行的协程就会返回真，否则创建future对象等待协程执行完成返回结果。  
####release_joined
如果还有等待或者执行的协程，那么不会被释放抛出异常，否则人为的给这些future对象设置为true.  
