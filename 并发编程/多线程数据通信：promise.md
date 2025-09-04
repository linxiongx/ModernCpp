<h1 align = "center">多线程数据通信：promise</h1>

```c++
	std::promise<int> prom;

	std::future<int> fut = prom.get_future();

	std::thread t([prom = std::move(prom)]()mutable {
		try
		{
			throw std::logic_error("一个异常");
		}
		catch (...)
		{
			prom.set_exception(std::current_exception());
			return;
		}
		std::this_thread::sleep_for(1s);
		prom.set_value(42);

	});

	
	t.join();

	try
	{
		cout << fut.get() << endl;
	}
	catch (std::exception e)
	{
		cout << e.what() << endl;
	}
```

#### <span style="background:#FF8000; color:#000000;">每个`promise`只能设置一次（要么`set_value`，要么`set_exception`）</span>
#### 一旦设置，再次设置会抛出`std::future_error`异常





### 只用future就可以实现使用 `future+promise` 的功能(获取值或者异常)，为什么还需要 `promise` ?

#### 答：`std::promise` 主要在这些场景下才必需：

+ ##### 1. 回调函数中设置结果

```c++
// 假设有个异步 API 只支持回调
void async_api_with_callback(std::function<void(int)> callback);

// 这种情况下必须用 promise
std::promise<int> prom;
auto fut = prom.get_future();

async_api_with_callback([&prom](int result) {
    prom.set_value(result);  // 在回调中设置值，只能用 promise
});

int result = fut.get();
```

+ ##### 2. 条件控制何时设置结果

```c++
std::promise<std::string> prom;
auto fut = prom.get_future();

// 在某个事件发生时才设置结果
if (user_clicks_button()) {
    prom.set_value("Button clicked");
} else if (timeout_occurred()) {
    prom.set_exception(std::make_exception_ptr(std::runtime_error("Timeout")));
}

std::string result = fut.get();
```

+ ##### 3. 在类成员函数中跨方法设置结果

```c++
class AsyncOperation {
    std::promise<int> prom;
    std::future<int> fut;
    
public:
    AsyncOperation() : fut(prom.get_future()) {}
    
    void start() {
        // 启动异步操作
        std::thread([this]() {
            // ... 复杂的异步逻辑
            on_complete(42);
        }).detach();
    }
    
    void on_complete(int result) {
        prom.set_value(result);  // 必须用 promise
    }
    
    std::future<int>& get_future() { return fut; }
};
```

#### 总结：

- #### 大多数情况下，使用 `std::async` 或 `std::packaged_task` 只需要处理 `std::future` 就够了

- #### `std::promise` 只在需要手动控制"何时"和"在哪里"设置结果的场景下才必需

- #### 如果您的异步操作可以封装成一个函数或可调用对象，那么 `std::async` 和 `std::packaged_task` 确实更简单

  #### `std::promise` 更像是一个"低级"的工具，用于构建其他异步抽象，而不是日常开发的首选。

