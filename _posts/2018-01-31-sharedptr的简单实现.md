---
layout: chsx
titile: sharedptr的简单实现
tags: sharedptr 内存管理
---

sharedptr的简单实现
====

介绍
------

标准库中的sharedptr简化了内存管理，其内部使用了引用计数，当计数为0的时候就释放内存。

* 当发生拷贝构造的时候，计数器自增1；
* 当发生拷贝赋值的时候，操作符=左侧运算对象的计数器自减1，右侧的运算对象计数器自增1。
* sharedptr析构的时候计数器自减1， 不会直接释放所管理的内存，而是先判断计数器是否为0，只有计数器为0的时候才释放所管理的内存空间。


实现
------

```c++
template<typename T>
class SharedPointer
{
public:
	SharedPointer(T *);
	SharedPointer &operator=(const SharedPointer<T> &);
	SharedPointer(const SharedPointer<T> &);
	~SharedPointer();
	inline T* get() const
	{
		return obj;
	}
private:
	T *obj;
	size_t *counter;
};
template<typename T>
SharedPointer<T>::SharedPointer(T *t): obj(t), counter(new size_t(1))
{
	cout << "call sharedptr's constructer: counter = " << *counter << endl;
}

// 拷贝构造，共享计数器和管理对象指针。
template<typename T>
SharedPointer<T>::SharedPointer(const SharedPointer &ptr)
{
	counter = ptr.counter;
	obj = ptr.obj;
	*counter += 1;
	cout << "call sharedptr's copy constructer: counter = " << *counter << endl;
}
// 拷贝赋值，运算符左侧对象计数器自减1，右侧对象计数器自增1
// 如果左侧对象计数器为0，释放内存 
template<typename T>
SharedPointer<T>& SharedPointer<T>::operator= (const SharedPointer<T> &ptr)
{
	*ptr.counter += 1;
	*counter -= 1;
	if (counter == 0)
	{
		delete counter;
		delete obj;
	}
	obj = ptr.obj;
	counter = ptr.counter;
	cout << "call sharedptr's operator: counter = " << *counter << endl;
	return *this;
}
template<typename T>
SharedPointer<T>::~SharedPointer()
{
    *counter -= 1;
    cout << "call sharedptr's desconstructer: counter = " << *counter << endl;
    if (*counter == 0)
    {
    	delete counter;
    	delete obj;
    }
}
```
