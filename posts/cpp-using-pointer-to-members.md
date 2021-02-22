---
title: Using Pointer to Members on STL Algorithms
description: Accessing members by a pointer to apply an algorithm to members of complex types
date: 2019-02-22
tags:
  - c++
  - stl
  - pointer
  - algorithm
layout: layouts/post.njk
image: /img/dreamstime_l_103989120-e1550786406758-min.jpg
---

Sometimes I need to execute an operation, like `std::sort` or `std::transform`, on a container of structs or objects using a lambda or function pointers which should take type defined criteria. Another scenario might be a global minima search of an arbitrary brane which could be described by a vector of 3-dimensional vectors. In such cases, we often use [pointer to members][1].

![Hero Image: Application Logic](/img/dreamstime_l_103989120-e1550786406758-min.jpg)

This topic might be a piece of cake for every experienced C++ veteran. But I remember back in the days when I was a novice, I was really irritated by a piece of code which was using pointer to members, to do a special operation. I didn't get what that piece of sh*t was doing and why it was written that wired. And even worse, it was neither easy to find documentation of that code, nor information of such "magic" it was doing on the internet.

So how might something like this look like? Let's say we have a vector of complex numbers which we want to sort by either the real or the imaginary number. This is our complex type:

```cpp
struct Complex {
  double real;
  double imaginary;
};
```

For sorting vectors, we would use std::sort of course and for vectors of primitive types, it's rather simple to use. And indeed, it's easy to use with complex types as well. Because `std::sort` is offering an interface which is taking a compare function object that has to fulfill the requirements of [Compare][2]:

```cpp
template<class RandomIt, class Compare>
constexpr void sort(RandomIt first, RandomIt last, Compare comp); //constexpr since C++ 20
```

Using std::sort for sorting according to real or imaginary numbers might then look like this:

```cpp
std::vector<Complex> vec = { { 3, 1},
                             { 1, 9 },
                             { 8, 2 },
                             { 3, 14 }};

double Complex::*var;
auto comparator = [&var](const Complex &a, const Complex &b) { return a.*var < b.*var; };

var = &Complex::real; //Sorting for real numbers
std::sort(vec.begin(), vec.end(), comparator);

var = &Complex::imaginary; //Sorting for imaginary numbers
std::sort(vec.begin(), vec.end(), comparator);
```

The interesting point in the code above is the declaration of `double Complex::*var` (pointer to member of Complex of type double), in line 6, which is passed to the lambda over the captures clause. The lambda function is using the pointer by dereferencing it to a comparable type (in our case a double) in line 7. It's then used by assigning it a concrete pointer in line 9 and 12 to steer the `std::sort` algorithm.

For me, such indirections are sometimes quite useful but not always. The biggest problem might be the readability for programming beginners. Also, there might be a better way to have the same functionality without pointers to members. I would be glad to hear any suggestions or feedback.

[1]: https://stackoverflow.com/questions/670734/pointer-to-class-data-member
[2]: https://en.cppreference.com/w/cpp/named_req/Compare