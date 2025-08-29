<h1 align = "center">多线程数据通信：packaged_task</h1>

## 一、定义

	#### std::packaged_task<> 会将 future 与可调用对象进行绑定。

##### 当调用 std::packaged_task<> 对象时，就会调用相关可调用对象，当 future 状态为就绪时，会存储返回值。



## 二、等价类

+ #### std::packaged_task 具有只移属性

```c++
// std::packaged_task<int(int)> 等价类
template<>
class packaged_task<int(int)>
{
public:
    template<typename Callable>
    explicit package_task(Callable&& f) //构造函数
    
    std::future<int> get_future(); // 内部封装了 std::future => 只可以移动不可以复制
    
    void operator()(int); // 重载了 () 操作符 => pakaged_task 对象可以直接调用，但是返回值是 void => 需要使用 get_futrue() 获取返回值。
}
```



## 三、使用方法

```c++
std::packaged_task<double(int,int)> task( [](int a, int b)->double {return std::pow(a,b);} );
std::future<double> future = task.get_future();

//	std::async(std::move(task)， 10, 2);

std::thread t{std::move(task), 10, 2}; //std::package_task 具有只移属性
t.join();

cout << future.get() << endl;
```



## 四、使用场景

+ #### 线程池

+ #### 任务队列

  