# go-styles
Our Best Practises for writing beautiful code in Go

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
