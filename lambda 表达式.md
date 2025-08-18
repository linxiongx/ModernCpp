<h1 align='center'>lambda 表达式</h1>

## 一、显式捕获、隐式捕获、值捕获、引用捕获

```c++
   auto lambda = [捕获列表](参数列表) -> 返回类型 { 函数体 };
```

```c++
    int nValue = 10, kkkk = 20;

    auto fun1 = [=]()->int { cout << "隐式值捕获, 捕获所有对象" << endl; return kkkk; };
    auto fun2 = [nValue]()->void { cout << "显式值捕获, 捕获指定对象" << endl; };
    auto fun3 = [&]() { cout << "隐式引用捕获, 捕获所有对象" << endl; kkkk; };
    auto fun4 = [&nValue]() { cout << "显式引用捕获, 捕获指定对象" << endl; };
    auto fun5 = [=, &nValue] { cout << "默认按值捕获所有变量，nValue 按引用捕获" << endl; };
    auto fun6 = [&, nValue] { cout << "默认按引用捕获所有变量，nValue 按值捕获" << endl; };
```



## 二、lambda 表达式默认使用 const 修饰

```c++
	auto lambda = [捕获列表](参数列表) const -> 返回类型 { 函数体 };
```

+ #### 不能改变值捕获的变量的值

  ```c++
  int value = 10;
  auto func = [value]() 
  {
  	value = 20; //error：无法在非可变 lambda 中修改,通过复制捕获
  };
  
  //使用 mutable 修饰
  int value = 10;
  auto func = [value]() mutable
  {
  	value = 20; //ok
  };
  ```

+ #### 可以改变引用捕获的变量的值

  ```c++
  int value = 10;
  auto func = [&value]() 
  {
  	value = 20; //ok
  };
  ```

  

+ #### lambda 表达式等价类

  + #### 值传递
    ```c++
    int value = 10;
    auto func = [value]() 
    {
    	value = 100; // error
    };
    // 等价于
    class _Lambda_1
    {
    private:
    	int value;
    public:
        _Lambda(int v): value(v){}
        void operation()() const
        {
            value = 100; // error
        }
    };
  
  + #### 引用传递
  
    ```c++
    int value = 10;
    auto func = [&value]() 
    {
    	value = 100;
    };
    // 等价于
    class _Lambda_2
    {
    private:
    	int& value; //声明一个引用
    public:
        _Lambda(int& v): value(v){} //引用必须在创建对象时立即绑定到一个对象
        void operation()() const
        {
            value = 100; // 可以修改
        }
    };
    ```
  
  + #### const 成员函数可以修改指针(引用)成员的所指向的值，不可以修改指针(引用)成员的指向
  
    ```c++
    class MyClass 
    {
        int* m_pValue; 
        int m_nValue;
        
    public:
        void test() const 
        {
            m_nValue = 100; //error!
            
            // const 修饰 this 指针
            // const MyClass* pThis; //不可以修改 pThis 指向的值。可以修改 pThis 的指向
            
            // pThis->m_pValue; //不可以修改 pThis 指向的值（m_pValue）
            this->m_pValue = new int(10); //error !
            
            // pTHis->m_pValue->value; //可以修改 (m_pValue->value)
            *(this->m_pValue) = 100;
        }
    };
    ```
  
    


## 三、避免使用默认捕获，尽可能的使用显式捕获



## 四、谨慎使用引用捕获

####              使用引用捕获的情况下，如果在该引用的对象被销毁后，调用该 lambda 表达式，会导致未定义行为。



## 五、小心对指针的值捕获

####                按值捕获了一个指针以后，在lambda式创建的闭包中持有的是这个指针的副本，但你并无办法阻止 lambda 式之外的代码去针对该 指针实施 delete 操作所导致的指针副本空悬。   	

```c++
    int* pInt = new int(19);
    auto fun7 = [pInt] { cout << "pInt 按值捕获，pInt 可能已经释放!" << *pInt << endl; };
    delete pInt; pInt = nullptr;
    fun7();
```



## 六、初始化捕获

+ #### 可以实现只移对象的捕获

+ #### = 的左右左右两侧处于不同的作用域。

  #### 	左侧的作用域就是闭包类的作用域，右侧的作用域与 lambda 式加以定义之处的作用域相同。

  ```c++
  auto func = [pw = std::make_unique<Widget>()] ()
  {
      return pw->isValidated();
  }
  ```

  ```c++
  auto up = std::make_unique<int>(10);
  auto funC = [up = std::move(up)]()
  {
      std::cout << *up << std::endl;
  };
  funC();
  ```

  
