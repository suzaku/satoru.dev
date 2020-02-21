+++
title = "Preallocate, not only pointers but also structs"
date = 2019-10-21T11:03:33+08:00
draft = false
tags = ["golang", "performance"]
+++

If the size of a slice can be decided when it's created, we can specify its initial size to improve performance, because unnecessary copy operations and memory allocations are avoided. Many Go programmers already know this, but when it comes to a slice of pointers, things are a little more interesting.

Let's say we have a simple struct called `Obj`:

```go
type Obj struct {
    id   int
}
```

We want to create *N* `Obj`s and get a slice of `Obj` pointers. Since the size of the slice is known size, we can preallocate the underlying array by specifying the initial size. Here's a way to do this I have seen in some open source projects:

```go
func initObjRefsWithoutPreallocation() []*Obj {
    refs := make([]*Obj, 0, N)
    for i := 0; i < N; i++ {
        obj := Obj{id: i}
        refs = append(refs, &obj)
    }
    return refs
}
```

It uses `make([]*Obj, 0, N)` to preallocate space for the slice, but why do I name it initObjRefs**WithoutPreallocation**? Because it only preallocates space for the pointer slice, but the `Obj` structs are created one by one. We can improve it further:

```go
func initObjRefsWithPreallocation() []*Obj {
    objs := make([]Obj, N)
    refs := make([]*Obj, 0, N)
    for i := 0; i < N; i++ {
        objs[i].id = i
        refs = append(refs, &objs[i])
    }
    return refs
}
```

This time, we not only preallocate the pointer slice but also the struct one. The number of allocations is reduced from *1 + N* to *2* ([complete source code of the benchmark](https://play.golang.org/p/8Om1I4gAdO1)):

```shell
$ go test -benchmem -bench=. -count=3 prealloc_obj_test.go
goos: darwin
goarch: amd64
BenchmarkPreallocaAll-12         2964526               386 ns/op            1792 B/op          2 allocs/op
BenchmarkPreallocaAll-12         3012637               391 ns/op            1792 B/op          2 allocs/op
BenchmarkPreallocaAll-12         3078789               407 ns/op            1792 B/op          2 allocs/op
BenchmarkNoPreallocation-12       681558              1525 ns/op            1696 B/op        101 allocs/op
BenchmarkNoPreallocation-12       728301              1530 ns/op            1696 B/op        101 allocs/op
BenchmarkNoPreallocation-12       782371              1540 ns/op            1696 B/op        101 allocs/op
PASS
ok      command-line-arguments  9.384s
```

As you can see, with **N** set to **100**, the version with preallocation is about 4 times faster with only 2 `allocs/op`. Next time you need a slice of pointers with known size, remember to check if you have a chance to also preallocate the underlying struct slice.
