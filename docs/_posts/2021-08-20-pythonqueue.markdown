---
layout: post
title:  "Implementing Queue In Python"
date:   2021-08-20 18:30:00 -0500
categories: code
---

## Introduction

I've been practicing data-structures and algorithms in preperation for interviews. Every time I've needed to use a queue (maybe for BFS or something...), I just import deque: `from collections import deque`. 

Although using a standard library is the lowest cost and most robust solution, I've been wanting to implement Queue from scratch.

## Stacks Are Easy

When thinking about Queues, my mind often goes to Stacks. In python implementing your own stack is easy with lists. 

With lists, we have `O(1)` appends:
- Lists in Python are implemented as [dynamic arrays](https://en.wikipedia.org/wiki/Dynamic_array) in C, so when we append, Python has a pointer ready for the next memory address to be added in the block where the rest of the array is stored. 
- This makes append *amortized* `O(1)` because if the list exceeds its allocated memory, it needs to copy every element to reallocate the list.

and `O(1)` pops from the end of the list:
- To pop the last element, we only need to remove the last memory address in the array. None of the other elements in the array are affected, so that's it.

**So, since we have `O(1)` appends and pops, we a built in stack.**

### Stack Implementation

Here is our beautiful python stack using lists:

```python
class Stack:

    def __init__(self, xs = []):

        if type(xs) is not list:
            raise TypeError
        else:
            self.stack = xs
    
    def push(self, val):
        self.stack.append(val)
    
    def pop(self):
        return self.stack.pop()
    
    def peek(self):
        return self.stack[-1]
    
    def __str__(self):
        if not self.stack: return 'Empty Stack'
        output = ['\nStack: \n']
        for el in reversed(self.stack):
            output.append(str(el) + '\n')
        return ''.join(output)
    
my_stack = Stack()
my_stack.push(3)
my_stack.push(10)
my_stack.push(1)
my_stack.pop()

print(my_stack)
# Stack: 
# 10
# 3
```
Pretty easy! But what about Queues?

## To Do Queue, First Linked Lists

When I was first studying computer science, it was not clear to me why a linked list would ever be relavant. They seemed almost like a list, but with less features.

As I've studied more, I can see that every data structure has "constrains and gains." With a linked list, while we give up constant look up time for arbitrary elements, we gain the ability to pop the left side of a list in `O(1)`, while maintaining `O(1)` appends, the key to a Queue.

### Linked List Implementation

To get right into it, here's our Linked List:

```python
class LinkedNode:

    def __init__(self, val = None, next = None):
        self.val = val
        self.next = next
    
    def __str__(self):
        return str(self.val)

class LinkedList:

    def __init__(self, head = LinkedNode(), end = None):
        self.head = head
        self.endPointer = head if not end else end
    
    def search(self, target = None):
        
        if not target: return -1

        pointer = self.head
        idx = 0
        if pointer.val is None: return -1

        while pointer is not None:
            if pointer.val == target: return idx
            pointer = pointer.next
            idx += 1
        return -1
    
    def add(self, node):

        if not isinstance(node, LinkedNode): raise TypeError
        if self.head.val is None:
            self.head = node
            self.endPointer = node
        else:
            self.endPointer.next = node
            self.endPointer = node
        return
    

    def __str__(self):

        pointer = self.head
        buildStr = ''

        if pointer.val == None: return 'Empty'

        while pointer is not None:
            buildStr += str(pointer) + ' -> '
            pointer = pointer.next
        
        buildStr += str(pointer)
        return buildStr
    
def linkedFromArr(arr):

    if not arr: raise ValueError

    if len(arr) == 1: return LinkedList(head=LinkedNode(arr[0]))

    headNode = LinkedNode(val=arr[0])
    pointer = headNode

    for v in arr[1:]:
        pointer.next = LinkedNode(val=v)
        pointer = pointer.next
    
    return LinkedList(head=headNode, end=pointer)
```

Without getting too much into the details, the important thing is that we keep two pointers: one for the head of the list and one for the end and every node has a pointer to the next node. 

**To make a queue, we will be able to pop from the left side of the Linked List, by setting the head to the node that the head is pointing to in constant time.**

## Queue Finally

```python
from linkedList import LinkedList, LinkedNode, linkedFromArr

"""
Implement Queue with Linked List. Because we keep pointers for the head
and the end of the linked list, we can have O(1) pops and appends
"""
class Queue:

    def __init__(self, initVal = None, initQ = None):

        # for initializing with a list
        if initQ is not None:
            # check for populated list
            if initQ:
                self.q = initQ
                self.isEmpty = False
                return
            else:
                self.resetQ()
        
        # for initialzing with a val
        if initVal is not None:
            self.q = LinkedList(LinkedNode(initVal))
            self.isEmpty = False
        else:
            self.resetQ()
            
    # reset the q to empty
    def resetQ(self):
        self.q = LinkedList()
        self.isEmpty = True

    # enqueue a val of any type (except None) to the end of the queue
    def enqueue(self, anyVal):
        if anyVal is None: raise TypeError
        self.q.add(LinkedNode(anyVal))
        self.isEmpty = False
        return
    
    def dequeue(self):

        #can't pop because the q is empty
        if self.isEmpty: raise ValueError
        popped = self.q.head
        self.q.head = self.q.head.next

        #After pop the q is empty...
        if self.q.head is None: 
            self.resetQ()
        return popped
    
    def peekFront(self):
        return self.q.head
    
    def peekRear(self):
        return self.q.endPointer
    
    def __str__(self):
        linkedString = str(self.q)
        if linkedString == 'Empty': return linkedString
        return linkedString[:-8] #clean final None print from Linked List str method

def queueFromArr(arr):
    if not arr: raise ValueError
    return Queue(initQ=linkedFromArr(arr))
```

In simple terms, our `Queue` class just extends the Linked List class with the proper naming conventions, print methods, and initialization options. And, since it's made purely with pointers, our Queue is completely generic. 

Only type `None` will be a problem because this is the convention we chose to signal an empty node in the Linked List. 

**So, Linked Lists were the key to making a queue!** In fact, the library I mentioned at the beginning, `deque` is implemented with a doubly linked list! 

## Addendum Why is pop() only O(1) from the end of this list?

**This question is what got me started on this post in the first place, so here's what I found.**

In order for an array to have constant lookup time for all elements, we need to be able to query an element in the array with only its index. 

We can do this with formula:

- `address = startAddress  + (wordSize * index)`

So, if the word size is 4 bytes, maybe our array looks like this in memory.

| Address  | Index |  Value |
| -------  | ----   | ----|
| 0xBD000  | 0 | 'hello' |
| 0xBD004  | 1 |'world' |
| 0xBD008  | 2 |'this'  |
| 0xBD00C  | 3 |'is'    |
| 0xBD010  | 4 |'my'    |
| 0xBD014   | 5| 'array |

For index = 0, the formula gives us `address = startAddress`, and for index = 5, the formula gives us `address = startAddress +  5 * 4 = 0xBD000 + 20 = 0xBD014`. 

Notably, the addressing of each element is relative to its distance from the start address.

#### Pop() Complexity, Revisited

So, if we were to pop the last element of our array, **the formula to calculate the addresses of the remaining elements, and their indexes, would be unchanged**. This is why we have O(1) pops from the end of a list.

If we pop the second to last element, only the address of our new last element, `'array'` would need updating. We update its index from `5` to `4`, shifting `array` up one position.

If we popped the third to last element, we'd do two index updates/shifts and so on. Our worst case is popping the first element, where we'd have to update/shift `n-1` elements. 

**So, this re-indexing and shifting is why we have O(n) complexity to pop arbitrary elements from a list in Python.** As mentioned earlier on, we have to do this re-indexing and shifting because python lists are implemented as dynamic arrays in C.

So, to make a Queue, we had to go through all of the trouble to make a Linked List.

## End

If you've made it this far, I hope you enjoyed. Thanks for reading :)
