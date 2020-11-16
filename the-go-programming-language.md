# Go characteristics
- Compiled (vs interpreted)
- Concurrent
- Structured (not object oriented)
- Launched in 2009
- C-like syntax
- Static typing (verified during compilation)
- No type hierachy, that is, not object oriented
- Duck typing (if it *looks* like a duck, *swims* like a duck an *quack* like a duck)
- No traditional try/catch error handling

# [Simple REST API w/ GO](https://youtu.be/kd-8mb6HfGA)
```bsah
mkdir golang-rest-api
cd $_
go mod init github.com/arllanos/golang-rest-api
code .
```
In vscode create a new file called `server.go`
```go
package main

func main() {
	
}
```
We are telling Golang compiler that this should compile as an executable rather than a shared library, which should not have neither a main package nor a main function.

After that, still on vscode click **View** -> **Command Pallete** or type **Ctrl+Shift+P** and type **goinstall update/tools.**

Wthin main() func create a new instance of mux router that will hel map the URLs and the HTTP methids to the functions that we're going to create.
- install Mux library `go get github.com/gorilla/mux`
- import library and create the router
- create a handler
```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

func main() {
	router := mux.NewRouter()
	const port string = ":8000"
	router.HandleFunc("/", func(res http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(res, "Up and running...")
	})
	router.HandleFunc("/posts", getPosts).Methods("GET")
	router.HandleFunc("/posts", addPost).Methods("POST")
	log.Println("Server listening on port", port)
	log.Fatalln(http.ListenAndServe(port, router))
}
```
Write the function `getPosts` and `addPosts` in a separate file `route.go`

- If I provide a function named `init()` it is going to be executed automatically when the application starts
- Marshal, transform a variable object to the JSON encoding of it (in the example `posts` variable is an array of struct). Example: `result, err := json.Marshal(posts)`
- Unmarshal, is to put a json encoded data into an object. In REST API, I need the HTTP request body in the request unmarshaled, that is in the corresponding struct type variable:
	- Parsed: `json.NewDecoder(req.Body)`
	- Decoded: it to variable passed by reference using `&variable`. Example: `err := json.NewDecoder(req.Body).Decode(&post)`

# Testing with Go

Go has built-in support for testing, so unlike languages like Java or C#, you don't need to add a unit testing framework to your project. All you need to do is import the *testing* library in your code, have your Go file name end with *_test.go*, write a method with a name starting with *Test*, then run:
```bash
go test
```
## Useful Testing Libraries for Go
[Testify](https://github.com/stretchr/testify). Library that allow to write assertions and mock repositories. It also allows to create test suites and use setup and teardown methods. 
[Resty](https://github.com/go-resty/resty). Simple HTTP and REST client library for Go. Similar to the requests library in Python.

## Basic imports
Import testing library from Golang will allow to use built-in testing functionality in Go.
In the testify library we have the assert package to create assertions and the mock package to create mocks.
To install testify library.
```bash
go get github.com/stretchr/testify
```
To import the testing and assert packages.
```go
import (
	"testing"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/mock"
)
```
> Resources
[Alex Ellis article](https://blog.alexellis.io/golang-writing-unit-tests/)
[Tutorial](https://quii.gitbook.io/learn-go-with-tests/)
