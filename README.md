# go-styles
Our Best Practises for writing beautiful and clean code in Go

 ## 0.1 Naming Conventions

   ### 0.1.1 Naming Variables, CamelCase/camelCase, not snake_case
 ```go
 // Make sure you name variables with Nouns and with its role
 CountrCode := "UZ"
 score := 50.0
 
 // Booleans with adjectives
 alive := true
 
 // or
 isSent := false
 ```
 
   ### 0.1.2 Naming Functions
  ```go
  // The same for functions, use CamelCase/camelCase
  // name them with verbs
  func increaseInt(i int) int {
    return i++
  }
  
  
  // keep them short  and descriptive, unlike
  func increaseNumberByOneAndThisIsLongName(){
    // Don't forget indenting
  }
 ```
 
   ### 0.1.3 Naming Structs, Maps, Arrays
  ```go
  // for Structs and Maps it's handy to use sigular nouns
  
  type Person struct {
    name string
    age int
  }
  
  priceList := map[string]float{
    "notebook": 15.0,
    "pen": 24.99,
    "bag": 149.99, 
  }
  
  // And arrays and slices - plural
  people := [5]Person{}
  odds := []int{1,3,5,7,9}
 ```
 
   ### 0.1.4 Naming Errors
 ```go
  // Don't use the same variable - err for every error
  // Here is the standard we use
  ErrReadFail := errors.New("could not read file")
  
  // and
  ErrFileExists := errors.New("File Already Exists")
  
  // but not
  FileExistsError := errors.New("File Already Exists")
 ```
 
   ### 0.1.5 Naming Packages
  ```go
  `it's bad practice to name packages utils, helpers, ...`
  `Use descriptive names and keep them short`
  `standard library packages are the best example`
  // such as one word describes all
  import "archive"
  // I can intuitively guess there are zip, tar and more compressors and uncompressors
  // it was not named "tar.zip.things"  
 ```
 
 ## 0.2 Project Structure
 
   ### 0.2.1 Ordering imports
  ```go
  // Group them the following order:
  // first standard packages
  // then external packages
  // then project internal packages
  import (
    "fmt"
    "net/http"
  
    "github.com/gorilla/mux"
  
    "./handlers"
  )
  
  // Order group alphabetically
  // There are many extentions that take care of it
 ```
 
   ### 0.2.2 Structuring of main package
  ```go
  // it's bad idea to declare variables outside the main function,
  // though you may define constants outside.
  
  import "net/http"
  
  const (
    API_ROOT = "https://example.com/api"
    STATUS_OK = 200
    ...
  )
  
  func main(){
    answer := otherFunc()
    ...
  }
  
  func otherFunc() int {
    return 42
  }
  
  func anotherFunc(){
    // some code
  }
  
 ```
 
   ### 0.2.3 Structuring packages
  ```go
  `Seperate package into logical parts and store them in different files`
  `Each having the same package name`
  // I will continue from here next time
 ```
 
 ## 0.3 Documenting Code
 
 ## 0.4 Methods vs Functions
 
 ## 0.5 Go Routines
 
 ## 0.6 Performance vs Readability
 
 ## 0.7 Testing
 
 ## 0.8 Indenting, Encoding, Spaces vs Tabs
 
  ### 0.8.1 Indenting
  Always indent code for better readablity, Preferably with 2 space indent level. We recommend goformat extenttion for visual studio code which formats your code when you save the file.
 ```go 
 // Bad:
 w = Wheel{Circle{Point{8, 8}, 5}, 20}
 ```
 with visual indent code is much more readible
 ```go
 // Better
 w = Wheel{
    Circle: Circle{
       Point: Point{X: 8, Y: 8},
       Radius: 5,
    },
    Spokes: 20,
 }
 ```
 ### 0.8.2 Encoding
  UTF-8
 ### Spaces vs Tabs
 [![Tabs versus Spaces - YouTube](https://img.youtube.com/vi/V7PLxL8jIl8/0.jpg)](https://www.youtube.com/watch?v=V7PLxL8jIl8) 
 
 ## 0.9 IDE vs Editor
  - Statistics of our usage:
    1. 50% - Visual Studio Code
    2. 25% - Sublime Text
    3. 25% - GoLand by JetBrains
 



## 1. Flat is better than nested
```go
// Bad:
var bd, err = sql.Open("postgres", "user=USER password=PASSWORD")
if dbErr == nil {
	var rows, queryErr = db.Query("SQL Query")
	if queryErr == nil {
		// Some Code
	}
}
```
```go
// Good:
var bd, err = sql.Open("postgres", "user=USER password=PASSWORD")
if dbErr != nil {
	// Handle error
}
var rows, queryErr = db.Query("SQL Query")
if queryErr != nil {
	// Handle error  
}
//Some Code
```


## 2. Don't Repeat Yourself
```go
// Bad:
http.HandleFunc("/event/", eventHandler)
http.HandleFunc("/events/", eventsHandler)

func eventHandler(w http.ResponseWriter, r *http.Request) {
	// Write Header
	w.Header().Set("Content-Type", "application/json")
	// Some Code
	fmt.Fprintf(w, `{"text": "Event Registered"}`)
}

func eventsHandler(w http.ResponseWriter, r *http.Request) {
	// Write Header
	w.Header().Set("Content-Type", "application/json")
	// Some Code
	fmt.Fprintf(w, `{"text": "Events Registered"}`)
}
```


```go
// Good:
http.HandleFunc("/event/", Jsonify(eventHandler))
http.HandleFunc("/events/", Jsonify(eventsHandler))

// Reusable Func, like Decorators in Python
func Jsonify(f func(http.ResponseWriter, *http.Request)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "application/json")
		f(w, r)
	}
}

func eventHandler(w http.ResponseWriter, r *http.Request) {
	// Some Code
	fmt.Fprintf(w, `{"success": 1, "success_text": "Registered"}`)
}

func eventsHandler(w http.ResponseWriter, r *http.Request) {
	// Some Code
	fmt.Fprintf(w, `{"success": 1, "success_text": "Registered"}`)
}
```

```go 
// or Even Better
http.HandleFunc("/event/json/", ContentType("application/json", jsonHandler))
http.HandleFunc("/event/xml/", ContentType("application/xml", xmlHandler))

func ContentType(ct string, f func(http.ResponseWriter, *http.Request)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", ct)
		f(w, r)
	}
}

func jsonHandler(w http.ResponseWriter, r *http.Request) {
	// Some Code
	fmt.Fprintf(w, `{"success": 1, "success_text": "Registered"}`)
}

func xmlHandler(w http.ResponseWriter, r *http.Request) {
	// Some Code
	fmt.Fprintf(w, `<?xml version="1.0" encoding="UTF-8"?>
			<root>
			   <success>1</success>
			   <success_text>Registered</success_text>
			</root>`)
}
```


## 3. Avoid Concurrency in your `not-main` functions
```go
// Not Good
func doConcurrently(job string, err chan error){
	go func(){
		fmt.Println("doing job:", job)
		time.Sleep(1 * time.Second)
		err <- errors.New("Something Went Wrong") 
	}
}

func main(){
	jobs := []string{"one", "two", "three"}
	
	errc := make(chan error)
	for _, job := range jobs {
		doConcurrently(job, errc)	
	}
	
	for range jobs {
		if err:=<-errc; err!=nil{
			fmt.Println()
		}
	}
}

```

```go
// Better
func do(job string){
	fmt.Println("doing job:", job)
	time.Sleep(1 * time.Second)
	return errors.New("Something Went Wrong") 
}

func main(){
	jobs := []string{"one", "two", "three"}
	
	errc := make(chan error, len(jobs))
	for _, job := range jobs {
		go func(job string){
			errc <- do(job)	
		}(job)
	}
	
	for range jobs {
		if err:=<-errc; err!=nil{
			fmt.Println()
		}
	}
}
```

## Then ...
```go
// To be continued
```
