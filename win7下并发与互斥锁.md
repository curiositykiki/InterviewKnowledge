### 互斥锁+多线程

其实一开始是因为想要实现一个单例模式，如果是懒汉型其实**不需要线程安全**，但是如果是饿汉型就需要线程安全了；

实现方法一：

头文件

```c++
#pragma once
#include<string>
#include<iostream>
using namespace std;
class singleton {
private:
	singleton() {
		cout << "singleton setup." << endl;
		return;
	}
	singleton(const singleton& sg) {}
	singleton& operator=(const singleton &sg) {}
	~singleton() {
		cout << "singleton destory." << endl;
		return;
	}
	static singleton* instance;
	class CGar {
	public:
		CGar() {}
		~CGar() {
			if (instance != NULL) {
				delete instance;
				instance = nullptr;
			}
			cout << "instance clear." << endl;
		}
		static CGar backend;
	};
public:
	static singleton* getInstance() {
		if (instance == NULL)
			instance = new singleton();
		return instance;
	}
};
```

操作文件：

```c++
#include<bits/stdc++.h>
#include<map>
#include<unordered_map>
#include"Bank.h"
using namespace std;
singleton* singleton::instance = nullptr;
singleton::CGar singleton::CGar::backend;
int main() {
	singleton *p = singleton::getInstance();
	singleton *q = singleton::getInstance();
	cout << p << "--" << q << endl;
	return 0;
}
```

这样就实现了，但是这是**线程不安全的**；在多并发的情况下，有可能有多个线程同时突破instance==NULL的检测，从而有多个实例；

**因此使用互斥锁来保证只有一个实例会被创建**，同时，由于**加锁是有开销的**，我们双重判断，如果instance==NULL才去加锁，当然，加锁完了还需要判断一次；

头文件

```c++
#pragma once
#include<string>
#include<iostream>
#include<mutex>
using namespace std;
class singleton {
private:
	singleton() {
		cout << "singleton setup." << endl;
		return;
	}
	singleton(const singleton& sg) {}
	singleton& operator=(const singleton &sg) {}
	~singleton() {
		cout << "singleton destory." << endl;
		return;
	}
	static singleton* instance;
	class CGar {
	public:
		CGar() {}
		~CGar() {
			if (instance != NULL) {
				delete instance;
				instance = nullptr;
			}
			cout << "instance clear." << endl;
		}
		static CGar backend;
	};
public:
	static mutex mt;
	static singleton* getInstance() {
		if (instance == NULL) {
			mt.lock();
			if (instance == NULL)
				instance = new singleton();
			mt.unlock();
		}
		return instance;
	}
};
```

操作文件

```c++
#include<bits/stdc++.h>
#include<map>
#include<unordered_map>
#include<thread>
#include"Bank.h"
using namespace std;
mutex singleton::mt;
singleton* singleton::instance = nullptr;
singleton::CGar singleton::CGar::backend;
void justPrint() {
	singleton *p = singleton::getInstance();
	cout << p << endl;
	return;
}
int main() {
	thread th1(justPrint);
	thread th2(justPrint);
	thread th3(justPrint);
	thread th4(justPrint);
	th1.join();
	th2.join();
	th3.join();
	th4.join();
	return 0;
}
```

同时，这里要注意下：**如何在windows里面使用多线程**；首先需要头文件<thread>，然后直接使用

thread xx（func）

就可以开启一个名为xx的线程开始执行func函数，最后**不要忘记使用join**来收回线程的资源；

**如果忘记解锁怎么办？**

最好的办法就是使用lock_guard，其原理跟智能指针一样，这里我们可以用类似的方式使用：

```c++
#include<bits/stdc++.h>
#include<map>
#include<unordered_map>
#include<thread>
#include<mutex>
#include<Windows.h>
#include"Bank.h"
using namespace std;
mutex mt;
int Count = 0;
void printSth(int x) {
	lock_guard<mutex> lck(mt);//在这里使用lock_guard加锁，然后不需要解锁
	Count += x;
	Sleep(500);
	cout << Count << endl;
	return;
}
int main() {
	thread th1(printSth, 3);
	thread th2(printSth, 2);
	thread th3(printSth, 1);
	th1.join();
	th2.join();
	th3.join();
	return 0;
}
```

注意看，这样操作会很方便快捷；

### 只执行一次的函数

好像在<thread>头文件里面就有保证只执行一次的函数；

组合使用once_flag和call_once；

头文件

```c++
#pragma once
#include<string>
#include<iostream>
#include<mutex>
#include<thread>
using namespace std;
class singleton {
private:
	singleton() {
		cout << "singleton setup." << endl;
		return;
	}
	singleton(const singleton& sg) {}
	singleton& operator=(const singleton &sg) {}
	~singleton() {
		cout << "singleton destory." << endl;
		return;
	}
	static singleton* instance;
	class CGar {
	public:
		CGar() {}
		~CGar() {
			if (instance != NULL) {
				delete instance;
				instance = nullptr;
			}
			cout << "instance clear." << endl;
		}
		static CGar backend;
	};
public:
	static once_flag flag;
	static void createSingleton() {
		instance = new singleton();
		return;
	}
	static singleton* getInstance() {
		call_once(flag, createSingleton);
		return instance;
	}
};
```

操作文件

```c++
#include<bits/stdc++.h>
#include<map>
#include<unordered_map>
#include"Bank.h"
using namespace std;
/*
input:


output:


*/
singleton* singleton::instance = nullptr;
singleton::CGar singleton::CGar::backend;
once_flag singleton::flag;
void justPrint() {
	singleton *p = singleton::getInstance();
	cout << p << endl;
	return;
}
int main() {
	thread th1(justPrint);
	thread th2(justPrint);
	thread th3(justPrint);
	thread th4(justPrint);
	th1.join();
	th2.join();
	th3.join();
	th4.join();
	return 0;
}
```

这样可以保证instance必然是一个单例模式；

### 带参数的并发

注意到其实这个thread是一个类，也就是说，可以生成匿名的thread；

```c++
// lock_guard example
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard
#include <stdexcept>      // std::logic_error

std::mutex mtx;

void print_even (int x) {
  if (x%2==0) std::cout << x << " is even\n";
  else throw (std::logic_error("not even"));
}

void print_thread_id (int id) {
  try {
    // using a local lock_guard to lock mtx guarantees unlocking on destruction / exception:
    std::lock_guard<std::mutex> lck (mtx);
    print_even(id);
  }
  catch (std::logic_error&) {
    std::cout << "[exception caught]\n";
  }
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(print_thread_id,i+1);

  for (auto& th : threads) th.join();

  return 0;
}
```

注意到里面的threads里面的每一个函数都**生成一个thread实例**，然后调用print_thread_id函数，**传入的参数为i+1**，这里就学会了如何传参；