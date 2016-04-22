##Pthread Basic

###[POSIX Threads Programming](https://computing.llnl.gov/tutorials/pthreads/)

###Thread
```
// 创建线程
pthread_create (thread,attr,start_routine,arg)
// 退出线程
pthread_exit (status)
// 在别的线程中通过id取消线程
pthread_cancel (thread)
// 线程的attr
pthread_attr_init (attr)
// 销毁线程的attr
pthread_attr_destroy (attr)
```

```
// 等带某一线程完成后再运行
pthread_join (threadid,status)
// 显式地detach一个线程，就是不join
pthread_detach (threadid)
// 设置detach的state，就是join或者detach
pthread_attr_setdetachstate (attr,detachstate)
// 获取detachstate
pthread_attr_getdetachstate (attr,detachstate)
```

```
// 线程栈的设置
pthread_attr_getstacksize (attr, stacksize)
pthread_attr_setstacksize (attr, stacksize)
pthread_attr_getstackaddr (attr, stackaddr)
pthread_attr_setstackaddr (attr, stackaddr)
```
```
// 获取当前线程的id
pthread_self ()
// 通过线程id来判断是否相等
pthread_equal (thread1,thread2)
// 只运行一次的用于初始化的线程
pthread_once (once_control, init_routine)
```
###Mutex
```
// 初始化一个互斥锁
pthread_mutex_init (mutex,attr)
// 销毁互斥锁
pthread_mutex_destroy (mutex)
// 互斥锁的属性
pthread_mutexattr_init (attr)
pthread_mutexattr_destroy (attr)

// 线程是获取锁，可能造成block
pthread_mutex_lock (mutex)
// 尝试获取锁，不block
pthread_mutex_trylock (mutex)
// 解锁
pthread_mutex_unlock (mutex)
```
###Condition Variables
```
pthread_cond_init (condition,attr)
pthread_cond_destroy (condition)
pthread_condattr_init (attr)
pthread_condattr_destroy (attr)

// 与mutex一起使用，wait的时候要先lock住mutex，随后会自动解锁
// 等待信号过来后，会自动获取mutex
pthread_cond_wait (condition,mutex)
// 唤醒用了cond的wait的至少一个线程 
pthread_cond_signal (condition)
// 广播唤醒所有wait了同一个cond的线程
pthread_cond_broadcast (condition)
```