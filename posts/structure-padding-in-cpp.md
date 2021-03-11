---
title: Structure Padding in C++
description: Post about structur padding and space consuption of structures in C++.
date: 2020-09-14
tags:
  - c++
  - memory
layout: layouts/post.njk
image: /img/pexels-pixabay-159298.jpg
---

Lately, I was discussing with other developers at my company about the memory alignment of structure members and I thought why not make a short post about it. To be honest I think it’s an underrated topic nowadays with machines with basically infinite memory and billions of cycles per second. Regardless, even if it’s not important for many developers, it’s still good to know and important if you develop on devices with limited resources.

![Hero Image: Gearbox, Foto von Pixabay von Pexels](/img/pexels-pixabay-159298.jpg)

## Data Models and Resulting Sizes of Fundamental Types

The C++ standard is at least guarantying 16bit width for `short`, `unsigned short`, `int`, and `unsigned int`. 32bit width is guaranteed for `long` and `unsigned long` and 64bit for `long long` and `unsigned long long`. Eventually, the resulting bit size of the fundamental types is defined by the implemented data model. Four data models are widely-used, LP32, ILP32 on 32bit machines, and LLP64, LP64 on 64bit machines.

| Type                           | Standard | LP32 | ILP32 | LLP64 | LP64 |
| ------------------------------ | -------- | ---- | ----- | ----- | ---- |
| `short/unsigned short`         | 16       | 16   | 16    | 16    | 16   |
| `int/unsigned int`             | 16       | 16   | 32    | 32    | 32   |
| `long/unsigned long`           | 32       | 32   | 32    | 32    | 64   |
| `long long/unsigned long long` | 64       | 64   | 64    | 64    | 64   |
*Guaranteed bit size and data model depended bit size, source [cppreference.com][1]*

Let's have a look at the bit size of some fundamental types of a specific implementation on a Ubuntu Linux with clang 7 compiler. 

```bash
Size of char = sizeof(char) = 1 byte -> 8bit
Size of short = sizeof(short) = 2 byte -> 16bit
Size of int = sizeof(int) = 4 byte -> 32bit
Size of long = sizeof(long) = 8 byte -> 64bit
```

## Size of Structures

Let's assume we have the following c++ structure.

```cpp
struct S1 {
  char a;
  int b;
  char c;
  short d;
};
```

Then we would apparently assume the size of the structure in memory will be 8 byte. Let's check this:

```bash
Size of S1 = 1 + 4 + 1 + 2 = 8 byte, real size is 
sizeof(S1) = 12 byte
```

That's interesting. Why is the size of `S1` 12 byte instead of the expected 8 byte? Tho find an answer we have to dig a little bit deeper into how memory is managed by processors. Roughly explained a processor is capable of transferring 1 [word][2] (can be 4 bytes on 32bit and 8 bytes on 64bit machines) to and from memory in one [cycle][3], and an element of fundamental type can be stored at multiples of its byte size in memory (depending on the used compiler). Let's call it 1/2/4 Rule...

- Types with a size of 1 Byte can be stored at multiple of 1 Byte
- Types with a size of 2 Byte can be stored at multiple of 2 Byte
- Types with a size of 4 Byte can be stored at multiple of 4 Byte
- and so on

As a result, the memory alignment of our structure `S1` would probably look like the following in memory with memory address from 0 to f and the resulting size of 12 bytes with 4 bytes of empty memory. This is called padding. The print out of the real memory address (first byte of the allocated memory of an element) is confirming it.

|                   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ----------------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Memory Adress:    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | a | b | c | d | e | f |
| Allocated Memory: | a | - | - | - | b | b | b | b | c | - | d | d |   |   |   |   |
*Memory alignment of structure `S1` which results in a size of 12 byte*

```bash
Address a: 0x7ffe2b97da30
Address b: 0x7ffe2b97da34
Address c: 0x7ffe2b97da38
Address d: 0x7ffe2b97da3a
```

Now it might strike you that it would probably possible to save memory by simply rearranging structure members of `S1` and leveraging how the compiler is aligning elements in memory.

```cpp
struct S2 {
  char a;
  char c;
  short d;
  int b;
};
```

```bash
Size of S2 = 1 + 1 + 2 + 4 = 8 byte, real size is 
sizeof(S2) = 8 byte
```

```bash
Address a: 0x7fff3ae69ab8
Address c: 0x7fff3ae69ab9
Address d: 0x7fff3ae69aba
Address b: 0x7fff3ae69abc
```

|                   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ----------------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Memory Adress:    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | a | b | c | d | e | f |
| Allocated Memory: |   |   |   |   |   |   |   |   | a | c | d | d | b | b | b | b |
*Memory alignment of structure `S2` which results in a size of 8 byte*

This is much better now. But be aware, it's not recommended to just reorder structure elements in ascending order (smallest to the largest element). It is always necessary to keep in mind what we called earlier the 1/2/4 rule. What is the size of `S3`? Does it have the size of 5 bytes?

```cpp
struct S3 {
  char a;
  int b; 
};
```

```bash
Size of S3 = 1 + 4 = 5 byte, real size is 
sizeof(S3) = 8 byte
```

Again, keep in mind the 1/2/4 rule. Because of this, the memory alignment of the elements look now like this:

```bash
Address a: 0x7ffe2e8dc7a8
Address b: 0x7ffe2e8dc7ac
```

|                   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| ----------------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Memory Adress:    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | a | b | c | d | e | f |
| Allocated Memory: |   |   |   |   |   |   |   |   |   |   | a | - | b | b | b | b |
*Memory alignment of structure `S3` which results in a size of 8 byte*

The full source code of this example:

```cpp
#include <iostream>

void printTypeSize() {
  std::cout << "Size of char = " << sizeof(char) << " byte" << std:: endl;
  std::cout << "Size of short = " << sizeof(short) << " byte" << std:: endl;
  std::cout << "Size of int = " << sizeof(int) << " byte" << std:: endl;
  std::cout << "Size of long = " << sizeof(long) << " byte" << std:: endl;
}

struct S1 {
  char a;   // 1 
  int b;    // 4
  char c;   // 1
  short d;  // 2
};

void printS1() {
  std::cout << "Size of S1 = 1 + 4 + 1 + 2 = " 
  << sizeof(char) + sizeof(int) + sizeof(char) + sizeof(short) << " byte, real size is "
  << sizeof(S1) << " byte" << std:: endl;
}

void printS1MemoryLayout() {
  S1 s;
  std::cout << "Address a: " << (void*)&s.a << std::endl;
  std::cout << "Address b: " << (void*)&s.b << std::endl;
  std::cout << "Address c: " << (void*)&s.c << std::endl;
  std::cout << "Address d: " << (void*)&s.d << std::endl;
}

struct S2 {
  char a;   // 1
  char c;   // 1
  short d;  // 2
  int b;    // 4
};

void printS2() {
  std::cout << "Size of S2 = 1 + 1 + 2 + 4 = " 
  << sizeof(char) + sizeof(int) + sizeof(char) + sizeof(short) << " byte, real size is "
  << sizeof(S2) << " byte" << std:: endl;
}

void printS2MemoryLayout() {
  S2 s;
  std::cout << "Address a: " << (void*)&s.a << std::endl;
  std::cout << "Address c: " << (void*)&s.c << std::endl;
  std::cout << "Address d: " << (void*)&s.d << std::endl;
  std::cout << "Address b: " << (void*)&s.b << std::endl;
}

struct S3 {
  char a;
  int b; 
};

void printS3() {
  std::cout << "Size of S3 = 1 + 4 = " 
  << sizeof(char) + sizeof(int) << " byte, real size is "
  << sizeof(S3) << " byte" << std:: endl;
}

void printS3MemoryLayout() {
  S3 s;
  std::cout << "Address a: " << (void*)&s.a << std::endl;
  std::cout << "Address b: " << (void*)&s.b << std::endl;
}

int main() {
  printTypeSize();
  printS1();
  printS1MemoryLayout();
  printS2();
  printS2MemoryLayout();
  printS3();
  printS3MemoryLayout();
}
```

## tl;dr

Nowadays micro-optimizations like the padding/memory alignment of structures might be irrelevant to many fields of software engineering. But still, I think it is an important topic in cases of limited resources and should be always known by every C++ developer in general. It's good to know your memory.

[1]: https://en.cppreference.com/w/cpp/language/types
[2]: https://en.wikipedia.org/wiki/Word_(computer_architecture)
[3]: https://en.wikipedia.org/wiki/Instruction_cycle