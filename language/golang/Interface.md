
A variable of interface type stores a pair: the concrete value assigned to the variable, and that value’s type descriptor.  
To be more precise, the value is the underlying concrete data item that implements the interface and the type describes the full type of that item.

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

