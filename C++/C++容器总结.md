# 1. **序列容器**

序列容器用于存放有序的元素集合，允许按顺序访问元素。常用的序列容器有：

## a. `std::vector`

- **数据结构**：动态数组，支持随机访问，元素存储在<font color="red">连续的内存中</font>。
- **特点**：高效随机访问，支持动态扩展。

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    vec.push_back(6); // 添加元素
    for (int num : vec) {
        std::cout << num << " "; // 输出: 1 2 3 4 5 6
    }
    return 0;
}
```

### 1. `std::vector` 的内存管理

当你创建一个 `std::vector` 对象时，例如：

```c++
std::vector<int> vec;
```

这个 `vec` 对象本身会存储在栈上。其内部管理的存储空间（即存储 `int` 元素的数组）是分配在堆上的。`std::vector` 会自动管理这个动态分配的内存，包括在需要时为存储新元素而重新分配更大的堆内存。

### 2. 使用 `push_back` 的问题

当使用 `push_back` 方法向 `std::vector` 添加元素时：

- 如果现有的存储空间不足以容纳新的元素，std::vector会进行内存重分配。这包括：
  1. 分配一块新的更大的堆内存（通常是当前容量的两倍）。
  2. 将现有元素复制（或移动）到新的内存中。
  3. 释放旧的内存。
- 这种操作可能导致性能下降（因为需要进行内存重分配和元素复制）。

### 3. 内存问题的潜在情况

虽然在正常情况下使用 `push_back` 不会造成内存错误，但在以下情况下可能会遇到问题：

- **内存泄漏**： 如果你使用 `std::vector` 的指针或引用来在外部持有其元素，并且在 `push_back` 时发生异常（例如内存分配失败），那么可能会导致内存泄漏。
- **异常安全**： 在极少数情况下，如自定义的对象在复制或移动时抛出异常，可能会导致预期外的行为。

### 4. 如何解决或避免问题

若要有效地使用 `std::vector`，且不担心频繁的内存分配带来的性能问题，可以采取几种策略：

1. **预分配空间**： 使用 `reserve` 方法提前分配一定量的内存，以减少后续 `push_back`调用的频繁重分配：

   ```c++
   std::vector<int> vec;
   vec.reserve(100);  // 预留空间以容纳 100 个元素
   ```

2. **使用 `emplace_back`**： 使用 `emplace_back` 方法直接在 `vector` 内部构造对象，可能会更有效率，尤其当存储自定义对象时：

   ```c++
   vec.emplace_back(/* constructor args */);
   ```

3. **定期调整大小**： 在极端情况下，如果知道将要插入的元素数量，可以在使用 `push_back` 之前，调整 `vector` 的大小：

   ```c++
   vec.resize(desired_size);
   ```



**emplace_back:**

- `emplace_back` 函数直接在容器的尾部构造元素，它可以接受任意数量和类型的参数，这些参数正是容器中的元素类型的构造函数所需要的。
- 使用 `emplace_back` 可以避免临时对象的创建和可能的拷贝或移动操作。因为它是直接在容器内存空间中构建对象的，所以它可能会比 `push_back` 更高效。
- `emplace_back` 对于含有非复制或移动构造的对象来说尤其有用，因为它允许在容器中直接构建复杂对象。

**push_back:**

- `push_back` 函数是在容器末尾添加一个已经构造好的对象的副本（或移动副本，如果您使用了[右值引用](https://zhida.zhihu.com/search?content_id=650057997&content_type=Answer&match_order=1&q=右值引用&zhida_source=entity)）。
- 当向容器添加元素时，`push_back` 通常涉及到拷贝或移动构造函数，因为它需要一个完整的对象作为参数。在 C++11 后，如果传入一个临时对象，`push_back` 可以利用移动语义来减少拷贝开销。
- 对于简单数据类型（如 `int`、`float`、指针等），`push_back` 和 `emplace_back` 的效率差别并不明显。

两者之间的选择取决于使用情景：

- 如果您已经有一个对象实例并且想要将其添加到容器中，使用 `push_back` 是合适的。
- 如果您想要构造一个新对象并直接放到容器中，使用 `emplace_back` 可以避免额外的拷贝或移动操作，从而更为高效。





## b. `std::list`

- **数据结构**：双向链表，元素不连续存储，每个元素都通过指针链接到前后的元素，允许快速插入和删除。但随机访问性能差。

```c++
#include <iostream>
#include <list>

int main() {
    std::list<int> lst = {1, 2, 3, 4, 5};

    lst.push_back(6); // 添加元素到尾部
    for (int num : lst) {
        std::cout << num << " "; // 输出: 1 2 3 4 5 6
    }
    return 0;
}
```

## c. `std::deque`

- **数据结构**：双端队列，支持在两端快速插入和删除。
- **内存模型**：分段连续数组（也称作双端队列）
- **是否连续**：否，`std::deque` 的元素可以存储在多个非连续的内存块中。虽然它支持有效的两端插入和删除操作，但不是在单一的连续内存块中。

```c++
#include <iostream>
#include <deque>

int main() {
    std::deque<int> deq = {1, 2, 3, 4, 5};

    deq.push_front(0); // 在前面添加元素
    for (int num : deq) {
        std::cout << num << " "; // 输出: 0 1 2 3 4 5
    }
    return 0;
}
```

## d. `std::array`

- **数据结构**：固定大小的数组封装。
- **是否连续**：是，`std::array` 的元素存储在连续的内存空间中。它的大小在编译时确定，并且不支持动态扩展。

```c++
#include <iostream>
#include <array>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    for (int num : arr) {
        std::cout << num << " "; // 输出: 1 2 3 4 5
    }
    return 0;
}
```



# 2. **关联容器**

关联容器通过键值对存储元素，允许快速查找。

## a. `std::set`

- **数据结构**：基于红黑树实现的集合，所有元素唯一且按顺序存储。
- **内存模型**：基于红黑树（自平衡二叉树）
- **是否连续**：否，`std::set` 的元素存储在非连续的内存块中，树的每个节点都可能在不同的内存位置，因此不保证连续性。

```c++
#include <iostream>
#include <set>

int main() {
    std::set<int> mySet = {5, 3, 1, 4, 2};

    mySet.insert(6); // 添加元素
    for (int num : mySet) {
        std::cout << num << " "; // 输出: 1 2 3 4 5 6
    }
    return 0;
}
```

## b. `std::map`

- **数据结构**：基于红黑树的关联数组，键值对存储，键唯一且按顺序存储。
- **内存模型**：基于红黑树的关联数组
- **是否连续**：否，`std::map` 的元素（键值对）同样存储在非连续的位置，因为它们也形成一个红黑树结构。

```c++
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> myMap;
    myMap["Alice"] = 25;
    myMap["Bob"] = 30;

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl; // 输出: Alice: 25, Bob: 30
    }
    return 0;
}
```

## c. `std::unordered_set`

- **数据结构**：哈希表实现的集合，元素唯一，存储顺序不固定。
- **内存模型**：哈希表，只有key
- **是否连续**：否，`std::unordered_set` 的元素存储在散列表中，表中的每个桶可能对应不同的内存位置。

```c++
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> mySet = {5, 3, 1, 4, 2};

    mySet.insert(6); // 添加元素
    for (int num : mySet) {
        std::cout << num << " "; // 输出顺序可能不同
    }
    return 0;
}
```

## d. `std::unordered_map`

- **数据结构**：哈希表实现的关联数组，键值对存储，键唯一，存储顺序不固定。
- **内存模型**：哈希表。哈希冲突会拉链。
- **是否连续**：否，`std::unordered_map` 的元素（键值对）存储在散列表中，与 `std::unordered_set` 类似。

```c++
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> myMap;
    myMap["Alice"] = 25;
    myMap["Bob"] = 30;

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl; // 输出顺序可能不同
    }
    return 0;
}
```



# 3. **其他容器**

## a. `std::stack`

- **数据结构**：基于容器的堆栈，通常基于 `deque` 或 `vector` 实现。
- **内存模型**：基于底层容器（通常是 `std::deque` 或 `std::vector`）
- **是否连续**：若底层容器是 `std::vector`，则是连续的；若是 `std::deque`，则不是。这取决于使用的底层容器。

```c++
#include <iostream>
#include <stack>

int main() {
    std::stack<int> myStack;
    myStack.push(1);
    myStack.push(2);
    myStack.push(3);

    while (!myStack.empty()) {
        std::cout << myStack.top() << " "; // 输出: 3 2 1
        myStack.pop();
    }
    return 0;
}
```



## b. `std::queue`

- **数据结构**：基于容器的队列，通常基于 `deque` 实现。
- **内存模型**：基于底层容器（通常是 `std::deque`）
- **是否连续**：若底层容器是 `std::deque`，则不是连续的。可以通过指定底层容器来改变这一特性（例如，使用 `std::vector`）。

```c++
#include <iostream>
#include <queue>

int main() {
    std::queue<int> myQueue;
    myQueue.push(1);
    myQueue.push(2);
    myQueue.push(3);

    while (!myQueue.empty()) {
        std::cout << myQueue.front() << " "; // 输出: 1 2 3
        myQueue.pop();
    }
    return 0;
}
```

