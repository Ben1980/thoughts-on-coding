---
title: "Introduction into Algorithm in Python"
description: Ongoing post about basic algorithm, like sorting and many more, in python.
date: 2022-06-30
tags:
  - python
  - algorithm
  - sorting
layout: layouts/post.njk
image: /img/pexels-daniel-watson-762679.jpg
---

Because I'm currently attending....

![Hero Image: Gearbox, Foto von Pixabay von Pexels](/img/michael-dziedzic-aQYgUYwnCsM-unsplash.jpg)

## Sorting

### Insertion Sort and Introduction Into Computation of Algorithm Efficiency

Insertion sort is a fairly easy and straight forward sorting algorithm. Imagine you have a deck of cards upside down on a table. You pick the first card and take it into your left hand. Then you pick a second card from the deck and compare its value to the first card already in your left hand. If the newly picked card has a lower value you add it before the first card. If the value is higher you add it after the first card. For all following cards the algorithm goes the same way, comparing the value of the newly picked card to all cards already in your left hand and adding it to a position where on the left side all values are equal or less, and on the right side all values are equal or high, than your newly picked card. To derive the algorithm performance efficency we assign each instruction with a *cost* and a *amount* how often the instruction is executed.

```python
def InsertionSort(A):                   # cost  amount
    for j in range(1, len(A)):          # c0    n
        key = A[j]                      # c1    n - 1
        i = j - 1                       # c2    n - 1

        while i >= 0 and A[i] > key:    # c3    Sum_1^n (tj)
            A[i + 1] = A[i]             # c4    Sum_1^n (tj - 1)
            i = i - 1                   # c5    Sum_1^n (tj - 1)
        A[i + 1] = key                  # c6    n - 1
```
*Insertion Sort implementation in Python*

The execution time $T\left(n\right)$ is then the sum of all instruction *cost* times the *amount*:

$$T\left(n\right)=c_0 n + c_1 \left(n-1\right) + c_2 \left(n-1\right) + c_3 \sum_{j=1}^n t_j +  c_4 \sum_{j=1}^n \left(t_j - 1\right) + c_5 \sum_{j=1}^n \left(t_j - 1\right) + c_6 \left(n-1\right)$$

The best case that could happen is if the list is already in an ascending order. Then `i >= 0 and A[i] > key` is always false and our sum is reduced to:

$$T\left(n\right)=c_0 n + c_1 \left(n-1\right) + c_2 \left(n-1\right) + c_6 \left(n-1\right)$$
$$T\left(n\right)=\left(c_0 + c_1 + c_2 + c_6\right)n - \left(c_6 + c_2 + c_1\right)$$

We can say $T\left(n\right)=an+b$. Therefore the runtime is a *linear function*.

In most cases we are interested in the worst case scenarios. For a sorting algorithm thats the case if the list is in an descending order. Lets work through the formula $T\left(n\right)$. First of all the a sum like $\sum_{k=1}^n k$ is a arithmetic series:

$$\sum_{k=1}^n k = \frac{1}{2}n\left(n+1\right)$$

Because For the sum in $T\left(n\right)$ that would be

$$\sum_{k=1}^n t_j = \frac{1}{2}n\left(n+1\right) - 1$$
$$\sum_{k=1}^n \left(t_j - 1\right) = \frac{1}{2}n\left(n+1\right)$$


### Merge Sort

### Heap Sort

### Quick Sort

### Radix Sort

### Bucket Sort
