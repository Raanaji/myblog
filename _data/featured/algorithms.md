---
template: BlogPost
path: /algorithms
mockup: 
thumbnail:
date: 2019-10-09
name: Introduction to basic algorithms
title: A simple introduction to algorithms in quantitative finance
category: Quantitative Finance
description: 'We take a practical look at algos in trading.'
Tags: s
  - Algos 
  - Data Structures
  - Trading
  - Pseudo Code
---

This is a brief post on what algorithms actually are from a financial engineering perspective. We are no computer science majors that we will waste our time understanding floats and pointers! Most books and university curriculums on technical financial engineering topics, deserve to be sold on toilet tissue shelf of super markets. I try to be as brief as possible when it comes to complex topics, if you do not understand any specific word, just double click on it, right click, select search with google.

If you are under the impression that algo trading involves softwares and platforms to do automated trading (based on technical analysis principles), then you are mistaken. Algo trading simply means that we use computational algorithms and base our decisions on the output after applying those algos. The final execution can be automated (wherein the minute details such as the bid-ask, sell price / buy price, quantity, etc). However, the decision to perform these task on which specific asset, its volume, price (buy/sell), the process used to come to the conclusion using purely computational analysis of data is what is called algotrading.

## Introduction

If data structures are the parts of a car, then algorithms form the engine that make the car go. An algorithm is a set of instructions to perform one or more actions. The most typical algorithms used in computer science are sorting and searching algorithms. Sorting provides an order to a list of elements that can each be compared. A searching algorithm looks to find a particular element within a list of elements. Sorting and searching go hand in hand given that a sorted list is of course easier to search. Sorting a list or searching for an element in a list as efficiently as possible is a basic and important part of computer science

## AVL Trees

AVL trees are a type of self-balancing binary search tree, named after inventors **A**delson-**V**elskii and **L**andis. They are often compared with red black trees, though unlike the former, they are not perfectly balanced. This [paper](https://www.ijert.org/discovering-approach-of-classification-for-stock-market-prediction-using-cart-with-avl-tree) has an interesting analysis of using CART and AVL for stock prediction. This can be easily reproduced in a google collar or a locally hosted Jupyter notebook. [Here](https://github.com/Raanaji/MachineLearning/blob/020a5fadd7e44705b3ed49b3e11e34ecd34e58da/Clustering,%20CART%20and%20SVM.ipynb) is a clustering CART and SVM analysis I have conducted using the same. 

AVL trees have the following properties:

- The sub-trees of every node differ in height by at most one
- Every sub-tree is an AVL tree

This can cause some unbalanced trees though. Each node stores a balance factor which is the difference in height between the left sub-tree and right sub-tree (for an AVL tree, this should be -1, 0, or 1 – if not, then it’s not an AVL tree). When modifying an AVL tree, we re-arrange the part of the tree that is no longer AVL to create an AVL tree using rotations

For insertions:

- We first find the part of the tree where we can insert the element
- We then recursively go up the tree and find a node that is not an AVL sub-tree 
- We use a rotation (left or right) to ensure the sub-tree is an AVL tree

For deletions:

- We delete the node as we would from a normal binary search tree until we reach a leaf node
- We then recursively go back up the tree like with insertions and ensure that each node is an AVL sub-tree

### AVL Trees vs Red Black Trees

AVL trees are generally better performant than red black trees for searching. Red black trees are better for frequent insertions/deletions since rotations on an AVL tree can be expensive. AVL stores a balance factor at each node, whereas a red black tree stores the red/black indicator as a single bit, meaning AVL tree storage is larger

### What is time complexity?

To recap **time complexity** estimates how an algorithm performs regardless of the kind of machine it runs on. You can get the time complexity by “counting” the number of operations performed by your code. This time complexity is defined as a function of the input size `n` using Big-O notation. `n` indicates the input size, while O is the worst-case scenario growth rate function. 

Best case for bubble sort complexity is *O(n)* complexity, worst case is *O(n^2)* complexity, average case is *O(n^2)* complexity. While for selection sort complexity the best case, worst case, and the average case is *O(n^2)* complexity.

We use the Big-O notation to classify algorithms based on their running time or space (memory used) as the input grows. The `O` function is the growth rate in function of the input size `n`.

Here are the **big O cheatsheet**

| Big O Notation | Name         | Example(s)                                                   |
| -------------- | ------------ | ------------------------------------------------------------ |
| *O(1)*         | Constant     | # [Odd or Even number](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Odd-or-Even), # [Look-up table (on average)](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Look-up-table) |
| *O(log n)*     | Logarithmic  | # [Finding element on sorted array with **binary search**](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Binary-search) |
| *O(n)*         | Linear       | # [Find max element in unsorted array](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#The-largest-item-on-an-unsorted-array), # Duplicate elements in array with Hash Map |
| *O(n log n)*   | Linearithmic | # [Sorting elements in array with **merge sort**](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Mergesort) |
| *O(n2)*        | Quadratic    | # [Duplicate elements in array **(naïve)**](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Has-duplicates), # [Sorting array with **bubble sort**](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Bubble-sort) |
| *O(n3)*        | Cubic        | # [3 variables equation solver](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Triple-nested-loops) |
| *O(2n)*        | Exponential  | # [Find all subsets](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Subsets-of-a-Set) |
| *O(n!)*        | Factorial    | # [Find all permutations of a given set/string](https://adrianmejia.com/most-popular-algorithms-time-complexity-every-programmer-should-know-free-online-tutorial-course/#Permutations) |



## Bubble Sort

Bubble sort is an iterative sorting algorithm that takes `O(n^2)` complexity. The sorting takes consecutive elements *l_i* and *l_(I+1)* in the list *l* and compares them, going from *i=0* to *i=n-2* in a *n*-element list

If we are sorting the elements in ascending order, then we swap the two elements if *l_i* > *l**i+1*, Thus each pass over the list takes *n-1* operations. 

We repeat this over the list until we have sorted it completely and thus no longer need to swap any two elements. We will potentially do this *n* times, thus making a maximum of *n(n-1)*, giving complexity of *O(n^2)*

### Example

Let’s take the below list as an example – we have the following 8 elements to start with: [ 34, 46, 25, 5, 56, 16, 68, 36 ]

Here is how we iteratively sort the array:

- [ 34, 25, 5, 46, 16, 56, 36, 68 ] -> Iteration 1 (Comparisons=7, Swaps=4)

- [ 25, 5, 34, 16, 46, 36, 56, 68 ] -> Iteration 2 (Comparisons=7, Swaps=4)

- [ 5, 25, 16, 34, 36, 46, 56, 68 ] -> Iteration 3 (Comparisons=7, Swaps=3)

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 4 (Comparisons=7, Swaps=1)

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 5 (Comparisons=7, Swaps=0)

Here we have taken 5 iterations to sort this list, each with 7 comparisons, making 35 comparisons in total



## Selection Sort

Selection sort is a simple sorting algorithm that also takes *O(n^2)* complexity. Like bubble sort, it is an in-place sorting algorithm where elements are sorted within the original list rather than needing to create a new one. 

Selection sort divides the list into two parts – the sorted part (at the left) and the still-to-be sorted sublist (at the right). The sorted sublist begins as an empty list, and the other part is the entire list at this point

Selection sort searches the sublist at the right and finds the smallest element (if we are sorting into ascending order). It takes this element and places it at the right of the sorted sublist, iteratively doing this until we have an empty sublist at the right and the sorted sublist is the entire list itself

### Example

Let’s take the example above as our input list: [ 34, 46, 25, 5, 56, 16, 68, 36 ]

We can sort this list iteratively as follows – we mark the right sublist in red font:

- [ 5, **34**, **46**, **25**, **56**, **16**, **68**, **36** ] -> Iteration 1, Comparisons=8

- [ 5, 16, **34**, **46**, **25**, **56**, **68**, **36** ] -> Iteration 2, Comparisons=7

- [ 5, 16, 25, **34**, **46**, **56**, **68**, **36** ] -> Iteration 3, Comparisons=6

- [ 5, 16, 25, 34, **46**, **56**, **68**, **36** ] -> Iteration 4, Comparisons=5

- [ 5, 16, 25, 34, 36, **46**, **56**, **68** ] -> Iteration 5, Comparisons=4

- [ 5, 16, 25, 34, 36, 46, **56**, **68** ] -> Iteration 6, Comparisons=3

- [ 5, 16, 25, 34, 36, 46, 56, **68** ] -> Iteration 7, Comparisons=2

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 8, Comparisons=1

Here we have 8 iterations with the number of comparisons decrementing by 1 each time, for a total of 35 comparisons. Happens to match bubble sort in this case!

## Insertion Sort

Insertion sort is a simple sorting algorithm that also takes *O(n^2)* complexity. Like the rest of the algorithms, it is an in-place algorithm. It works in a similar manner to selection sort in that it splits into two parts – the sorted left sublist and the right yet-to-be-sorted sublist.

It takes the leftmost element in the right sublist and inserts it into the sorted left sublist by traversing the left sublist from left to right until it finds the correct position for it. It does this iteratively until the right sublist is empty and the sorted left sublist is the entire list

### Example

Let’s take the example above as our input list: [ 34, 46, 25, 5, 56, 16, 68, 36 ]

We can sort this list iteratively as follows – we mark the right sublist in red font:

- [ 34, **46, 25, 5, 56, 16, 68, 36** ] -> Iteration 1, Comparisons=0

- [ 34, 46, **25, 5, 56, 16, 68, 36** ] -> Iteration 2, Comparisons=1

- [ 25, 34, 46, **5, 56, 16, 68, 36** ] -> Iteration 3, Comparisons=1

- [ 5, 25, 34, 46, **56, 16, 68, 36** ] -> Iteration 4, Comparisons=1

- [ 5, 25, 34, 46, 56, 16, 68, 36 ] -> Iteration 5, Comparisons=4

- [ 5, 16, 25, 34, 46, 56, **68, 36** ] -> Iteration 6, Comparisons=2

- [ 5, 16, 25, 34, 46, 56, 68, **36** ] -> Iteration 7, Comparisons=6

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 8, Comparisons=5

Here we have 8 iterations with a total of 20 comparisons

This is the fastest algorithm so far. Best case is *O(n)* complexity, worst case: *O(n^2)* complexity, average case: *O(n^2)* complexity



## Merge Sort

Merge sort is a more sophisticated sorting algorithm that takes *O(n log n)* complexity. This is our first example of a divide and conquer algorithm. 

First we divide the array into *n* sublists each with 1 element. We then merge each adjacent sublist recursively by copying to a temporary array and going in order through each sublist, choosing the smallest element of each as we go along to move to the temporary array This then forms our next sublist, which we can in turn merge with adjacent sorted sublists

### Example

Let’s take the example above as our input list:

[ 34, 46, 25, 5, 56, 16, 68, 36 ]

We can sort this list iteratively as follows by starting with 1-item sublists:

- [ 34 ] [ 46 ] [ 25 ] [ 5 ] [ 56 ] [ 16 ] [ 68 ] [ 36] -> Iteration 1, Comparisons=0

- [ 34 , 46 ] [ 5 , 25 ] [ 16, 56 ] [ 36 , 68 ] -> Iteration 2, Comparisons=4

- [ 5 , 25, 34, 46 ] [ 16, 36, 56, 68 ] -> Iteration 3, Comparisons=6

- [ 5 , 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 4, Comparisons=6

Here we have 4 iterations for a total of 16 comparisons. Now this is our fastest algorithm so far. Best, worst, and average case is  *O(n log n)* complexity.



## Heap Sort

Heap sort is similar to selection sort in that it divides the list into a sorted and unsorted part. But the use of a heap data structure makes it more efficient than selection sort – it has complexity *O(n log n)*.

Heap sort has the advantage over our other faster sorting algorithms (e.g. quick sort) in that it stays consistently around *O(n log n)* complexity given the use of the heap data structure.

Best, worst, and average case is *O(n log n)* complexity.



## Quick Sort

Quick sort is generally the fastest algorithm out there for searching through lists and is the typical algorithm of choice. Quick sort was developed by Tony Hoare in 1957. It is a divide-and-conquer algorithm much like merge sort and takes *O(n log(n))* complexity

The central principal behind this algorithm is to choose a pivot and then divide the array into two sublists, the left sublist being less than the pivot and the right sublist being greater than the pivot (when we are sorting into ascending order). The pivot is then placed between the two arrays. We choose the first element to be the pivot and then iteratively sort the array. Then we take each left and right subarray and recursively apply the same algorithm

### Example

Let’s take the example above as our input list: [ 34, 46, 25, 5, 56, 16, 68, 36 ]

We can sort this list iteratively as follows – we mark each pivot in red font dividing each subarray:

- [ 25, 5, 16, 34, 46, 56, 68, 36 ] -> Iteration 1, Comparisons=7

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 2, Comparisons=5

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 3, Comparisons=2

- [ 5, 16, 25, 34, 36, 46, 56, 68 ] -> Iteration 4, Comparisons=0

Here we have 4 iterations for a total of 14 comparisons. This is our fastest algorithm. Best case: *O(n log n)* complexity. Worst case: *O(n**2**)* complexity. Average case: *O(n log n)* complexity



## Other Sorting Algorithms

Several other sorting algorithms include:

### Tim Sort

- Combines aspects of merge sort and insertion sort
- Takes advantage of data sets that may be partially ordered already
- Identifies “runs” in the data (i.e. sublists that are already ordered) and merges them together

### Shell Sort

- Similar to bubble sort and insertion sort

- Sorts elements far apart from each other and gradually reduces the gap

Bucket Sort

- Divides the list into sublists (i.e. buckets) and sorts them iteratively



Hopefully through this post I am able to drill down the basics of algorithmic trading, which is all about algorithms and hardly anything about trading. The part which consists of trading is only about the order limits and managing the downsides of certain trades.