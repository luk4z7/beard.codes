---
title: Type assertions
subtitle: A type assertion provides access to an interface value's underlying concrete value.
date: 2018-09-21
tags: ["go"]
---

Mechanism used to access a value of an `interface{}`

<!--more-->

As is natural in `Go` we always use static types for function parameters or returns, we are in doubt in some 
contexts of how not to duplicate our code when we create a function to return a particular type and another function 
with the same structure returning another specific type, to take advantage of the same structure we can use `interface{}` 
and `Type assertions` mechanism that provides access to the value of the `interface{}`.


```go
var i interface{} = "hello"
```

Example of `Type Assertions` with a `i` variable, knowing that the value of the` i` variable is a `string`, we have
the following `Assertion`
```go
s := i.(string)
fmt.Println(s) // hello
```

in this example we know exactly the type of value that the interface received, but when we try to do
an assertion of a value that does not match the value of `i`?
```go
f = i.(float64)
fmt.Println(f)
```

we received a `panic: interface conversion: interface {} is string, not float64` error, to deal with
this error we must use a Boolean validation to identify if `assertion` was successful.
```go
if f, ok := i.(float64); ok {
    fmt.Println(f)
} else {
    fmt.Println("not a float")
}
```


to be more practical we can create a check with `Type switches`, with this verification we can
condition more types and/or `structs`
```go
package main

import "fmt"

func check(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("%v is int\n", v)
	case string:
		fmt.Printf("%s is string\n", v)
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	check(21)
	check("hello")
	check(true)
}
```

with that simple piece of code that can be very useful to us on the day by day to perform `Type assertions` and reuse
our code with different types in `Go`.

##### Referências

 - [Type assertions](https://tour.golang.org/methods/16)
