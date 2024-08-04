# 1. CMPXCHG（Compare and Exchange）用来实现无锁数据结构
```c++
#include <atomic>
#include <iostream>
#include <thread>

template<typename T>
class LockFreeStack {
private:
    struct Node {
        T data;
        Node* next;
        Node(const T& data) : data(data), next(nullptr) {}
    };

    std::atomic<Node*> head;

public:
    LockFreeStack() : head(nullptr) {}

    void push(const T& data) {
        Node* new_node = new Node(data);
        new_node->next = head.load();
        while (!head.compare_exchange_weak(new_node->next, new_node)) {
            // 如果交换失败，new_node->next 会更新为当前的 head，再次尝试
        }
    }

    bool pop(T& result) {
        Node* old_head = head.load();
        while (old_head && !head.compare_exchange_weak(old_head, old_head->next)) {
            // 如果交换失败，old_head 会更新为当前的 head，再次尝试
        }
        if (old_head == nullptr) {
            return false; // 栈为空
        }
        result = old_head->data;
        delete old_head;
        return true;
    }
};

int main() {
    LockFreeStack<int> stack;
    stack.push(1);
    stack.push(2);
    stack.push(3);

    int value;
    while (stack.pop(value)) {
        std::cout << value << std::endl;
    }

    return 0;
}
```
atomic<T>.compare_exchange_weak, 底层是由lock, cmpxchgl 指令共同实现的。
其中lock前缀确保在多处理器系统中的原子性，cmpxchgl 指令执行比较和交换操作。
``` c++
bool compare_and_exchange(int* ptr, int old_value, int new_value) {
    bool result;
    asm volatile(
        "lock; cmpxchgl %2, %1"
        : "=a" (result), "+m" (*ptr)
        : "r" (new_value), "0" (old_value)
        : "memory"
    );
    return result;
}
```
上面代码利用硬件层面的指令，实现了compare_and_exchange。

下面同样利用底层指令机制，实现一个无锁计数器:
```c++
#include <atomic>
#include <iostream>
#include <thread>

std::atomic<int> counter(0);

void increment() {
    for (int i = 0; i < 1000; ++i) {
        ++counter;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Counter: " << counter << std::endl;
    return 0;
}
```
atomic<T>.fetch_add()底层一种实现:
```c++
#include <atomic>
#include <iostream>

int atomic_add(int* ptr, int increment) {
    asm volatile("lock; xadd %0, %1"
                 : "+r" (increment), "+m" (*ptr)
                 : // No input-only operands
                 : "memory"); // Clobbers memory
    return increment;
}

int main() {
    int counter = 0;
    atomic_add(&counter, 1);
    std::cout << "Counter: " << counter << std::endl;
    return 0;
}
```

# 2. c++ memory model
关键概念
- 数据竞争：如果至少有两个线程访问同一个共享变量，且至少一个访问是写操作，且这些访问没有使用同步机制，那么就会发生数据竞争。
- 顺序一致性：这是一个强内存模型，假定所有线程的所有操作都按照全局顺序执行，且每个线程的操作按程序顺序执行。C++ 提供了顺序一致性的内存序 memory_order_seq_cst。
- 松散的内存顺序：为了性能优化，现代处理器和编译器可能会重新排序指令执行顺序。C++ 提供了不同的内存序来控制这种重排序，包括 memory_order_relaxed、memory_order_acquire、memory_order_release、memory_order_acq_rel 和 memory_order_seq_cst。

内存序
C++ 提供了以下几种内存序，供 std::atomic 操作使用：
- memory_order_relaxed：不保证任何同步，仅保证原子操作本身的原子性。
- memory_order_acquire：确保此操作之前的所有写操作在当前线程可见。
- memory_order_release：确保此操作之后的所有写操作在其他线程可见。
- memory_order_acq_rel：组合了 memory_order_acquire 和 memory_order_release 的特性。
- memory_order_seq_cst：顺序一致性内存序，保证所有原子操作按全局顺序执行。

memory_order_acquire 实现自旋锁：
```c++
#include <atomic>

class Spinlock {
private:
    std::atomic_flag lock_flag = ATOMIC_FLAG_INIT;

public:
    void lock() {
        while (lock_flag.test_and_set(std::memory_order_acquire)) {
            // 自旋等待锁释放
        }
    }

    void unlock() {
        lock_flag.clear(std::memory_order_release);
    }
};
```

memory_order_acquire 和 memory_order_release实现线程级别的producer-consumer:
```c++
#include <atomic>
#include <iostream>

std::atomic<int> data(0);
std::atomic<bool> ready(false);

void producer() {
    data.store(42, std::memory_order_relaxed);
    ready.store(true, std::memory_order_release);  // 发布数据
}

void consumer() {
    while (!ready.load(std::memory_order_acquire)) {  // 获取数据
        // 等待生产者发布数据
    }
    std::cout << "Data: " << data.load(std::memory_order_relaxed) << std::endl;
}
```
