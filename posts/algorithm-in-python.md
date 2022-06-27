---
title: "Introduction into Algorithm in Python"
description: Ongoing post about basic algorithm, like sorting and many more, in python.
date: 2019-06-06
tags:
  - python
  - algorithm
  - sorting
layout: layouts/post.njk
image: /img/pexels-daniel-watson-762679.jpg
---

Because I'm currently attending....

![Hero Image: Gearbox, Foto von Pixabay von Pexels](/img/pexels-pixabay-159298.jpg)

## Sorting

### Insertion Sort

Insertion sort is a fairly easy and straight forward sorting algorithm. Imagine you have a deck of cards upside down on a table. You pick the first card and take it into your left hand. Then you pick a second card from the deck and compare its value to the first card already in your left hand. If the newly picked card has a lower value you add it before the first card. If the value is higher you add it after the first card. For all following cards the algorithm goes the same way, comparing the value of the newly picked card to all cards already in your left hand and adding it to a position where on the left side all values are equal or less, and on the right side all values are equal or high, than your newly picked card.

```python
def InsertionSort(A):                   # Cost  Count
    for j in range(1, len(A)):          # c0    n
        key = A[j]                      # c1    n - 1
        i = j - 1                       # c2    n - 1

        while i >= 0 and A[i] > key:    # c3    Sum_1^n (tj)
            A[i + 1] = A[i]             # c4    Sum_1^n (tj - 1)
            i = i - 1                   # c5    Sum_1^n (tj - 1)
        A[i + 1] = key                  # c6    n - 1
```
*Insertion Sort implementation in Python*

The computational performance of the algorithm is then

$$T\left(n\right)=c_0 n + c_1 \left(n-1\right) + c_2 \left(n-1\right) + c_3 \sum_{j=1}^n t_j +  c_4 \sum_{j=1}^n \left(t_j - 1\right) + c_5 \sum_{j=1}^n \left(t_j - 1\right) + c_6 \left(n-1\right)$$




### Merge Sort

### Heap Sort

### Quick Sort

### Radix Sort

### Bucket Sort
