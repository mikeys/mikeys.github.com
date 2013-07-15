---
layout: post
title: "Understanding GO Slices Behaviour"
date: 2013-07-15 22:35
comments: true
categories: [Go, golang]
---

During my initial steps learning the GO language,
familiar with pointers, I ran into some difficulties grasping why do "reference types" (as the [golang official docs](http://golang.org/ref/spec#Making_slices_maps_and_channel)
 as well as other golang resources, actually call them) behave the way they do, and how do they really look like under the hood, as the name 'reference types' confuses a bit.  

Searching through the web, I came across
[this](https://groups.google.com/forum/#!msg/golang-nuts/xQUsmdo6oSs/RJ8SF4NsbowJ) thread which cleared it all up for me
and hopefully I'll be able to summarize the good parts in this post.

Also note, that although this post focuses on Slices- Maps and Channels
work the same way.

- You may think of slices as a struct with the following 3 fields:  
  -  array (holds a pointer to an array)
  -  len (holds the pointed array's length)
  -  cap (holds the pointed array's capacity)

In Go, arguments are always passed to functions by value (you can argue with that by saying that you can always pass a pointer to the real instance, but even then:
you're actually passing a __copy__ of the original pointer).  
Slices are no different: when passing a slice as an argument, you're actually passing a copy of the struct I described above, to the function.  
The pointer in that 'struct copy' refers to the same array as the original struct, meaning that when you're changing the slice's elements, you're actually changing the pointed array's elements.
the changes are reflected in the original slice. This part is pretty straightforward.

Now for the more interesting part:  
When you perform operations that alter the struct's fields (as opposed to altering the slice's elements), like expanding the slice using `append(slice)` (resulting a greater `len` and sometimes a greater `capacity`) they will __not__ be reflected in the original struct,
the reason is obvious: all you're doing is altering a __copy__.

Hopefully, the following example will better reflect the above:

``` go
package main

import (
	"fmt"
)

func doSomething(slice []int) []int {
	fmt.Printf("(pre-append) len: %d\n", len(slice))
	fmt.Printf("(pre-append) value: %d\n", slice)

	slice[0] = 3
	slice[1] = 4

	slice = append(slice, 4, 5)

	fmt.Printf("(post-append) len: %d\n", len(slice))
	fmt.Printf("(post-append) value: %d\n", slice)

	return slice
}

func main() {
	fmt.Printf("(pre-doSomething) len: %d\n", len(mySlice))
	fmt.Printf("(pre-doSomething) value: %d\n", mySlice)

	mySlice := make([]int, 2, 4)
	doSomething(mySlice)

	fmt.Printf("(post-doSomething) len: %d\n", len(mySlice))
	fmt.Printf("(post-doSomething) value: %d\n", mySlice)
}

// Output:
//  (pre-append) len: 2
//  (pre-append) value: [0 0]
//  (post-append) len: 4
//  (post-append) value: [3 4 4 5]
//  (post-doSomething) len: 2
//  (post-doSomething) value: [3 4]
```
