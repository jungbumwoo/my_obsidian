


사용하고 있는 http package의 경우 client


builtin package
```go
// The error built-in interface type is the conventional interface for  
// representing an error condition, with the nil value representing no error.  
type error interface {  
  Error() string  
}
```


// net.pprof
```go
// context.DeadlineExceeded 로 타임아웃을 잡고 있음.
select {  
case <-r.Context().Done():  
  err := r.Context().Err()  
  if err == context.DeadlineExceeded {  
   serveError(w, http.StatusRequestTimeout, err.Error())  
  } else { // TODO: what's a good status code for canceled requests? 400?  
   serveError(w, http.StatusInternalServerError, err.Error())  
  }  return  
case <-t.C:  
}
```

