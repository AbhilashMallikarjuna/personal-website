+++
title = 'JSON tags in Golang Structs are case-insensitive'
date = 2024-09-30T12:33:03+05:30
draft= false
tags = ["Golang"]
+++

After debugging an issue for a while, I realized that the JSON tags in Golang structs are case-insensitive.
Below is a code sample that demonstrates this.

```go
package main

import (
    "fmt"
	"encoding/json"
)

type Person struct {
	Name string `json:"Name"`
}

var stringifiedJSON = `{"Name": "mallikarjuna", "name": "abhilash"}`

func main() {
	var person Person
	if err := json.Unmarshal([]byte(stringifiedJSON), &person); err != nil {
		fmt.Println(err)
        return
	}
	fmt.Println(person.Name) // abhilash
}
```
<sub>_[Go playground link](https://goplay.tools/snippet/sk06Okou5oi)_</sub>
\
\
Explanation from the docs:
> To unmarshal JSON into a struct, Unmarshal matches incoming object keys to the keys used by Marshal (either the struct field name or its tag), **preferring an exact match but also accepting a case-insensitive match.**  

Read entire docs [here](https://pkg.go.dev/encoding/json#Unmarshal).
