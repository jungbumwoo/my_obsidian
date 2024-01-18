
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

