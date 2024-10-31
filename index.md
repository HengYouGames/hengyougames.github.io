### 如何在C++中实现一个线程安全的单例模式

 **1. 【非线程安全的单例模式】** 

```
class Singleton
{
public:
	// 静态方法：提供全局访问点
	static Singleton* getInstance()
	{
		if (instance == nullptr)
		{
			instance = new Singleton();
		}
		return instance;
	}

private:
	// 私有构造函数：防止外部通过new创建实例，确保一个类只有一个实例
	Singleton(){}

	// 静态指针变量：保存单例实例
	static Singleton* instance;
};

// 单例实例初始化
Singleton* Singleton::instance = nullptr;
```

 **2. 【使用互斥锁(Mutex)实现线程安全的单例模式】** 

```
#include <mutex>

class Singleton
{
public:
	static Singleton* getInstance()
	{
		// 加锁保护
		// std::lock_guard<std::mutex>用于自动管理锁的生命周期，确保在作用域结束时自动解锁
		std::lock_guard<std::mutex> lock(mutex_);
		if (instance == nullptr)
		{
			instance = new Singleton();
		}
		return instance;
	}

private:
	Singleton(){}

	static Singleton* instance;
	// 静态互斥锁，用于保护instance的创建过程
	static std::mutex mutex_;
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex_;
```

 **3. 【使用双重检查锁(Double-Checked Locking)实现线程安全的单例模式】** 

```
#include <mutex>
#include <atomic>

class Singleton
{
public:
	static Singleton* getInstance()
	{
		// 加锁前，第一次检查instance是否为空，为了避免不必要的加锁操作，提高了性能
		if (instance == nullptr)
		{
			// 加锁保护
			// std::lock_guard<std::mutex>用于自动管理锁的生命周期，确保在作用域结束时自动解锁
			std::lock_guard<std::mutex> lock(mutex_);
			// 加锁后，第二次检查instance是否为空，确保只有一个线程能够创建实例
			if (instance == nullptr)
			{
				instance = new Singleton();
			}
		}
		return instance;
	}

private:
	Singleton(){}

	static Singleton* instance;
	// 静态互斥锁，用于保护instance的创建过程
	static std::mutex mutex_;
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex_;
```

 **4. 【使用静态局部变量(C++11起)实现线程安全的单例模式】** 

```
class Singleton
{
public:
	static Singleton& getInstance()
	{
		// 线程安全的静态局部变量
		static Singleton instance;
		return instance;
	}

private:
	// 私有构造函数：防止外部通过new创建实例，确保一个类只有一个实例
	Singleton() {}

	// 禁用拷贝构造和赋值操作
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
};
```

 **5. 【使用智能指针和call_once(C++11起)实现线程安全的单例模式】** 

```
#include <mutex>
#include <memory>

class Singleton
{
public:
	static std::shared_ptr<Singleton> getInstance()
	{
		// std::call_once确保初始化函数initInstance只会被调用一次，即使多个线程同时尝试调用
		std::call_once(init_flag, &Singleton::initInstance);
		return instance;
	}

private:
	// 私有构造函数：防止外部通过new创建实例，确保一个类只有一个实例
	Singleton(){}

	// 初始化函数
	static void initInstance()
	{
		instance = std::make_shared<Singleton>();
	}

	// 使用智能指针std::shared_ptr自动管理内存，避免手动new和delete
	static std::shared_ptr<Singleton> instance;
	// std::once_flag是一个标志，用于指示初始化函数initInstance是否已经调用过
	static std::once_flag init_flag;
};

// 初始化
std::shared_ptr<Singleton> Singleton::instance = nullptr;
std::once_flag Singleton::init_flag;
```
