---
layout: post
title: "Effective STL: Efficiency"
permalink: effective-stl-efficiency
comments: true
---

### Overview
// TODO: simple explanation of the post

### Example 1: Argument Passing
In C++, passing an argument by value creates a copy of the object. Consider the following code:

```cpp
// inefficient version: count occurrences of value
int count_value(map<string, string> map, string value) {
  int count = 0;
  for (auto pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

Since `map` is passed by value, C++ creates a copy of the _entire_ container.

If `map` is relatively small, then the copy is often insignificant. But, if `map` is 100 or 1000 elements, then the copy is a _significant_ waste of time. In the range-based for loop, `pair` is initialized by value, introducing an additional copy.

Therefore, when passing (potentially large) objects, pass by reference instead -- the resulting code is significantly faster:

```cpp
// efficient version: count occurrences of value
int count_value(const map<string, string> &map, const string &value) {
  int count = 0;
  for (const auto &pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

**Exception**: if creating a copy of the object is necessary, _then_ pass by the value. For instance, rather than:

```cpp
// incorrect style: pass by const-reference
void vector_copy(const vector<T> &x) {
  // create copy
  vector<T> y(x);

  // modify y
}
```

create a direct copy:

```cpp
// idiomatic style: pass by value
void vector_copy(vector<T> x) {
  // modify x
}
```

### Example 2: Vector Initialization
When initializing a vector, it is common to provide an initial size (similar to array initialization) through the _fill constructor_:

```cpp
// inefficient version: vector with 1000 elements
vector<T> v(1000);
```

The fill constructor, however, _default constructs_ the number of elements provided by the initial size. Therefore, 1000 elements of type `T` are constructed in the previous example. (If `T` is a user-defined type, then the initialization is significantly more expensive).

To avoid the performance introduced by the fill constructor, while maintaining _amortized_ constant time insertion, use the `reserve` method:

```cpp
vector<T> v;
v.reserve(1000);
```

`reserve` sets the internal capacity of the container to _expect_ `n` elements, allowing the doubling strategy to optimize when to grow the size of the vector. This avoids the overhead of default constructing the vectors and in practice, out performs the fill constructor.

### Example 3: Maps
The STL provides two maps: `std::map` and `std::unordered_map`. The `std::map` is implemented as a red-black tree and therefore, preserves the order of the keys. Conversely, `std::unordered_map` is implemented as a hash-table with chaining.

Since `std::map` is effectively a binary-search tree, accessing an element is `O(logn)`. By contrast, `std::unordered_map` is, on average, constant time access. Therefore, if the order in which keys are accessed is not important, `std::unordered_map` is significantly more efficient than `std::map`.

### Example 4: Map Element Access
Maps are perhaps the most useful data structure in computer science. However, most C++ programmers use them inefficiently. Here is a standard (but inefficient way) to interact with a map:
```cpp
// inefficient version: access value if key exists
void random_func(unordered_map<string, int> &map, const string &key) {
  // key exists
  if (map.find(key) != map.end()) {
    do_something(map[key]);
  }
}
```

Compare the above code with this version:

```cpp
// efficient version: access value if key exists
void random_func(unordered_map<string, int> &map, const string &key) {
  auto it = map.find(key);
  // key exists
  if (it != map.end()) {
    do_something(it->second);
  }
}
```

In this example, the difference in performance is extremely subtle, but non-trivial. The first `random_func` hashes the key _twice_: to determine if the key exists and to return the key. The second `random_func` hashes the key _once_: to return an iterator to the map element. Since hashing a key is linear in the length of the key, the first function is significantly less efficient than the second (This example also demonstrates the power of iterators as well.)

### Example 5: Increment Operators
// TODO: discuss difference between post/pre-increment operator
