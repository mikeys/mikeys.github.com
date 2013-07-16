---
layout: post
title: "Understanding Go Slices Behaviour"
date: 2013-07-15 22:35
comments: true
categories: [Go, golang]
---

During my initial steps learning the Go language,
familiar with pointers, I ran into some difficulties grasping why do "reference types" (as the [Go language spec](http://golang.org/ref/spec#Making_slices_maps_and_channel)
 as well as other Go resources, actually call them) behave the way they do, and how do they really look like under the hood, as the name "reference types" confuses a bit.  

Searching through the web, I came across this ['golang-nuts' forum thread](https://groups.google.com/forum/#!msg/golang-nuts/xQUsmdo6oSs/RJ8SF4NsbowJ) which cleared it all up for me
and hopefully I'll be able to summarize the good parts in this post.

Also note, that although this post focuses on Slices, Maps and Channels
work the same way.

- You may think of slices as a struct with the following 3 fields:  
  -  array (holds a pointer to an array)
  -  len (holds the pointed array's length)
  -  cap (holds the pointed array's capacity)

In Go, arguments are always passed to functions by value (you can argue with that by saying that you can always pass a pointer to the real instance, but even then:
you're actually passing a __copy__ of the original pointer).  
Slices are no different: when passing a slice as an argument (by value), you're actually passing a copy of the struct I described above, to the function.  
The pointer in that 'struct copy' refers to the same array as the original struct, meaning that when you're altering the slice's elements, you're actually altering the pointed array's elements and as a result,
the changes are reflected in the original slice. This part is pretty straightforward.

Now for the more interesting part:  
When you perform operations that alter the struct's fields (as opposed to altering the slice's elements), like expanding the slice using `append(slice)` (resulting a greater `len` and sometimes a greater `capacity`) they will __not__ be reflected in the original struct,
the reason is obvious: all you're doing is altering a __copy__.

Hopefully, the following example will better reflect the above:

``` go main.go
package main

import (
  "fmt"
)

func byValue(slice []int) {
  fmt.Printf("byValue(pre-append) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))

  slice[0] = 3
  slice[1] = 4
  slice = append(slice, 4, 5)

  fmt.Printf("byValue(post-append) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))
}

func byPointer(slicePtr *[]int) {
  slice := *slicePtr
  fmt.Printf("byPointer(pre-append) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))

  slice[0] = 3
  slice[1] = 4
  slice = append(slice, 4, 5)

  fmt.Printf("byPointer(post-append) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))

  *slicePtr = slice
}

func main() {
  slice := make([]int, 2, 4) // Reset slice
  fmt.Printf("main(pre-byValue) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))
  byValue(slice) // Slice's elements are changed, len and cap are still the same.
  fmt.Printf("main(post-byValue) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))

  fmt.Println()

  slice = make([]int, 2, 4) // Reset slice
  fmt.Printf("main(pre-byPointer) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))
  byPointer(&slice) // Slice's elements as well as len and cap are changed.
  fmt.Printf("main(post-byPointer) value: %d, len: %d, cap: %d\n", slice, len(slice), cap(slice))
}

// Output:
//   main(pre-byValue) value: [0 0], len: 2, cap: 4
//   byValue(pre-append) value: [0 0], len: 2, cap: 4
//   byValue(post-append) value: [3 4 4 5], len: 4, cap: 4
//   main(post-byValue) value: [3 4], len: 2, cap: 4
//
//   main(pre-byPointer) value: [0 0], len: 2, cap: 4
//   byPointer(pre-append) value: [0 0], len: 2, cap: 4
//   byPointer(post-append) value: [3 4 4 5], len: 4, cap: 4
//   main(post-byPointer) value: [3 4 4 5], len: 4, cap: 4
```
