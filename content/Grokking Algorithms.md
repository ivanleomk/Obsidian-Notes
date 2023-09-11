> Grokking Algorithms is a illustrated book by Aditya Bhargava. You can check it out [here](https://www.manning.com/books/grokking-algorithms)


# Algorithms

Algorithms are a set of instructions for accomplishing tasks. When we evaluate algorithms against one another, what we're interested in is the runtime of the algorithm. This is known as Big O Notation, which measures how fast the algorithm runs as the size of the input ( or some other measured quantity ) grows.

**It's important to note that Big O Notation establishes a worst-case run time.**

We need to know the following few runtimes.

| Runtime     | Example                                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------------------------- |
| $O(log(n))$ | Binary Search - input is decreased by half each time                                                        |
| $O(n)$      | Happens when we need to iterate through the entire list ( Eg. linear scan of elements in array )            |
| $O(nlogn)$  | Normally happens when we have sorting algos (Eg. Merge Sort)                                                |
| $O(n^2)$    | When we have a slow algorithm which iterates through the entire list on each iteration (Eg. selection sort) |
| $O(n!)$     | This is exponential runtime, happens when we're exploring a collection of different probabilities (Eg. travelling salesman))                                                                                                            |

## Binary Search

> Input : **Sorted List**
> Output : **Index of Element/Null**

The intuition for binary search is that if we have a sorted list, then we can remove large chunks of the list at a time, in this case, exactly half of the list at each iteration.

For instance, if we have the following list 

```
[1,2,3,4,5,6,7,8,9]
```

Our middle element of the list is going to be `5`

```
>>> x
[1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> x[len(x)//2]
5
```

If we want to find the number `8`, we know that it's definitely not going to lie in the lower part of the list. It's probably going to lie in the second half of the list - between `mid+1` and the last element in our list.

We can implement binary search in python using the code below. We use a while loop to do so but recursive implementations work just fine too.

```python
def binary_search(arr,target)->int:
  left = 0
  right = len(arr)-1
  while left <= right:
    mid = left + (right-left)//2
    if arr[mid] == target:
      return mid
    elif arr[mid] > target:
      right = mid - 1
    else:
      left = mid + 1

  return -1


assert binary_search([1,2,3,4,5,6,7,8],2) == 1
assert binary_search([1,2,3,4,5,6,7,8],8) == 7
assert binary_search([1,2,3,4,5,6,7,8],-10) == -1
assert binary_search([1,2,3,4,5,6,7,8],100) == -1
```

## Selection Sort

> **Input** : Unsorted List
> **Output** : Sorted list of elements

Selection sort is how we would sort a deck of elements. We scan through the list, find the smallest or largest element and then insert it in its proper place.

We can implement it in python using the following code

```python
def selection_sort(arr):
  for insertIdx in range(len(arr)):
    smallest = arr[insertIdx]
    chosenIdx = insertIdx
    for lookup in range(insertIdx,len(arr)):
      if arr[lookup] < smallest:
        chosenIdx = lookup
        smallest = arr[lookup]
    arr[insertIdx],arr[chosenIdx] = arr[chosenIdx],arr[insertIdx]
  return arr

assert selection_sort([3,2,1]) == [1,2,3]
assert selection_sort([3,1,2,5,4]) == [1,2,3,4,5]
assert selection_sort([3,3,2,5,4]) == [2,3,3,4,5]
```

# Memory

## Arrays vs Linked Lists

When we're storing stuff in memory, we can either store it in an entire contiguous chunk or we can store references at each step. 

**Arrays** : We store it all in a single chunk. If we run out of memory, we resize everything

**Linked Lists** : Each element stores a value and a reference to the next value. We never run out of space as long as there is memory

Generally, when we want to sequentially access elements one at a time, we might want to use a linked list. However, when we need to randomly access elements at a specific index, then we might want to use an array.

| Operation | Arrays | Lists |
| --------- | ------ | ----- |
| Reading   | $O(1)$   | $O(n)$  |
| Insertion | $O(n)$   | $O(1)$  |
| Deletion  | $O(n)$   | $O(1)$      |


# Recursion

Recursion involves finding two things

1. A Base Case
2. A way to get to the base case

Often times I find it easier to implement a recursive approach as compared to a imperative approach. 

Internally, our computer uses a call stack to track recursion. This means that a recursive function call results in a set of prior function values and states being stored on the call stack. This is great because **we don't need to keep track of it at all ourselves**. A more advanced method is to use tail recursion, whereby we utilise an accumulator instead of the call stack to keep track of values. 

## Quicksort

Quicksort is an example of a recursive algorithm. It works as follows

1. Choose an element as a pivot
2. Partition elements in the array according to a randomly chosen pivot
3. Recursively apply the same algorithm to the left of the pivot and the right of the pivot

Quicksort is like a game where you pick a number, called the pivot, and then move all the smaller numbers to the left and bigger numbers to the right. You keep doing this with the smaller groups until everything is sorted.

```python
def quicksort(arr):
  def walk(left,right):
    if left >= right:
      return
      
    # For simplicity we always choose the last element as the pivot
    pivot = arr[right]
    lastPivot = left
    for idx in range(left,right):
      if arr[idx] < pivot:
        arr[idx],arr[lastPivot] = arr[lastPivot],arr[idx]
        lastPivot+=1
    arr[lastPivot],arr[right] = arr[right],arr[lastPivot]
    walk(left,lastPivot-1)
    walk(lastPivot+1,right)

  walk(0,len(arr)-1)
  return arr
```

Quicksort is going to be $O(nlogn)$ in our best case scenario and $O(n^2)$ in the worst case scenario. We get our best case scenario if our chosen pivot splits the array into two nicely and our worst case scenario if our chosen pivot is either smaller/larger than every other element in the array at each step.

## Merge Sort

Merge sort's intuition simply goes like this

1. An array with one or no elements is sorted
2. Therefore, we should break down a large array into small arrays with one or no elements AND then merge them back

![[Pasted image 20230911000353.png]]


```python
def mergesort(arr):
  if len(arr) <= 1:
    return arr

  pivot = len(arr)//2
  left = mergesort(arr[:pivot])
  right = mergesort(arr[pivot:])

  currLeft = 0
  currRight = 0
  res = []
  while currLeft < len(left) and currRight < len(right):
    if left[currLeft] < right[currRight]:
      res.append(left[currLeft])
      currLeft+=1
    else:
      res.append(right[currRight])
      currRight+=1
      
  res+=left[currLeft:]
  res+=right[currRight:]
  return res
```


# Hash Functions

Hash functions help us map inputs to a unique output.

