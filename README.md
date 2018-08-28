# go-styles
Our Best Practises for writing beautiful and clean code in Go

## 0. Simple is better than complex

 - Name your variables and functions intuitive.
 	- though you can use i for index, v for value
 - Don't format strings with + operator, use fmt.Fprintf for different varibles
 	- it resolves the type issues as well
 - Comment on 'hacks' you wrote, that are not self explanatory.
 	- Don't say if it was hard to write, it should be hard to read

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

