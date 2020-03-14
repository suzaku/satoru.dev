+++
title = "Don't Treat append Like a Pure Function"
date = 2020-03-14T11:20:58+08:00
tags = ["golang"]
categories = ["Programming Languages"]
+++

In *Go*, we add elements to a slice by calling the built-in `append` function, one typical use case is:

```go
names = append(names, "satoru")
```

Unlike programming languages like *Python* or *Java* in which we call some append method on a list and update it in place, in *Go*, we have to assign the returned value of `append` back to the variable itself.

Judging from the function signature, `append` seems like a pure function, but **this is not the case**.

Let's quote a statement in the official [document](https://golang.org/pkg/builtin/#append) of `append`:

>  If it has sufficient capacity, the destination is resliced to accommodate the new elements.

What does that mean? Let's look at an example:

```go
a := make([]int, 0, 4)
a = append(a, 1)
a = append(a, 2)
x := append(a, 3)
y := append(a, 4)
fmt.Println(a)  // => [1 2]
fmt.Println(x)  // => [1 2 4]
fmt.Println(y)  // => [1 2 4]
```

Both `x` and `y` would end up being `[1 2 4]`, why's that?

The slice `a` has a capacity of `4` and a length of `2`, it has sufficient capacity for two more elements. When we later call `append(a, 3)` and `append(a, 4)`, the array underlying `a` is actually reused. This would be clear if I print some more information in the example:

```go
a := make([]int, 0, 4)
a = append(a, 1)
a = append(a, 2)
fmt.Printf("Address: %p, Len: %d, Cap: %d\n", &a[0], len(a), cap(a))
x := append(a, 3)
fmt.Printf("Address: %p, Len: %d, Cap: %d\n", &x[0], len(x), cap(x))
y := append(a, 4)
fmt.Printf("Address: %p, Len: %d, Cap: %d\n", &y[0], len(y), cap(y))
fmt.Println(a)
fmt.Println(x)
fmt.Println(y)
```

The output would be like (you may see different array addresses):

```
Address: 0xc000098000, Len: 2, Cap: 4
Address: 0xc000098000, Len: 3, Cap: 4
Address: 0xc000098000, Len: 3, Cap: 4
4
[1 2]
[1 2 4]
[1 2 4]
```

As you can tell from the identical addresses of the first element, the three slices are pointing to the same underlying array. When we call `append` to add a third element to `a`, it's actually like setting the third element of the underlying array, that's why `x` also becomes `[1 2 4]`.

So `append` is not actually a pure function, don't treat it like one! If you want to play with the code yourself, here's a link to the [playground](https://play.golang.org/p/OeqEZ7_3R32).