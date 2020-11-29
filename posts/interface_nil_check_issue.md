## Interface Nil Check Issue in GO

I have been programming in Go for 4 years and recently I came across with an interesting problem.

My interface nil check was not working even the interface was nil. It took some time for me to understand underlying issue, and it
turns out that when you return a struct pointer as a nil from a method, it does not actual nil for an interface value assignment even struct implements all methods of the interface

**Here example:**

This is the dummy ReaderTest interface 
```go
type ReaderTest interface {
	NewReaderBlabla() (*ReaderStruct, error)
}
```

This is the struct which implements `ReaderTest` method.
```go
type ReaderStruct struct {
}

func (rs *ReaderStruct) NewReaderBlabla() (*ReaderStruct, error) {
	return nil, errors.New("dummy error")
}
```

This is the method that uses `ReaderTest` interface as a property
```go
type AnotherStructEncapsulator struct {
	r ReaderTest
}
```

Main method where the magic happens
```go
func main() {
	var err error
	rsDummy := &ReaderStruct{}
	ase := AnotherStructEncapsulator{}
	ase.r, err = rsDummy.NewReaderBlabla()
	if err != nil {
		fmt.Println("ERRR")
		//This is the problem, even struct covers the interface, struct pointer nil is not equal to interface nil.
		fmt.Printf("ReaderTest nil is not equal to ReaderStruct nil %t", ase.r == nil)
	}
}
```

When you run the above main, you will realize that `ase.r == nil` will return false. The reason is that `rsDummy.NewReaderBlabla()` method
returns nil  in a `&ReaderStruct(nil)` structure where its not nil for the `ReaderTest` interface in `AnotherStructEncapsulator`. 
To prevent this problem to happen again, we should return interface itself instead of `*ReaderStruct` for `NewReaderBlabla` method.

So method should be written like
```go
type ReaderStruct struct {
}

func (rs *ReaderStruct) NewReaderBlabla() (ReaderTest, error) {
	return nil, errors.New("dummy error")
}
```

This problem caused an issue that one of services that I wrote to panic in run time.

Lesson has learnt :)


GoPlayground Link: https://play.golang.org/p/hJ6KyfSjh0P

 