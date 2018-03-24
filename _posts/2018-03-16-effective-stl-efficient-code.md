---
layout: post
title: "Effective STL: Efficient Code"
permalink: effective-stl-efficient-code
comments: true
---

### Overview

### Function Arguments
In C++, passing an object by _value_ invokes the [copy constructor](http://en.cppreference.com/w/cpp/language/copy_constructor).

To avoid recursively copying objects, pass STL containers by _reference_. Conceptually, a [reference](http://en.cppreference.com/w/cpp/language/reference) is an alias to an existing object (similar to a pointer). Therefore, rather than:

```cpp
// pass by value
int count_value(map<string, string> map, string value) {
  int count = 0;
  // assign by value (create a copy of pair)
  for (auto pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

change the function definition and ranged-based for loop to references:

```cpp
// pass by reference
int count_value(const map<string, string> &map, const string &value) {
  int count = 0;
  // assign by reference (obtain a reference to pair)
  for (const auto &pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

The second version is _significantly_ faster.

**Exception**: if creating a copy is necessary, _then_ pass by the value. For instance, rather than:

```cpp
// pass by const-reference
void f(const vector<T> &x) {
  // create copy
  vector<T> y(x);

  // modify y
}
```

create a direct copy:

```cpp
// pass by value
void f(vector<T> x) {
  // modify x
}
```

### Vector Initialization
The STL provides the _fill constructor_ to initialize the size of a vector (similar to array initialization):

```cpp
vector<T> v(1000);
```

However, the STL _default constructs_ the number of elements specified by the fill constructor. Therefore, `v` contains 1000 elements of type `T`.

If `T` is not a primitive type, the fill constructor is relatively expensive.

To optimize vector initialization, the STL provides the `reserve` method:

```cpp
vector<T> v;
v.reserve(1000);
```

`reserve` avoids explicit memory allocation and simply sets the capacity of the vector to the specified size (which optimizes the [doubling-strategy](https://en.wikipedia.org/wiki/Dynamic_array) of the container).

Therefore, rather than:

```cpp
vector<int> v(1000);

for (int i = 0; i < 1000; i++) {
  v[i] = i;
}
```

`reserve` and `push_back` 1000 elements instead:

```cpp
vector<int> v;
v.reserve(1000);

for (int i = 0; i < 1000; i++) {
  v.push_back(i);
}
```
### Maps
In C++, the standard associative container introduced to a beginner is `std::map`. However, the STL actually provides two map interfaces: `std::map` and `std::unordered_map`.

Internally, `std::map` is a red-black tree (to preserve the _order_ of the keys) and `std::unordered_map` is a simple hash table (with buckets and chaining).

In general, element access is logarithmic time with `std::map` and constant time with `std::unordered_map`.

Therefore, if maintaining keys in sorted order is not a requirement, `std::unordered_map` is significantly faster than `std::map`.

### Map Access
Consider the following code:

```cpp
int decode_value(unordered_map<string, int> &map, const string &key) {
  if (map.find(key) != map.end()) {
    return decode(map[key]);
  }
  return DECODE_ERR;
}

int decode(int value) {
  // decode value
}
```

If the key exists in the map, `decode_value` decodes the associated value. Otherwise, `decode_value` returns an error code.

In Java or Python, `decode_value` is a perfectly reasonable implementation: determine if the key exists and then decode the value. However, in C++, a general optimization exists.

Internally, `find` hashes the key to determine the index of the element and then traverses a linked-list to find the _actual_ key-value pair. Therefore, map access operators (particularly `find` and `operator`) are non-trivial operations (hashing a key is linear in the length of a key and traversing a linked-list involves dereferencing pointers & a higher cache-miss rate).

The optimization is to reduce the number of look-ups by storing the result of `find` in an iterator:

```cpp
int decode_value(unordered_map<string, int> &map, const string &key) {
  auto it = map.find(key);
  if (it != map.end()) {
    return decode(it->second);
  }
  return DECODE_ERR;
}
```

Although a trivial example, this idea is generally applicable to code involving maps (particularly when a key is accessed multiple times).
