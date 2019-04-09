---
layout: post
title: "Effective STL: Efficiency"
permalink: effective-stl-efficiency
comments: true
---

### Overview
Although the title of this post is _efficiency_, the objective is not how to write _efficient_ code, but how to leverage STL features to write _effective_ code (which is, by definition, also efficient).

Prior experience with C++ is recommended.

### Function Arguments
In C++, passing an object by _value_ invokes the [copy constructor](http://en.cppreference.com/w/cpp/language/copy_constructor).

To avoid recursively copying objects, pass STL containers by _reference_. Conceptually, a [reference](http://en.cppreference.com/w/cpp/language/reference) is an alias to an existing object (similar to a pointer).

Therefore, rather than writing:

```cpp
// pass by value
int count_value(unordered_map<string, string> map, string value) {
  int count = 0;
  // create a copy of pair
  for (auto pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

change the function definition to:

```cpp
// pass by reference
int count_value(const unordered_map<string, string> &map, const string &value) {
  int count = 0;
  // obtain a reference to pair
  for (const auto &pair : map) {
    if (pair.second == value) {
      count++;
    }
  }
  return count;
}
```

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
The STL provides the _fill constructor_ to initialize the size of a vector (similar to array declaration):

```cpp
vector<T> v(1000);
```

However, the STL _default constructs_ the number of elements requested by the fill constructor. Therefore, `v` contains 1000 elements of type `T`.

If `T` is not a primitive type, then the fill constructor is relatively expensive.

To optimize vector initialization, the vector class provides the `reserve` method:

```cpp
vector<T> v;
v.reserve(1000);
```

`reserve` avoids explicit memory allocation and instead, sets the capacity of the vector to the specified size (which optimizes the performance of the container).

Therefore, rather than writing:

```cpp
vector<int> v(1000);

for (int i = 0; i < 1000; i++) {
  v[i] = i;
}
```

`reserve` 1000 elements (note `push_back` in the for loop):

```cpp
vector<int> v;
v.reserve(1000);

for (int i = 0; i < 1000; i++) {
  v.push_back(i);
}
```

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

The current implementation of `decode_value` is relatively standard: determine if the key exists and then decode the value. However, there is a subtle inefficiency.

Internally, `find` and `operator[]` hash the key to determine the entry of the hash table and then traverse a linked-list to resolve collisions.

Therefore, map access operators are non-trivial operations (hashing a key is linear in the length of a key and traversing a linked-list involves higher cache miss rates).

The optimization is store the result of `find` in an _iterator_:

```cpp
int decode_value(unordered_map<string, int> &map, const string &key) {
  auto it = map.find(key);
  if (it != map.end()) {
    return decode(it->second);
  }
  return DECODE_ERR;
}
```

Therefore, rather than computing the hash of the key twice, `decode_value` computes the hash of the key, stores the result, and then references the value.

Although `decode_value` is a trivial example, this technique is generally applicable.

### Additional Resources:
* [Effective C++](https://www.aristeia.com/books.html)
* [Efficiency with Algorithms, Performance with Data Structures](https://www.youtube.com/watch?v=fHNmRkzxHWs)
* [High Performance Code 201: Hybrid Data Structures](https://www.youtube.com/watch?v=vElZc6zSIXM)

Thanks for reading!
