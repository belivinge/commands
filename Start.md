**Flow of control**:
When the last handler in the chain returns, control is passed back up the chain in the reverse direction.

```secureHeaders - > servemux - > application handler - > servemux - >secureHeaders```

Code which comes before next.ServeHTTP() will be executed on the way down, code after will be executed on the way back up.

**Early returns**:
- Common use-case for early returns is authentication miidleware which only allows execution of the chain to continue

func MiddleWare():
        return http.handlerfunc(func(w, r)){
            if !Aythorized(r){
                w. writeHeader(Forbidden)
                return 
            }
        }
        next.ServeHttp(w, r)

**Panic Recovery**

```panic("oops! something went wrong!")```
```$curl -i http://localost:4000```
```curl (52) Empty reply from server```

it would be more appropriate and meaningful to send them a proper HTTP response with a 500 Internal Server Error status instead.

**Panic Recovery in the Background Goroutines**
if you have a handler which spins up another goroutine then any panics that happen in the second goroutine won't be recovered

to fix it - > create a goroutine: go func()

**Third-party package**
Go is clever enough to recognize that our code is importing a new third-party package and automatically downloads the package - > it always retrieves the latest version

**RESTful Routing Example**
We add a HTML form to the web application so that users can create new snippets

![Web Visualization]()