---
layout: post
title: "Effective STL: Vectors"
permalink: effective-stl-vectors
comments: true
---

### Overview
A vector is a resizable array, similar to an ArrayList in Java.

The main difference between a vector and an array is flexibility. A vector dynamically resizes to accommodate new elements, whereas an array requires a fixed size. This feature allows vectors to store an arbitrary number of elements.

Another interesting property of vectors is that inserting a new element is
_essentially_ constant time. Therefore, the performance tradeoff between a vector and an array is often insignificant.

### Declaration
To use the vector class, add the header `#include <vector>` to the source file.

Since vectors are template classes, the declaration syntax follows the structure `vector<T>`, where `T` is a primitive or user-defined type. The following examples invoke the _default constructor_, creating an empty vector with a length of zero:

```cpp
struct Point {
  int x, y;
};

// vector of type Point
vector<Point> v1;

// vector of type int
vector<int> v2;

// vector of type string
vector<string> v3;
```

To create a vector with an initial length, there is the _fill constructor_:

```cpp
// 5 ints with value 0
vector<int> v1(5);

// 5 ints with value 100
vector<int> v2(5, 100)
```

In general, the first parameter to the fill constructor is the length of the vector. The second parameter is an _optional_ value. If a value is not specified (as in the first example), the fill constructor will assign the elements to the default constructed value of the declared type. In the case of `int`, the default constructed value is zero.

To initialize a vector to a set of elements, the STL provides the _initializer list constructor_ and the _range-based constructor_:

```cpp
// initializer list
vector<int> v1 = {1, 2, 3, 4, 5};

// initializer list
vector<char> v2 = {'h', 'e', 'l', 'l', 'o'};

// ranged-based: ['h', 'e', 'l']
vector<int> v3(v2.begin(), v2.end() - 2);

// ranged-based: [3, 4]
vector<int> v4(v1.begin() + 2, v1.end() - 1);
```

### Element Access
Accessing a vector by position is identical to the array syntax. In C++, vectors are indexed by zero:

```cpp
vector<string> num = {1, 2, 3, 4, 5};

// reverse vector by swapping the outermost elements
int size = num.size();
for (int i = 0; i < size / 2; i++) {
  swap(num[i], num[size - i - 1]);
}

// attempt to access 6th element...
int test = num[size];
```

Similar to arrays, `operator[]` does _not_ perform bounds checking. Therefore, the last line produces _undefined behaviour_ (the program may crash or continue silently by assigning `test` to a random value).

To avoid undefined behaviour, the vector class provides an `at` method which throws an exception if the specified index is _not_ within bounds. In particular, `v.at(size)` will throw an `out_of_range` exception.

### Mutation
Vectors have several modifier methods, including: `push_back`, `insert`, `erase` and `clear`.

The `push_back` method adds an element to back of the vector, increasing the size of the container by 1.

Here is a simple example:

```cpp
// initialize file stream
ifstream file("test_file.txt");

// create vector to store input
vector<string> input;

// read file line-by-line
string line;
while (getline(file, line)) {
  // add line to vector
  input.push_back(line);
}
```

Thanks to [amortized analysis](https://en.wikipedia.org/wiki/Amortized_analysis), `push_back` is, on average, constant time.

A common source of error is to confuse `push_back` with `operator[]`. Consider initializing a vector to the numbers 1 through 100:

```cpp
// set initial size
vector<int> num(100);

// add i to the vector
for (int i = 1; i <= 100; i++) {
  num.push_back(i);
}
```

To an inexperienced C++ programmer, this code _seems_ reasonable. But, it is incorrect.

In fact, `num` contains 200 elements -- the first hundred elements are zeroes (created by the constructor) and the remaining elements are 1 through 100 (added by `push_back`).

Another common modifier method is `insert`:

```cpp
vector<char> word = {'h', 'l', 'l', 'o'};

// insert 'e' between 'h' and 'l'
word.insert(word.begin() + 1, 'e');
```

In general, `insert` is a linear-time operation. Therefore, avoid calling `insert` in a loop (consider the list container instead).

### Algorithms
To use the STL algorithms, add the header `#include <algorithm>` to the source file.

#### std::sort
By default, the `sort` function arranges elements in non-decreasing order:

```cpp
vector<int> random = {100, 5, -1, 3, 99, 7, 5};

// random -> [-1, 3, 5, 5, 7, 99, 100]
sort(random.begin(), random.end());
```

The time-complexity of `sort` is `O(nlogn)`, where `n` is the "distance" between the two iterators passed to `sort`. In the previous example, `n` is the length of the vector.

Since the STL algorithms are generic, it is possible to sort user-defined objects:

```cpp
class Entry {
  string key;
  int value;
};

vector<entry> entries;
entries.push_back({"hello", 1});
entries.push_back({"world", 0});
entries.push_back({"apples", 10});

// sort entries by value
sort(v.begin(), v.end(),
     [](const Entry &e1, const Entry &e2) {
       return (e1.value < e2.value);
     }
);
```

#### std::for_each
To iterate through a container, the STL provides the `for_each` function:

```cpp
// sample implementation
template<class Iterator, class UnaryOperator>
UnaryOperator for_each(Iterator first, Iterator last, UnaryOperator f) {
    for (; first != last; ++first) {
        f(*first);
    }
    return f;
}
```

In general, `for_each` is equivalent to the range-based for loop introduced in C++11. Here is an example which illustrates the similarity:

```cpp
vector<string> input = {"hello", "world", "pineapples", "mangoes"};

// print input: for_each
for_each(input.begin(), input.end(),
  [](const string &s) {
    cout << s << "\n";
  }
);

// print input: for loop
for (const string &s : input) {
  cout << s << "\n";
}
```

#### std::accumulate
Similar to `for_each`, the `accumulate` function provides an for loop abstraction to add the elements of a container:

```cpp
// sample implementation
template<class Iterator, class T>
T accumulate(Iterator first, Iterator last, T init) {
    for (; first != last; ++first) {
        init = init + *first;
    }
    return init;
}
```
**Note**: `accumulate` is defined in the `#include <numeric>` header.

Here is an example which compares `accumulate` and `for_each`:

```cpp
// seed RNG with time
srand(time(NULL));

// generate 100 random numbers
vector<int> random;
for (int i = 0; i < 100; i++) {
  random.push_back(rand() % 100);
}

// calculate sum: accumulate
int acc = accumulate(random.begin(), random.end(), 0);

// calculate sum: for_each
int sum = 0;
for_each(random.begin(), random.end(),
  [&](const int &n) {
    sum += n;
  }
)
```

### Additional Resources
* [Template Classes](http://www.cplusplus.com/doc/oldtutorial/templates/)
* [Iterators](http://www.cplusplus.com/reference/iterator/)
* [Lambda Expressions](https://msdn.microsoft.com/en-ca/library/dd293608.aspx)
* [std::vector](http://www.cplusplus.com/reference/vector/vector/)

Thanks for reading!
