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

![Web Visualization](https://github.com/belivinge/commands/blob/master/aoaoa.png)

**Installing a Router**
hundred third-party routers: Pat(for simple structure) and Gorilla Mux(for full-featured) are recommended
- Pat matches patterns in order that they are registered

**Parsing from Data**
1) r.ParseForm() method to parse the request body. It checks if the request body is well-formed, and then stores the data in r.PostForm map. If there any errors when parsing the body, then it will return an error.
2) Get the data form r.ParseForm() by using the r.PostForm.Get() method. for example title - > r.PostForm.Get("title")

**Creating template**

``{{$exp := or (.FormData.Get "expires") "365"}}``

it creates a new $exp template variable which uses *or* template function to set the variable to the value holded by .FormData.Get "expires" or if it is empty then the default value of "365" 

we then use this variable in *eq* function to add the checked attribute to the radio button:

``{{if (eq $exp "365")}}checked{{end}}``

**Forms**
If your application has many *forms* then there will be lots of repetition in the code and validation rule.
So, you better create a separate folder where you save your forms package to return to it time by time if there is any error.

**Satetful HTTP**
To display one-time confirmation message - > we need to share data(or state) between HTTP requests for the user. The most common way to do that is to create a *session* for the user

Good session management packages:
- gorilla/sessions
- alexedwards/scs
- golangcollege/sessions  

To add the confirmation message we use the *Session.Put() method. - > ``app.session.Put(r, "flash", "Snippet successfully created!")`` where the second parameter is the key for the data
To retrieve the data we use the *Session.Get() method or alternatively, the *Session.GetString which takes care of the type conversion
Then to remove it, we use the *Session.PopString() method 

**Security Improvements**

- create a self-signed TLS certification
- all responses and requests are served securely over HTTPS
- sensible tweaks to the default TLS settings
- set connection timeouts to prevent slow-client attacks

HTTPS is essentially HTTP sent across a TLS (Transport Layer Security) connection. Because it's sent over a TLS connection the data is encrypted and signed, which helps ensure its privacy and integrity during transit.

TLS - > modern version of SSL (Secure Sockets Layer). 

To start using HTTPS, generate a TLS certificate. For development purposes the simplest thing to do is to generate own self-signed certificate.

``go run /usr/local/go/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost``

how it works:
1. Generates a 2048-bit RSA key, which is cryptographically secure - > public and private key
2. It then stores the private key in a key.pem file, and generates a self-signed TLS certificate for the localhost containing public key which is stored then in a cert.pem file.

![Web Visualization](https://github.com/belivinge/commands/blob/master/bababa.png)

After we set this, our server will still be listening on port 4000 - but now it will be taking HTTPS, not HTTP. And every time it will show "Your connection is not secure" warning.

If you press Ctrl+i using Firefox, the 'Technical Details' section confirms that the connection is encrypted and working as expected.

'Acme Co' is a hard-coded placeholder name. HTTPS also automatically updates to the HTTP/2 version if user connection supports it. It makes pages load faster for users.
The user that is used to run the Go application must have read permissions for both cert.pem and key.pem files, otherwise ListenAndServeTLS() will return a *permission denied* error.

By default, the generate_cert.go grants read permission for all users for the cert.pem file, but read permission for key.pem file only to the owner.  

![Web Visualization](https://github.com/belivinge/commands/blob/master/cacac.png)

If you want to add an ignore rule so the *.pem files are not accidentally committed:

``$ echo 'tls/' >> .gitignore ``

**Restricting Cipher Suites**
It may be desirable to limit HTTPS server to only support cipher suits which use ECDHE(forward secrecy), because there are some weak cipher suits that you might want not to use RC4, 3DES or CBC.

It is done via:

![Web Visualization](https://github.com/belivinge/commands/blob/master/dododod.png)

If there is no timeouts like Idle, Read and Write, then it is easy to get slow-client attacks, known such as *Slowloris* - which can keep a connection open indefinitely by sending incomplete requests - > affected servers will keep these connections open, filling their maximum concurrent connection pool, by time denying additional connection attempts from clients.