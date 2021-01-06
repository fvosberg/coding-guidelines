# coding-guidelines

This coding guidelines are intended to give guidelines on typical code style
discussions, mainly in Go. 

This is an addendum to: https://github.com/golang/go/wiki/CodeReviewComments

I want to also read: https://gist.github.com/adamveld12/c0d9f0d5f0e1fba1e551

## Golang

### String Pointer

A quite common discussion is whether to use string pointer in Go. Sometimes they
are useful, but the most of the time, I believe they should be avoided. Most of
the time this discussion comes up, when a JSON should be decoded into a struct
and string fields are optional.

#### Pro-Argument: Fail fast

```
You should use string pointers in fields, which are required to be set, to
ensure your code fails, when the check is missing.
```

The argument is fail fast. This is a valid argument, but I don't like the
drawbacks of this. Because if we miss the required check, our code panics. 

#### Pro-Argument: Make the optional field obvious

```
A pointer draws the attention to this field and shows, that this field is
optional.
```

This is true, because we are used to it. But I argue, that it also sends the
wrong impressions to the developers, who understand the language mechanics
behind it. Pointers are made to share a value and manipulate it outside of the
current functions frame. A string on itself can be empty, as an empty string.
That this is possible should always be the assumption as a developer. 

#### Pro-Argument: You can omit empty fields in JSON

```
You have to use pointers, so you can omit this field, if not set.
```

You don't have to use pointers for that. You can omit also empty strings, as you can see in the code example. The only difference is, that you can't differentiate between empty string and not set.

[https://play.golang.org/p/IIEKR4tSavq](Go Playground)

```
type Test struct {
	SetToNil         *string `json:"setToNil,omitempty"`
	SetToEmptyString *string `json:"setToEmptyString"`
}

func main() {
	var s string
	t := Test{
		SetToNil:         nil,
		SetToEmptyString: &s,
	}

	b, err := json.Marshal(t)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}
```

#### Pro-Argument: You can differentiate between an empty string and not set

```
With being a string pointer, you can differentiate between an empty string and
the string not beeing set. E.g. after unmarshalling JSON.
```

This is true, if you want to differentiate these two cases, using a string
pointer is an easy option to achieve this.

#### Counter-Argument: Panic through nil pointer dereference

```
If string pointers are used, the code can panic due to nil pointer dereference.
```

This is the case with all pointers. This is not a counter-argument on itself,
but it's a drawback of pointers. So they come with a cost. 

#### Counter-Argument: Misleading pointer semantic

As William Kennedy points out, pointer semantics means, that you share the value
of a variable with functions outside of the currents function frame, so they can
manipulate it (https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html).

If you don't intend to do that, you should have a very good reason to use a pointer IMHO.

```go
package main

import (
	"encoding/json"
	"fmt"
)

type foo struct {
	Random string  `json:"random,omitempty"`
	Bar    *string `json:"bar"`
}

func main() {
	bar := "I'm happy"
	f := foo{
		Bar: &bar,
	}
	onlyPassedByValue(f)
	b, err := json.Marshal(f)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b)) // prints {"bar":"OVERWRITTEN"}
}

func onlyPassedByValue(f foo) {
	*f.Bar = "OVERWRITTEN"
}
```

#### Counter-Argument: Defining string pointer is cumbersome

``` go
type Foo struct {
	Bar *string
}

func main() {
	f := Foo{
		Bar: &"bar", // this will fail with cannot take the address of "bar"
	}
}
```

You will have to define it in two steps, call an inline function or define a
function for that.

```
	b := "bar"
	f := Foo{
		Bar: &b,
	}
```

```
func main() {
	f := Foo{
		Bar: stringPointer("bar"),
	}
}

func stringPointer(s string) *string {
	return &s
}
```


Made with ♥ by Fredi
