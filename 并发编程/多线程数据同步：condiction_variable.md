<h1 align = "center">多线程数据同步：条件变量</h1>

### 一、一个线程等待另一个线程完成：

+ #### 持续检查共享数据标记 

+ #### 使用 std::this_thread::sleep_for() 周期性检查共享数据标记

+ #### 使用条件变量等待事件发生

  

## 二、条件变量

+ ## <span style="color:#FF8000;">std::condition_variable</span> 

  #### 只能和 std::mutex 配合使用

  #### 比 std::condition_variable_any 更高效

+ ## <span style="color:#FF8000;">std::condition_variable_any</span>

	#### 可以和任何满足 Lockable 的锁一起使用
	
	#### 比如：std::unique_lock&lt;std::mutex&gt;、std::shared_lock&lt;std::shared_mutex&gt;、自定义锁

```c++
std::shared_mutex sm;
std::condition_variable_any cv;
bool ready = false;

void reader() 
{
    std::shared_lock<std::shared_mutex> lock(sm);
    cv.wait(lock, [] { return ready; });  // ✅ 可以和 shared_lock 一起用
    // 读数据
}

void writer() 
{
    std::unique_lock<std::shared_mutex> lock(sm);
    ready = true;
    cv.notify_all();
}
```



## 三、std::condition_variable

```c++
std::queue<int> dataQueue;
std::mutex mtx;
std::condition_variable cond;
std::atomic<bool> isComplete = false;

void Producer()
{	
	for (size_t i = 0; i < 5; i++)
	{
		size_t data = i * 10;
		std::this_thread::sleep_for(1s);

		//锁的粒度要尽可能的小，否则可能出现消费者饿死的情况
		std::unique_lock<std::mutex> locker(mtx); 
		dataQueue.push(data);
		locker.unlock();

		cond.notify_one(); //保护同一个共享资源的要使用同一个互斥锁
	}

	isComplete = true;
	cond.notify_one(); //通知消费者退出线程
}

void Consumer()
{
	while (true)
	{
		std::unique_lock<std::mutex> locker(mtx);	
		cond.wait(locker, []() { return !dataQueue.empty() || isComplete.load() == true; });

		if (dataQueue.empty() == false)
		{
			size_t data = dataQueue.front();
			dataQueue.pop();
		}
		else if (isComplete.load())
		{
			break;
		}			
	}
}
```



+ #### std::condition_variable::wait() 和 std::condition_variable_any::wait() 都不能使用 std::lock_guard。因为 std::lock_guard 不支持手动加锁解锁

+ #### std::condition_variable::wait() 会去检查条件谓词，当条件满足(返回true)时返回。如果条件不满足(返回false)，wait() 函数将解锁互斥量，并将这个线程置于阻塞或等待状态。

  

## 四、条件变量等待与唤醒的完整流程

### 1. 线程进入等待

+ ##### 线程先获取互斥锁

+ ##### 在线程持有锁的情况下调用 wait(lock, pred):

  + ##### wait 内部会自定解锁互斥量，让其它线程可以进入

  + ##### 线程进入条件变量的等待队列，状态变为休眠，不占 CPU


### 2. 线程发出唤醒通知

+ ##### 当条件发生变化，生产者调用 notify_one() 或 notify_all()

+ ##### 操作系统会把等待队列里的一个或多个线程标记为 [可运行]，即从 休眠队列 -> 就绪队列

### 3. 被唤醒线程的下一步

+ ##### 被唤醒的线程必须重新获取互斥锁 (因为在 wait 里已经释放了)

+ ##### 多个线程可能同时被唤醒，它们会竞争同一个锁

+ ##### 如果锁是空闲的，它立刻抢到锁，线程继续执行

+ ##### 如果锁被其它线程占用，未抢到锁的线程进入锁的等待队列，阻塞，直到锁被释放

### 4. 重新检查条件

+ ##### 被唤醒后拿到锁的线程，会重新检查 pred(条件谓词)

  + ##### 必须重新检查的原因是可能是虚假唤醒（虚假唤醒：系统可能会在没有 notify 的情况下唤醒线程）

+ ##### 如果条件满足，wait 函数返回，线程继续执行后续代码

+ ##### 如果条件不满足，再次调用 wait，重新释放锁并进入休眠

### 5. 执行完毕释放锁

+ ##### 线程执行完临界区代码后，离开作用域自动释放锁

+ ##### 其他等待该锁的线程可以获得执行机会

  
