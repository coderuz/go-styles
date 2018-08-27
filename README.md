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

// Good
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
