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

**Note**: in the code samples, `std::` is omitted for simplicity.

### Declaration
To use a vector in a program, add the header `#include <vector>` to the source file.

Since vectors are template classes, the declaration syntax is
`vector<T>` where `T` is a valid type (primitive or user-defined):

```cpp
// creates a vector of type int
vector<int> v;

// creates a vector of type string
vector<string> v;

// define a simple 2D-point struct
struct point {
  int x;
  int y;
};

// creates a vector of type point
vector<point> v;
```

The previous examples invoke the default constructor, creating an empty vector. To create a vector with an initial length, use the fill constructor:

```cpp
// creates a vector with 100 zeroes
vector<int> v(100);

// define maximum size
const int SIZE = 1000;

// creates a vector with SIZE (1000) zeroes
vector<int> v(SIZE);
```

With the fill constructor, the elements of a vector are default constructed (in the case of an `int`, the default constructed value is zero).

To initialize a vector with a set of elements, use the uniform initialization syntax:

```cpp
// creates a vector containing 4 elements (1, 2, 3)
vector<int> v = {1, 2, 3};

// creates a vector containing 2 elements (1, 2)
vector<int> v{1, 2};

// creates a vector containing 1 element (10)
vector<int> v{10};
```

The STL also provides a range based constructor:

```cpp
// creates a vector containing 5 elements (1, 2, 3, 4, 5)
vector<int> v1 = {1, 2, 3, 4, 5};

// creates a vector containing 2 elements (3, 4)
vector<int> v2(v1.begin() + 2, v1.end() - 1);

// creates a vector containing the values of v2 (3, 4)
// equivalent to:
// vector<int> v3(v2);
vector<int> v3(v2.begin(), v2.end());
```

### Accessing Elements
Accessing elements by index with a vector is identical to the syntax provided by arrays:

```cpp
// create a vector
vector<int> v = {1, 2, 3};

// change first element to -1
v[0] = -1;

// add 10 to the second element
v[1] += 10;

// v is now (-1, 12, 3)
// sum equals 14
int sum = v[0] + v[1] + v[2];

// attempt to access fourth element
int x = v[3]; // ???
```

It is important to note that `operator[]` does _not_ perform bounds checking. Therefore, the last example results in undefined behaviour (the program may crash or continue silently by assigning `x` to a garbage value).

To avoid this, the vector class provides the `at` method which throws an exception if the provided index is _not_ within bounds. In particular, `v.at(3)` will throw an `out_of_range` exception, instead of potentially crashing the program.

### Modifying Elements
The vector class provides several modifier methods, including: `push_back`, `insert`, `erase` and `clear`.

The most common method is `push_back`. As implied by the name, it "pushes" (adds) an element to back of the vector, increasing the size of the container by 1.

Here's an example which demonstrates how to store the contents of text file line-by-line with `push_back` (error checking is omitted for simplicity):

```cpp
// create the file stream
ifstream file("test_file.txt");

// create an empty vector
vector<string> input;

// read file line-by-line, adding the string to the input vector
string line;
while (getline(file, line)) {
  input.push_back(line);
}
```

Thanks to [amortized analysis](https://en.wikipedia.org/wiki/Amortized_analysis), the run-time of  this code is linear (since `push_back` is, on average, constant time).

A common source of error is to confuse `push_back` with `operator[]`. Consider the following example:

```cpp
// example:
// write code to create a vector
// containing the numbers 1 through 10

// construct a vector with initial size of 10
vector<int> v(10);

// add i to the vector
for (int i = 1; i <= 10; i++) {
  v.push_back(i);
}
```

To an inexperienced programmer, this code seems perfectly reasonable. But, it is technically incorrect.

The result is actually a vector with 20 elements -- the first ten are zeroes (created by the constructor) and the remaining elements are 1 through 10 (added by `push_back`).

Another useful method is `insert`:

```cpp
// initialization
vector<char> word = {'h', 'e' 'o'};

// inserts "ll" between "e" and "o"
word.insert(word.begin() + 2, {'l', 'l'});

// prints "hello"
for (int i = 0; i < word.size(); i++) {
  cout << word[i];
}
```

Given an iterator `it` and a value `v`, `insert` shifts all elements to the right of `it` and then copy/move constructs `v`. In the previous example, `v` is an initializer list (rather than a single value).

On average, `insert` is linear time. Therefore, multiple calls to `insert` can result in inefficient code (especially in a loop). Use it sparingly!

**Note:** if appending to the front/middle of a vector is a common operation, consider using the list container instead.

### Algorithms
This section of the tutorial is essentially an introduction to STL algorithm library.

To use the majority of the functions presented in the following examples, add the header `#include <algorithm>` to the source file.

Let's start with a simple example.

#### std::sort
To sort a vector, simply use the `sort` function (elements are arranged in increasing order by default):

```cpp
// create a vector of random numbers
vector<int> v = {100, 5, -1, 3, 99, 7, 5};

// v is now [-1, 3, 5, 5, 7, 99, 100]
sort(v.begin(), v.end());
```

The time-complexity of `sort` is `O(nlogn)`, where `n` is the "distance" between the two iterators passed to `sort`. In this case, `n` is the length of the vector. (For those interested, [introsort](https://en.wikipedia.org/wiki/Introsort) is the sorting algorithm used to implement `sort`).

Since `sort` is a generic algorithm, it is possible to sort vectors containing user-defined types as well:

```cpp
// create an entry struct
struct entry {
  string key;
  int value;
};

// add entries to v
vector<entry> v;
v.push_back({"hello", 1});
v.push_back({"world", 0});
v.push_back({"apples", 10});

// sort vector by passing a comparison function
// v is now [{"world", 0}, {"hello", 1}, {"apples", 10}]
sort(v.begin(), v.end(),
     // compare entries by the value field
     [](const entry &e1, const entry &e2) {
       return (e1.value < e2.value);
     }
);
```

#### std::for_each
Instead of writing a for-loop to iterate through a vector, the STL provides the `for_each` function:

```cpp
// sample implementation of for_each
template<class Iterator, class UnaryOperator>
UnaryOperator for_each(Iterator first, Iterator last, UnaryOperator f) {
    for (; first != last; ++first) {
        f(*first);
    }
    return f;
}
```

Here's a concrete example:

```cpp
vector<string> input = {"hello", "world", "pineapples", "mangoes"};

// print vector line-by-line
for_each(input.begin(),
         input.end(),
         [](const string &s) {
           cout << s << "\n";
         }
);
```

Another common use-case is to mutate the elements of a container:
```cpp
vector<int> v = {1, 2, 3, 4, 5};

// multiply elements by 10
// v is now [10, 20, 30, 40, 50]
for_each(v.begin(), v.end(), [](int &n) { n *= 10; });
```

#### std::accumulate
The final STL algorithm discussed in this tutorial is `accumulate`. Similar to `for_each`, it is essentially for-loop abstraction designed to "accumulate" (or add) the values of a container:

```cpp
// sample implementation of accumulate
template<class Iterator, class T>
T accumulate(Iterator first, Iterator last, T init) {
    for (; first != last; ++first) {
        init = init + *first;
    }
    return init;
}
```
**Note**: `accumulate` is defined in the `#include <numeric>` header.

The canonical example is to calculate the sum of all elements in a vector:

```cpp
// seed RNG with time
srand(time(NULL));

// generate 100 random numbers between 0 and 100
vector<int> v;
for (int i = 0; i < 100; i++) {
  v.push_back(rand() % 100);
}

// calculate sum
int sum = accumulate(v.begin(), v.end(), 0);
```

For user-defined types, `accumulate` is equally powerful. The following example calculates the total salary of employees:

```cpp
// employee struct
struct employee {
  string name;
  int salary;
};

// accumulate function
int employee_sum(int lhs, const employee &rhs) {
  return lhs + rhs.salary;
}

// create employees
vector<employee> v;
v.push_back({"bob", 10});
v.push_back({"alice", 100});
v.push_back({"shak", 10});

// calculate salary
// sum equals 120
int sum = accumulate(.begin(), v.end(), 0, employee_sum)
```

### Conclusion
...and that concludes `Effective STL: Vectors`. Thanks for reading!

Additional Resources: [template classes](http://www.cplusplus.com/doc/oldtutorial/templates/), [iterators](http://www.cplusplus.com/reference/iterator/), and [std::vector](http://www.cplusplus.com/reference/vector/vector/).
