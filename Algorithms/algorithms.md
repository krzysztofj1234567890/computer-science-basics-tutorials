# Data Structures and Algorithms in Java by Michael T. Goodrich, Roberto Tamassia, Michael H. Goldwasser 2014

## Java language

### Exceptions
All subtypes of __RuntimeException__ in Java are officially treated as __unchecked exceptions__, and any exception type that is not part
of the RuntimeException is a __checked exception__.
All __checked exceptions__ that might propagate upward from a method must be explicitly declared in its signature.

### Generics - TODO

### Functional Programming - TODO

## Data Structures

### Arrays

The array is allocated with the specified maximum capacity, but all entries are initially null.

Arrays are great for storing things in a certain order, but they have drawbacks. The capacity of the array must be fixed when it is created, and
insertions and deletions at interior positions of an array can be time consuming if many elements must be shifted.

```
int data[ ] = new int[10];
int[ ][ ] data = new int[8][10];
```

Java provides a class, java.util.Arrays, with a number of built-in static methods for performing common tasks on arrays.

```
java.util.Arrays:

sort(A): Sorts the array A based on a natural ordering of its elements, which must be comparable.
binarySearch(A, x): Searches the sorted array A for value x, returning the index where it is found, or else the index of where it could be inserted while maintaining the sorted order.
```

### LinkedList

A linked list, in its simplest form, is a collection of nodes that collectively form a linear sequence. In a singly linked
list, each node stores a reference to an object that is an element of the sequence, as well as a reference to the next node of the list 

Minimally, the linked list instance must keep a reference to the first node of the list, known as the head.
We can identify the tail as the node having null as its next reference.

An important property of a linked list is that it does not have a predetermined fixed size.
When using a singly linked list, we can easily insert an element at the head of the list,

## Algorithms

### sorting algorithm: insertion-sort

The algorithm proceeds by considering one element at a time, placing the element in the correct order relative to those before it. We
start with the first element in the array, which is trivially sorted by itself. When considering the next element in the array, if it is smaller than the first, we swap
them. Next we consider the third element in the array, swapping it leftward until it is in its proper order relative to the first two elements.

```
/∗∗ Insertion-sort of an array of characters into nondecreasing order ∗/
public static void insertionSort(char[ ] data) {
    int n = data.length;
    for (int k = 1; k < n; k++) { // begin with second character
        char cur = data[k]; // time to insert cur=data[k]
        int j = k; // find correct index j for cur
        while (j > 0 && data[j−1] > cur) { // thus, data[j-1] must go after cur
            data[j] = data[j−1]; // slide data[j-1] rightward
            j−−; // and consider previous j for cur
        }
        data[j] = cur; // this is the proper place for cur
    }
}
```

## Design Patterns






# Other Algorithms

## traversal

### Description

Traversing a binary tree is a fundamental operation in data structures and algorithms. It involves systematically visiting each node in the tree, and there are various approaches to accomplish this, including recursive and iterative methods. 



## sliding-window

## divide and conquer

## breadth-first search vs. depth-first

# References
https://builtin.com/data-science/sliding-window-algorithm

https://protegejj.gitbook.io/algorithm-practice/leetcode/graph/133-clone-graph

