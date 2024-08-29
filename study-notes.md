# Go

Go is a statically typed, compiled language.
All variables have assigned types (explicitly or implicitly).

**Sources**
- go.dev
- golang.org
- play.golang.org
- golang.org/pkg // standard library

---

- Go is case-sensitive
- Variable and package names are lowercase and mixed case
- Names of public fields have initial uppercase characters
- An initial uppercase character means the symbol is exported (aka public)
- Line feeds ends a statement - no semicolon required
  ```go
  var colors [2]string
  colors[0] = "black"
  ```
- Semicolons are part of the formal language spec
- The lexer add them as needed
- Go is sensitive to whitespace
- If a statement is determined to be complete and the lexer encounters a line feed that means it's the end of the statement and you don't need to add the semi-colon yourself. But that also means you can't add always line feeds freely or break up statements with extra line feeds.
- Built-in functions [golang.org/pkg/builtin](golang.org/pkg/builtin)
  - Members of `builtin` package: the Go compiler assumes that the `builtin` package is always imported
  - `len(string)`
  - `panic(error)` // stops execution, displays error message
  - `recover` // manages behaviour of a panicking goroutine

---

go mod init <module-name> // kinda like cargo new

---

## Example program:

```go
// main.go
package main
import ("fmt")

func main() {
  fmt.Println("Hello")
}
```

Run + build from terminal: 
`go run main.go`
`go run .`
`go build .` // build binary
`./hello` // run binary

---

## Built-in data types

Boolean `bool` 
String `string`
Fixed integer types:
`uint8` `int8`
`uint16` `int16`
`uint32` `int32`
`uint64` `int64`
 
Aliases can be used instead of full type names
`byte` `uint` `int` `uintptr`
In here `uint` and `int` are operating system specified (32 or 64 bits)

Floating `float32` `float64`
Complex numbers `complex64` `complex128` // real and imaginary numbers

Data collections:
`Arrays Slices` for managing ordered data collection
`Maps Structs` for aggregation of values

Language organization
`Functions Interfaces Channels`
In Go, function is a type and that's what makes it possible to take a function and pass it into another function as an argument

Data management
`Pointers` // reference variables

Every type has a default value!
Numeric types have 0 

**Type conversion before using:**
```go
vat anInt int = 6
var aFloat float64 = 42
sum := float64(anInt) + aFloat
fmt.Printf("Sum: %v, Type: %T\n", sum, sum)
```

var myvariable string = "my string"
myvariable := "my String" // := can only be used inside functions

const aConst int = 65

characters are wrapped in single quotes, full strings are wrapped in double quotes

---

`fmt.Println(sum)` // print value
`fmt.Printf("The type of %T is", sum)` // print value's (sum) type

---

For beyond basic math operations, use the math package

```go
import "math"

var aFloat = 3.14159
var rounded = math.Round(aFloat)

// Rounding a fractional number
f1, f2, f3 = 23.5, 68.9, 24.5
floatSum := f1 + f2 + f3 // eg. 165.568888888

floatSum = math.Round(floatSum * 100) / 100 // eg 165.6

circleRadius := 15.5
circumference := circleRadius * 2 * math.Pi
fmt.Printf("circumference: %.2f\n", circumference) // %.2f = a floating number with two digits decimal

```

**Dates and times**

```go
package main
import (
  "fmt"
  "time"
)

func main() {
  n := time.Now() // timestamp date time timezone
  fmt.Println("This is the time now: ", n)

  t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
  fmt.Println("Go launched at ", t)
  fmt.Println(t.Format(time.ANSIC))

  parsedTime, _ := time.Parse(time.ANSIC, "Tue Nov 10 23:00:00 2009") // time.Parse can return an error object and to ignore that error for now, use an underscore
  fmt.Printf("The type of parsed time is %T\n", parsedTime) // should be time.Time
}

```

**Memory is managed by the Runtime**

When you build a Go program, the runtime is included (the Go runtime is statically linked into the application). The runtime operates in the background on separate threads and the application depends on it. Memory is allocated and deallocated automatically. 

When you use complex types such as maps, you have to initialise them correctly. 
Use `make()` or `new()` to initialise maps, slices, and channels.

There's a difference between them!

The `new()` function
  - Allocates but does not initialise memory
  - Results in zeroed storage and you get back a memory address indicating the location of the map
  - If you try to add a key/value, it errors
  - ```go
    // INCORRECT CODE!
    m := new(map[string]int)
    m["theString"] = 42 // add an entry
    ERROR
    ```

With `make()`
  - You allocate and initialise memory
  - The storage is non-zeroed and its ready to accept values
  - ```go
    // CORRECT CODE!
    m := make(map[string]int)
    m["theString"] = 42 // add an entry
    ```
    
Memory is deallocated automatically by the garbage collector that's built into the Go runtime. When it kicks in, it looks for items that are out of scope or set to `nil`, free those from memory and return memory to the memory pool.
[More about garbage collection](talks.golang.org/2015/go-gc.pdf)

## Reference values with pointers

You can declare a pointer with a particular type but you don't have to point it at an initial variable: `var pointer *int` // this is a pointer, not a value, currently nil

```go
anInteger := 42
var p = &anInteger // & means pointing at the memory address of the variable, not at its value 
fmt.Println("Value of p: ", *p)

// If I change a value through its pointer, I'll change the value it's pointing at
// (Similarly to Java and C#)
value1 := 42.12
pointer1 := &value1
*pointer1 = *pointer1 / 21
```

---

## Ordered values in arrays

An array is an ordered collection of values or references. All items are of the same type and an array is not resizable. It's better to use slices to represent ordered collections of values. 
In Go, an array is an object. And like all objects, if you pass it to a function, a copy will be made of the array. You can't easily sort data, or manipulate it, that's why slices are better.

```go
var colors [3]string
colors[0] = "Red"
colors[1] = "Green"
colors[2] = "Blue"

var number = [5]int{5,3,1,2,4}
fmt.Println(len(number))
```

## Ordered values in Slices

A slice in Go is an abstraction layer that sits on top of an array. When you declare a slice, the runtime allocates the required memory and creates the array in the background but returns the slice. All items are of the same type. But, a slice is resizable. And they can be sorted easily. 

```go
var colors = []string{"Red", "Green", "Blue"} // NO size is specified and that's how you create a slice!
colors = append(colors, "Pink")

// To remove an item, also use `append()` but indicate a range
colors = append(colors[1:len(colors)]) // start with the item in array index 1 and also go get the rest of the data to the end of the slice
colors = append(colors[:len(colors)-1]) // deleting last item in the array

// Another way to initialise a slice
numbers := make([]int, 5, 6) // type, items, last argument = max capacity until the slice can expand; you can leave it out

// Sort a slice with the sort package
import "sort"
sort.Ints(numbers) // numerical order lowest to highest
```

## Unordered values in Maps

In Go, a map is an unordered collection of key/value pairs (aka hash table).
A map's keys can be any type that's comparable, that is the keys can be compared to each other.
Order of display doesn't matter and you can't rely on the order of items to always be the same.
If you want to guarantee an order you have to manage it yourself (eg. extract keys first and sort them then add another loop with an index)

```go
states := make(map[string]string])
states["WA"] = "Washington"
states["OR"] = "Oregon"
states["CA"] = "California"

california := states["CA"] // retrieve item
delete(states, "OR") // remove item
states["NY] = "New York" // add item

// Iterate
for k, v := range states {
  fmt.Printf("%v: %v\n", k, v) // %v = verb
}
```

## Group related values in structs

Data structure. They can encapsulate data and methods. A struct is a custom type. 

```go
// Dog is a struct
type Dog struct {
  Breed string // uppercase exported to the rest of the app (public)
  Weight int
}

// in main()
poodle := Dog{"poodle", 10}
fmt.Printf("%+v\n", poodle) // to print fields with values
poodle.Weight = 9 // update value
```

**Attach methods to a type**

In Go, a method is a member of a type
Go doesn't support method overrides, each methods need its own unique name.

```go
type Dog struct {
  Breed string // uppercase exported to the rest of the app (public)
  Weight int
  Sound string
}

// d = is the receiver, a reference to a Dog object; Speak is the function name
// Speak is how the dog speaks // !important: Format is "Function name" followed by "is" | "Speak is" 
func (d Dog) Speak {
  fmt.Println(d.Sound)
}

// SpeakThreeTimes is ...
func (d Dog) SpreakThreeTimes {
  d.Sound = fmt.Sprintf(%v %v %v", d.Sound, d.Sound, d.Sound )
}

// in main()
poodle := Dog{"poodle", 10, "Woof"}
poodle.Speak()
poodle.Sound = "arf"
```

---

## Conditions

### IF statements 

```go
theAnswer := 42
var result string

// variation 1
if theAnswer < 0 { // braces must start on the same line
  result = "less than zero"
} else if theAnswer == 0 {
  result = "equal to zero
} else {
  result = "greater"
}

// variation 2
// to initialise variable as part of the if 
if x := -42; x < 0 {
  result = "less than 0"
} else if x == 0 {
  result = "eq"
} else {
  result = "grater"
}
// x is unavailable here
```

### SWITCH

Same as other languages but expands. No use of the `break` because as soon as it's evaluated and executed, it jumps out of the switch.

```go

// variation 1
switch value {
  case 1:
    result = "something 1"
  case 2:
    result = "somethong 2"
  default:
    result = "another"
}

// variation 2
// with initialisation
switch value := rand.Intn(7) + 1; value {
  case 1:
    result = "something 1"
  case 2:
    result = "somethong 2"
    fallthrough // add in if you want fallthrough
  default:
    result = "another"
}
```

---

## Loops with for statements

```go
// variation 1
colors := []string{"Red", "Green", "Blue"}
for i := 0; i < len(colors); i++ {
  fmt.Println(colors[i])
}

// variation 2
// same logic
for i := range colors {
  fmt.Println(colors[i])
}

// foreach loop
// same result
for _, color := range colors {
  fmt.Println(color)
}

// while 
value := 1
for value < 10 {
  fmt.Println(value)
  value++
}

// go to label
sum := 1
for sum < 1000 {
  sum += sum
  fmt.Println(sum)
  if sum > 200 {
    goto theEnd // label
  }
}

theEnd : fmt.Println("end of program")
```

---

## Functions

Functions can be organised into their custom packages.

```go
func main() {
  doSomething()

  sum := addValues(5,8)

  multisum := addAllValues(1,2,3,4)

  multisum, multicount := addAllValuesMultiReturn(1,2,3,4)
}

func doSomething() {
  fmt.Println("doing soimeting")
}

func addValues(value1 int, value2 int) int { // see note below
  return value1 + value2
}
// note: if a function receives multiple arguments of the same type, it's not necessary to add the type to each
// instead, you can do this
func addValues(value1, value2 int) int {
  return value1 + value2
}

// arbitrary number of values
func addAllValues(values ...int) int {
  total := 0
  for _, v := range values {
    total += v
  }

  return total
}

// multi return
func addAllValuesMultiReturn(values ...int) (int, int) {
  total := 0
  for _, v := range values {
    total += v
  }

  return total, len(values)
}
```

---

## Work with files and the web

**Write and read local text files**

Important to use the `defer` keyword whenever you're working with anything that might not run atuomatically in the current thread

```go
import (
  "io"
  "io/ioutil"
  "fmt"
  "os"
)

func main() {
  content := "Hello from Go"
  file, err := os.Create("./fromString.txt")
  checkError(err)

  // wriute to the file
  length, err := io.WriteString(file, content)
  checkError(err)
  // length will return the number of chars written to the file

  // close the file
  defer file.Close() // defer = wait until everything is done, then execute this command

  // Read from file
  defer readFile("./fromString.txt")
}

func checkError(err error) {
  if err != nil {
    panic(err)
  }
}

func readFile(filename string) {
  data, err := ioutil.ReadFile(filename)
  checkError(err)
  fmt.Println("Text read from file: ", string(data))
}
```

**Read a text file from the web + Parse JSON formatted text** 

`http` package to send requests

```go
package main
import (
  "fmt"
  "net/http"
  "io/ioutil"
  "encoding/json"
  "strings"
)

const URL = "url.com"

func main() {
  resp, err := http.Get(URL)
  if err != nil {
    panic(err
  }

  fmt.Printf("response type %T\n", resp) // pointer to *http.Response

  defer resp.Body.close()

  // now read content
  bytes, err := ioutil.ReadAll(resp.Body)
  // check error..
  content := string(bytes)
  fmt.Print(content)

  // read parse JSON
  tours := toursFromJson(content)
  for _, tour := range tours {
    fmt.Println(tour.Name)
  }
  
}

func toursFromJson(content string) []Tour {
  tours := make([]Tour, 0, 20)
  decoder := json.NewDecoder(strings.NewReader(content))
  _, err := decoder.Token()  // error check
  if err != nil {
    panic(err)
  }

  var tour Tour
  for decoder.More() { // .More reads the next available object in the json content
    err := decoder.Decode(&tour)
    if err != nil {
      panic(err)
    }
    tours = append(tours, tour)
  }
  return tours
}

type Tour struct {
  Name, Price string // both has the same type so can go on one line
  // it's okay if they start with uppercase but the json key is lowercase - the parse knows how to match them up
}
```

---

# Standard Library

Print and Println

Printf and Sprintf

Strings


