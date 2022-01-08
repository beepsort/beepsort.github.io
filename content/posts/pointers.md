---
title: "Pointers"
date: 2022-01-08T17:13:55Z
draft: false
---

I see many people getting confused regarding pointers and over time I have noticed some common areas people get confused, so I decided to write a post explaining some of these areas.

# The Theory behind Pointers

## Some background definitions

### Lifetimes
A lifetime describes when a variable is first given space in the program's memory and when that memory will be
released.

### The Stack
The stack is the most important area of memory. Every function call will allocate some space on the stack to store it's variables; this is called a function's stack frame. After a function returns, it's stack frame and all it's variables are deleted and will potentially be overwritten by any other future function calls.

Variables created on the stack are assigned a size at compilation time, this means your source code must know the specific size required to store it on the stack. The stack is much smaller than the heap and has more possibility of filling up (a stack overflow); it is also very fast compared to the heap, especially on modern processors.

### The Heap
The heap is a much larger area of memory than the stack. It stores data in an unordered manner, for any part of the program that requests to store data on the heap. Space on the heap must be allocated before use, so the area of the heap is known to be in use. After the data is no longer required to be stored on the heap, it needs to be deleted from the heap to allow the space to be used again for new data.

#### Dynamic sizing
The size of a stack frame for a specific function is decided at compile time, this means all variables assigned on the stack need to have a predefined size. If we do not know the size of some data until the program runs, then we may need to use the heap to allocate the specific amount needed.
#### Sharing data
If we want to access a single shared data resource at multiple points in the program, we can store the data on the heap and allow each area of the program to access the single location.
#### Control over variable's lifetime
If we do not want a variable to be deleted after the function it was created within ends, then this may be a reason to use the heap instead of the stack.

## Why use pointers?

### Reducing the amount of copying
In C and C++, passing a value as a function parameter or assigning a value to a new variable will copy the direct value of the variable. In the case of large structs or objects, this can result in many unnecessary copies being made; as a result this can waste memory or slow down our code. This can also allow a function to modify the value of it's parameter, and for that modification to apply outside of that function.

### Using the heap
We often do not want to allocate data on the stack and instead need to use the heap. When we use the heap, we must use pointers to access data stored on the heap, as well as control when the data is freed up from the heap.

## What is a pointer?
When we create a variable, it is stored at a location specified by a memory address; a pointer is **a variable that can store a memory address of another variable**. Notice that a pointer is itself a variable; this means a pointer is subject to all the same details as a regular variable. Pointers need to be initialized just like regular variables, they can be stored on both the stack and heap, and they themselves can also have pointers point to them too! When we create a pointer, we are creating space to store a memory address, but not creating anything for the pointer to actually point to.

# Practical usage

## Pointer types

```c
int *a_ptr;
```

This type definition says two things:
- `*a` means `a` is a pointer variable
- `int` means `a` points to an integer variable

## Dereferencing

```c
int a = *a_ptr;
```

We are using the `*` operator again, but this time it is in an _expression_ rather than a _type definition_. When we use the `*` operator in an expression it behaves as the dereference operator. The dereference operator will take the value stored in the pointer (a memory address) and look at the location in memory at that memory address.

In this example, the program takes multiple steps:
1. load the _memory address_ stored at `a_ptr`
2. read the _value_ stored at the _memory address_
3. store the _value_ in `b`

It is also possible to write to the location of a pointer

```c
*a_ptr = 7;
```

In this second example, the program performs the following:
1. load the _memory address_ stored at `a_ptr`
2. store `7` as the _value_ of the location pointed to by the _memory address_

Note that in both of these examples, we haven't initialized the value of `a_ptr`. This is a big problem, as it means we are trying to access an undefined memory location. This could mean that our program tries to read/write to an area of memory it is not allowed to, which would cause a crash (segmentation fault); or even worse, our program could read/write an allowed part of our program's memory and continue with corrupted data! To stop this, lets look at one of the ways we can assign a value to a pointer.

## Address operator

```c
int a = 7;
int *a_ptr = &a;
```

Here we use the address operator `&`. The address operator will get the memory address of it's operand. So like before, let's break the program down:
1. create a memory location to store an integer `a`
2. create a memory location to store a pointer `a_ptr`
3. store 7 as the _value_ of `a`
4. get the _memory address_ of `a` as specified by `&a`
5. store the _memory address_ of `a` as the _value_ of `a_ptr` (remember `a_ptr` is itself a variable that holds a memory address as it's value)

## Arrays

```c
int arr[5] = {1,1,2,3,5};
```

In C, arrays are a special type of pointer, which has an additional size component as part of the type. When an array is passed to a function as a parameter, the type decays to a regular pointer (the size information is lost). Other than this, arrays are simply just pointers to the first element in the array.

```c
int arr[5] = {1,1,2,3,5};
*arr = 10;
```

This would modify the first element (`arr[0]`) to be 10, leaving the array to be `{10,2,3,4,5}`.

## Address arithmetic

```c
int arr[5] = {1,1,2,3,5};
int a = *(arr+2);
int b = arr[2];
```

It is possible for us to manipulate the value of a pointer, which will modify the memory location that is being pointed to. In this example, both a and b will be the same value (3).

By adding 2 to the pointer, we move 2 elements forwards in memory and arrays are stored contiguously (the elements are stored directly one after another in memory). Address arithmetic modifies the memory address by increments the size of the data type being pointed to, in this case an integer, which is 4 bytes in most 64-bit machines.

## Heap allocation

### C

#### malloc

```c
int *a_ptr = malloc(sizeof(int));
```

To use the heap we need to allocate space for our data, let's break our program down again:
1. create a memory location to store a pointer `a_ptr`
2. call malloc and request memory on the heap to store a single integer
3. receive a pointer from malloc to the allocated memory location
4. store the returned pointer as the _value_ of `a_ptr`

At this point we are free to use this memory location as we have with previous pointer variables, except it is stored on the heap instead of the stack now and has the trade offs mentioned earlier in the theory section.

One note regarding memory allocation with malloc, is memory allocation can fail if there is no space for the memory that was requested, in this circumstance malloc will return 0 (NULL).

Allocating an array in C requires calculating the size of the data type, multiplied by the number of elements.

```c
int *a_ptr = malloc(4*sizeof(int));
```

This creates an array of 4 integers on the heap.

#### free

As mentioned earlier, once we allocate memory on the heap, we also need to free it after we are finished with it, we do this using the free function.

```c
int *a_ptr = malloc(sizeof(int));
free(a_ptr);
```

Calling free marks the memory we were previously using as being able to be used elsewhere. If we were not to do this, we can cause a memory leak; where memory usage of our program keeps growing until it potentially crashes.

Freeing arrays is no different to any other data.

### C++

#### new

C++ has it's own version of the malloc function with the `new` keyword. It is somewhat similar to malloc, with a few differences:
- Initializes created objects by calling their constructors
- Throws an exception on memory allocation failure instead of returning 0
- Does not require using sizeof() calculations
- Has a specific array syntax

Creating a heap allocated integer
```c++
int *a_ptr = new int;
```

Creating a heap allocated array of 4 integers
```c++
int *a_ptr = new int[4];
```

#### delete

C++ also has it's own version of the free function with the `delete` keyword. Again it is similar to free, with a few differences:
- Calls object destructors
- Has a specific array syntax

Deleting a heap allocated variable
```c++
delete a_ptr;
```

Deleting a heap allocated array
```c++
delete [] a_ptr;
```
