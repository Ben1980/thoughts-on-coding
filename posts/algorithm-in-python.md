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
def InsertionSort(A):
    for j in range(1, len(A)):
        key = A[j]
        i = j - 1

        while i >= 0 and A[i] > key:
            A[i + 1] = A[i]
            i = i - 1
        A[i + 1] = key
```

### Merge Sort

### Heap Sort

### Quick Sort

### Radix Sort

### Bucket Sort
