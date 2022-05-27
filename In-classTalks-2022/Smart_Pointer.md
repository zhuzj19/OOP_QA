# 智能指针 (Smart Pointer)

## 为什么需要智能指针

* 在程序中，我们会用到指针来指向动态分配在堆中的对象。
* 当程序离开对象作用域时，我们需要回收对象占用的空间，避免内存泄漏(Memory Leak)。
* 不过程序在运行过程中可能会发生各种异常，我们需要在每一种可能的离开作用域情况时都释放内存。
* 智能指针为我们提供了一个自动释放内存的工具。

## C++中的智能指针

* C++中提供的智能指针包括`auto_ptr`(已废弃), `shared_ptr`, `weak_ptr`, `unique_ptr`。
* 这些智能指针包含在`#include <memory>`头文件中。
* 智能指针(除`weak_ptr`外)可以使用像正常指针(裸指针)一样使用`*`, `->`来访问其指向的对象。

## auto_ptr

* `auto_ptr`是一种独占式的指针，每个指针单独拥有对象的所有权。
* 在进行拷贝时，所有权会自动转移，多个指针无法共享一个对象。
* `auto_ptr`不能用于数组对象，由于其释放内存时调用的是`delete`而不是`delete[]`。
* `auto_ptr`已经被废弃，其功能目前可以使用`unique_ptr`代替。

## shared_ptr

* `shared_ptr`支持多个指针共享同一个对象。

* 记录每个对象被引用的个数，当最后一个`shared_ptr`的作用域结束时，内存会自动释放。

* 可以通过`use_count()`查询引用计数。

* `get()`返回当前的指针。

* `reset()`释放当前指针，如果当前指针为该对象的最后一个共享指针，那么内存也会自动释放。

    ```c++
    class Test {
    public:
        Test(int a = 0) : m_a(a) {}
        ~Test() {
            std::cout << "Calling destructor" << std::endl;
        }
        int m_a;
    };
    
    int main() {
        std::shared_ptr<Test> sptr1 = std::make_shared<Test>(100); // Test为手动定义的类
        std::cout << sptr1.use_count() << std::endl; // 1
        std::shared_ptr<Test> sptr2(sptr1);
        std::cout << sptr1.use_count() << std::endl; // 2
        sptr1.reset();
        std::cout << sptr2.use_count() << std::endl; // 1
        return 0;
    }
    ```

* 在函数调用时，既可以拷贝参数，也可以引用。（如果是拷贝了共享指针，那么引用计数就会增加1）

    ```c++
    void func1(std::shared_ptr<Test> sptr) { 
        std::cout << sptr.use_count() << " " << sptr->m_a << std::endl;
    }
    
    void func2(std::shared_ptr<Test>& sptr) {
        std::cout << sptr.use_count() << " " << sptr->m_a << std::endl;
    }
    ```

* `shared_ptr`也可以用于数组对象。可以直接定义针对数组对象的`shared_ptr`(在C++17中支持)，或者为对象手动定义释放内存时调用的析构函数。

    ```c++
    std::shared_ptr<Test[]> sptr1(new Test[5]); // support in C++17
    std::shared_ptr<Test> sptr2(new Test[5], [](Test* p) {delete[] p;});
    ```

## weak_ptr

```c++
class B;

class A
{
public:
    A() : m_sptrB(nullptr) {} ;
    ~A() {
        std::cout << "A is destroyed" << std::endl;
    }
    std::shared_ptr<B> m_sptrB;
};

class B
{
public:
    B() : m_sptrA(nullptr) {};
    ~B() {
        std::cout << "B is destroyed" << std::endl;
    }
    std::shared_ptr<A> m_sptrA;
};

int main() {
	std::shared_ptr<B> sptrB = std::make_shared<B>();
    std::shared_ptr<A> sptrA = std::make_shared<A>();
    sptrB->m_sptrA = sptrA;
    sptrA->m_sptrB = sptrB;
    return 0;
}
```

考虑两个class成员相互指向的情况：在释放指针`sptrA`时，由于`sptrB->m_sptrA`指向了相同的对象，因此`sptrA`指向的对象并不会释放。同样在指针`sptrB`离开作用域时，由于`sptrA`的对象并没有释放，它里面的成员指向`sptrB`的对象，因此`sptrB`指向的对象也不会释放。最终就导致了内存泄漏。

因此在这种情况下，我们需要使用`weak_ptr`。

* `weak_ptr`在语义上可以共享这个对象，但它并不拥有这个对象。(因此`weak_ptr`就不会影响到内存的释放)

* `weak_ptr`并不会导致引用计数增加，因此在刚才那个例子中使用`weak_ptr`就不会引起内存泄漏（将class A, B中的`shared_ptr`换成`weak_ptr`）。

* `weak_ptr`不支持像其它智能指针一样使用`*`, `->`来访问对象。

* 我们可以使用`expired()`来判断`weak_ptr`指向的对象是否已经被释放。

* 再使用`lock()`获取`shared_ptr`后访问对象。(如果对象已经被释放，`lock()`会导致运行错误)

    ```c++
    if (!sptrB->m_wptrA.expired())
        std::cout << sptrB->m_wptrA.lock()->m_a << std::endl;
    ```

## unique_ptr

* `unique_ptr`和`auto_ptr`提供了相似的功能，不过有几点不同。

* `unique_ptr`支持数组对象。

    ```c++
    std::unique_ptr<Test[]> uptr3 = std::make_unique<Test[]>(5);
    ```

* `unique_ptr`不支持拷贝构造。

    ```c++
    std::unique_ptr<Test> uptr1 = std::make_unique<Test>(10);
    // std::unique_ptr<Test> uptr2 = uptr1; // ERROR
    ```

* 函数调用时，不能直接拷贝参数。

    ```c++
    void func1(std::unique_ptr<Test> uptr) {
        std::cout << uptr.get() << " " << uptr->m_a << std::endl;
    }
    
    int main() {
        // func1(uptr1); // ERROR
        func1(std::move(uptr1));
    }
    ```

## 总结

* 智能指针为我们提供了更加方便管理内存的工具。
* 即使使用智能指针，还是有内存泄漏的风险（例如前面`shared_ptr`的例子）。使用智能指针时应熟悉它们各自的功能以及局限性。
* 实际使用时，要根据需要选择合适的智能指针。

## 参考资料

* https://blog.csdn.net/zone_programming/article/details/47000647
* https://www.codeproject.com/Articles/541067/Cplusplus-Smart-Pointers
* https://blog.csdn.net/weixin_45590473/article/details/113057545
* http://c.biancheng.net/view/1478.html
* https://docs.microsoft.com/zh-cn/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170

