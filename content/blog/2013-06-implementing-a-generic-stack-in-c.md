+++
aliases = ["/development/2013/06/19/implementing-a-generic-stack-in-c/"]
date    = "2013-06-19T22:02:15-05:00"
tags    = ["c", "data structures"]
title   = "Implementing a Generic Stack in C"
+++

In a [previous post]({{< relref "2013-05-implementing-a-generic-linked-list-in-c.md" >}}) I went over how we could
create a generic linked list implementation in C which would allow the caller to determine that type of information
stored in the list (via a `void *`).

In accordance with my desire to share and the nerdy, sadistic, love/hate relationship I have with C, I'm going to cover
how we can use the linked list code from the previous post to create a generic stack implementation with very little
effort.

## Why Use The Linked List?

The way a stack works is last-in, first-out (LIFO) like a stack of plates. This means that we will be doing all of the
work on one end of the collection of items. We could implement this with an array by using the end of the array as the
push/pop end. However, for large implementations this _**<u>could</u>**_ (depending on the implementation) be
problematic as we don't generally return the memory back to the heap as nodes are popped off.

Without sparking a debate, the most efficient way I could think of would be to use the linked list, which makes `push`,
`pop` and `peek` operations `O(1)` and allocates only enough memory to hold existing items on the stack (_plus the 4
byte next pointer to each node_).

And of course...we'll use a linked list because we already have a [generic implementation]({{< relref
"2013-05-implementing-a-generic-linked-list-in-c.md" >}}) available to us.

## The Code

{{< gist pseudomuto 6334735 "stack.h" >}}

{{< gist pseudomuto 6334735 "stack.c" >}}

As we can see the code is pretty simple since most of the heavy lifting is done by the [linked list]({% post_url 2013-05-02-implementing-a-generic-linked-list-in-c %}).

## Quick Overview of the Code

### stack_new

Stack new takes three arguments, a pointer to the stack instance, the size of each element that will be stored on the stack (to support `malloc/memcpy` operations) and finally a pointer to a function that will be called for each item when `stack_destroy` is invoked. All the heavy lifting is done by the linked list.

### stack_destroy

This method takes on argument, a pointer to the stack to destroy. It first destroys the list (and all child elements) and then frees the memory for the list.

### stack_push

This method takes two arguments; a pointer to the stack and the element to be pushed onto the stack. The method delegates most of the work to the list element by calling `list_prepend`.

### stack_pop

This method takes two arguments; a pointer to the stack and the address to copy the item to. All the details are handled by the list, we simply call `list_head` supplying the list, address and `true` (to remove the item).

### stack_peek

This method is identical to `stack_pop` except that it does not remove the item from the stack.

### stack_size

This method simply returns the lists size

Easy enough right?
