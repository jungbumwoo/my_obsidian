
A variable of interface type stores a pair: the concrete value assigned to the variable, and that value’s type descriptor.  
To be more precise, the value is the underlying concrete data item that implements the interface and the type describes the full type of that item.

One important detail is that the pair inside an interface variable always has the form (value, concrete type) and cannot have the form (value, interface type). Interfaces do not hold interface values.

```go
type Reader interface {  // In io. package
 Read(p []byte) (n int, err error)  
}  
  
type Writer interface {  // In io. package
 Write(p []byte) (n int, err error)  
}

var r io.Reader // r contains, schematically, the (value, type) pair, (tty, *os.File).  
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)  
if err != nil {  
   return nil, err  
}  
r = tty  
  
var w io.Writer  
w = r.(io.Writer) //  w will contain the pair (tty, *os.File).  
  
var empty interface{}  
empty = w // empty will contain the pair (tty, *os.File).  
  
return r, nil
```


Go's interfaces let you use [duck typing](http://en.wikipedia.org/wiki/Duck_typing) like you would in a purely dynamic language like Python but still have the compiler catch obvious mistakes like passing an `int` where an object with a `Read` method was expected, or like calling the `Read` method with the wrong number of arguments.
Unlike in languages like Python, if you pass a value with the wrong type, you get an error at compile time, not run time.

Interfaces aren't restricted to static checking, though. You can check dynamically whether a particular interface value has an additional method. 

```go
//Interfaces aren't restricted to static checking, though. You can check dynamically whether a particular interface value has an additional method. 


type Stringer interface {
    String() string
}

func ToString(any interface{}) string {
    if v, ok := any.(Stringer); ok {
        return v.String()
    }
    switch v := any.(type) {
    case int:
        return strconv.Itoa(v)
    case float:
        return strconv.Ftoa(v, 'g', -1)
    }
    return "???"
}
```

```go
type GeneralError struct {  
   err        error  
   needReport bool  
}  
  
func (e GeneralError) Error() string {  
   return e.err.Error()  
}  
  
func (e GeneralError) NeedReport() bool {  
   return e.needReport  
}  
  
func NeedReport(err error) bool {  
   if err == nil {  
      return false  
   }  
  
   if e, ok := err.(interface{ NeedReport() bool }); ok {  // 이렇게도 확인 가능하다.
      return e.NeedReport()  
   }  
   return true  
}  
  
type NotFoundError struct {  
   GeneralError  
}
```

https://research.swtch.com/interfaces
