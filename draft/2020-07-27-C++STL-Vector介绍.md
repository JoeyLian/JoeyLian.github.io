---
title: C++STL-Vector与vector<bool>介绍
date: 2020-07-27
categories:
- 编程语言
tags:
- C++
- STL标准库
- Vector
typora-copy-images-to: ..\images
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>
本文大部分参考于：[CPlusPlus - Vector](http://www.cplusplus.com/reference/vector/)

## Vector

### 简介

Vector 是表示表示动态改变大小的数组的顺序容器 (Sequence Container)。

和数组一样，Vector 使用连续的空间存储元素，因此可以在常数时间内访问指定的元素。与数组不同的是，Vector 的大小可以动态改变，有容器自动处理。

Vector 在内部使用动态分配的数组存储其元素。插入新元素时，可能需要重新分配数组以增大数组的大小，这意味着分配新数组并将所有元素移至该数组。这需要**线性时间**，因此，显然不能每次添加元素都重新分配数组。

Vector 采用的是每次分配都分配额外的空间，实际容量 (`capacity()`) 大于其包含元素的需要的容量(`size()`)。为了使插入元素的均摊时间复杂度是常数时间，重新分配容量大小应以**指数增长** (即空容器初始化后，容量以1, 2, 4, 8, 16, ... 增长)。

与数组相比，Vector 通过消耗更多的内存与一定的重新分配时间换取动态改变大小的能力。

与链表相比，Vector 在访问指定元素的时候更为高效，并且可以高效地在其末尾添加和删除元素。但是涉及末尾之外地插入和删除元素的操作，它的性能较差。

### 成员函数

##### 构造函数

```c++
#include <iostream>
#include <vector>
int main ()
{
  //1. 构造整数类型的空 Vector
  std::vector<int> first;
    
  //2. 构造一个size为 100 的Vector，每个元素被初始化为 0
  std::vector<int> second (100);     
    
  //3. 构造一个含有4个值100的元素的Vector
  std::vector<int> second (4,100);
    
  //4. 通过迭代器限制范围，按顺序将范围内的元素重新在 Vector 中构造
  std::vector<int> third (second.begin(),second.end()); 
    
  //5. 拷贝构造函数
  //6. 当参数是一个右值引用时，会使用移动构造函数。详见：上一篇文章
  std::vector<int> fourth (third);   
    
  //7. 也可以使用数组或其他列表构造 Vector
  int myints[] = {16,2,77,29};
  std::vector<int> fifth (myints, myints + sizeof(myints) / sizeof(int) );

  return 0;
}
```

**时间复杂度**：

+ 构造函数 1. 6. 使用常数时间。
+ 其他构造函数时间与元素个数成正比。其中 4. 参数若不是一个正向迭代器，无法知道元素个数，可能会出现重新分配空间带来的时间消耗。

**注意**：构造函数2. 虽然没有指定元素初始化的值，但元素已经被创建并初始化。此时，Vector的capacity与size相等。使用 `push_back()` 会增大 Vector 的容量并往之后添加元素。需要修改前面的元素应该直接用 `[]` 或 `at()`。

##### 赋值函数

```c++
#include <iostream>
#include <vector>
int main ()
{
  std::vector<int> foo (3,0);
  std::vector<int> bar (5,0);
   
  //1. 拷贝赋值函数
  //2. 当参数是一个右值引用时，会使用移动赋值函数。
  bar = foo;
    
  //3. 也可以使用数组或其他列表构造 Vector
  foo = {16,2,77,29};
  foo = std::vector<int>();
  foo = std::vector<int>(bar.begin(),bar.end());
  //......
  return 0;
}
```

#### 迭代器相关函数

迭代器使得我们可以用一致的方法遍历不同的容器类型。经典的使用迭代器的方法为：

```c++
for (auto it = myvector.cbegin(); it != myvector.cend(); ++it)
    std::cout << ' ' << *it;
```

`begin()`, `end()` 返回正向迭代器的第一个和最后一个元素。`rbegin()`, `rend()` 返回反向迭代器的第一个和最后一个元素。即：`rbegin()` 返回的迭代器指向的元素与 `end()` 相同，但通过 `it++` 操作会到达前一个迭代器。

C++11之后，在迭代器前加 `c ` (如 `cbegin()`) 可以返回 `const_iterator`，此时不允许通过返回的迭代器修改容器内的内容，但可以通过递增和递减移动迭代器。使得操作更加安全。

#### 容量相关函数

##### 查询函数

+ `size()` 返回当前容器内元素的数量
+ `capacity()` 返回当前为 Vector分配的存储空间的大小，以可以存储的元素数量表示。
+ `max_size()` 返回 Vector 可以容纳的最大元素数，由系统或库实现限制。
+ `empty()` 返回 Vector 是否为空

```c++
#include <iostream>
#include <vector>

int main ()
{
  std::vector<int> myvector;
  for (int i=0; i<100; i++) myvector.push_back(i);

  std::cout << "size: " << myvector.size() << "\n";			//100
  std::cout << "capacity: " << myvector.capacity() << "\n";	//128
  std::cout << "max_size: " << myvector.max_size() << "\n";	//1073741823
  return 0;
}
```

##### 操作函数

+ `resize()` 改变 Vector 的 size。

  + 如果n小于当前 Vector 的 size，则将内容减少到其前n个元素，并删除超出范围的元素，销毁它们。
  + 如果n大于当前 Vector 的 size，则通过在末尾插入所需数量的元素来扩展内容，以达到n的大小。如果指定了 val，则将新元素初始化为 val 的副本，否则，将对它们进行值初始化。
  + 如果n也大于当前 Vector 的 capacity，将自动重新分配存储空间。

  消耗时间与插入/删除的元素数量成正比

+ `reserve()` 改变 Vector 的capacity，使得 capacity 至少足以包含n个元素。

  + 如果 n 大于当前capacity，则该函数使容器重新分配其存储，将其 capacity 增加到 n（或更大）。
  + 在所有其他情况下，函数调用不会导致重新分配，capacity 也不会受到影响。

  这个函数对 Vector 的 size 没有影响，并且不能更改其元素。如果发生重新分配，会造成线性时间。

+ `shrink_to_fit()` 要求Vector 减小其 capacity 以适合其 size 。

  这可能会导致重新分配，但对 Vector 的 size 没有影响，并且无法更改其元素。

+ + 

#### 元素访问函数

+ `[]` 与 `at()`

  返回对vector中位置n处元素的引用。 `[]` 不检查边界。而 `at()`函数自动检查 n 是否在Vector的有效元素范围内，如果不存在，则抛出 out_of_range 异常。

+ `front()` 与 `back()`

  返回 vector 中第一个与最后一个元素的引用。与迭代器不同。

+ [`data()`](http://www.cplusplus.com/reference/vector/vector/data/)

#### 修改函数

##### 末尾修改

+ `push_back()` 调用拷贝构造函数将参数复制到Vector最后一个元素之后。当参数为右值引用时，先构建一个临时变量，调用移动构造函数将临时变量移动到最后一个元素之后。均摊时间为常数。
+ `emplace_back()` 在Vector末尾插入新元素，使用 args 作为其构造函数的参数就地构造新元素。均摊时间为常数。
+ `pop_back()` 删除最后一个元素。常数时间。

##### 其他修改

+ [`assign()`](http://www.cplusplus.com/reference/vector/vector/assign/) 

+ `swap()`  `a.swap(b)` 交换两个 Vector。常数时间。

+ `clear()` 线性时间，清除所有元素。不保证重新分配。强制重新分配的典型替代方法是使用`swap()`: `vector<T>().swap(x);   // clear x reallocating `

+ `insert()`, `erase()`, `emplace()` 通过迭代器定位，在指定位置插入、删除元素。由于需要移动插入、删除元素后的所有元素，需要线性时间。

  ```c++
  it = myvector.begin();
  it = myvector.insert ( it , 200 );
  myvector.insert (it,2,300);
  // "it" no longer valid, get a new one:
  ```



## vector\<bool\>

### 介绍

vector\<bool\> 是vector对于bool类型的特殊版本，进行了空间的优化，将每个值存储在单个位中。元素不是使用分配器构造的，而是将它们的值直接设置在内部存储器的适当位上。

### 成员函数

vector\<bool\> 大部分成员函数与 vector 相同，但是没有 `data()`, `emplace()`, `emplace_back()` 函数。此外，增加了 `flip()` 函数。

`a.flip()` 将所有元素翻转：所有 true 元素变为 false，所有 false 元素变为 true。 



参考资料：

> + [CPlusPlus - Vector](http://www.cplusplus.com/reference/vector/)
> + [C++ vector 容器浅析](https://www.runoob.com/w3cnote/cpp-vector-container-analysis.html)



最后编辑：2020-08-06