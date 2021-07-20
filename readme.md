### Network
##### TCP vs UDP
TCP creates a connection and guarantees message delivery thus is pretty slow (due to delivery confirmation messages)
UDP doesn't require a connection. It also doesn't guarantee message delivery or even message order.
##### NAT
Network address translation provides translation from private IPv4 addresses to a public address to make it possible for devices to communicate pith public.
##### HTTPS
HTTPS is a HTTP (hypertext transfer protocol) secure - http with SSL/TLS support
##### SSL/TLS
Secure socket layer and transport layer security. TLS uses assymetric encryption only while setting connection: it allows to choose a key for symmetric encryption for the whole session.

### DBMS
##### Indexes
What's that? What for? Normal/Unique/Composite/Partial etc.? Partial - to optimize index (for example by excluding popular values - reduces size and, improve select insert and update speed).
##### Deadlcok
Deadlock example:

|Transaction1|Transaction2|
|---------------|---------------|
|Update t1 query|...|
|...|Update t2 query|
|Update t2 query|...|
|blocked|Update t1 query|
|blocked|blocked|

##### Replication and Sharding

### Golang
##### Why go?
goroutines, network poller  
##### Declarative vs imperative
golang is imperative - tells HOW to achieve a goal  
sql is declarative - declares WHAT is to be achieved  
##### Serialization
Converting variable to a binnary representation (pointers might be a problem)  
##### ORM
object-relational mapping  
##### Strings
String is an immutable slice of bytes  
Concatenating strings might be costly, use string builder to make it effective:  
```go
s := strings.Builder{}
for _, w := range words {
    s.WriteString(w)
}
s.String()
```
##### Map
```go
m := make(map[string]int)
```
panic on wtiting to not inited maps  
fatal error: concurrent map writes  
fatal error: concurrent map iteration and map write  
use sync.Mutex or use sync.Map  
random order when iterating over it  
##### OOP
struct, visibility, embedding, cunstructor, methods, receivers by link/ by value, interfaces  
don't use pointer receivers for map, chan, func  
don't use pointer receivers for slices if no reallocation/reslicing  
don't use value receivers for structs with Mutex  
inheritance is not embedding (L in SOLID: successor cannot be used instead of base class)
##### Goroutines
goroutine is not a OS thread, it's smaller. Limit threads: GOMAXPROCS, limit goroutines: use workers pool pattern
##### Slice
pointer to an underlying array. Length, capacity. Realloctaion on capacity exceeding. Slice may change even when was passed by value (since it's a pointer) but only if it was not reallocated. Use pointer on a slice if you plan to change it
##### Channel
```go
m := make(ch chan int)
```
```go 
func f(c chan<- string) {}
```
buffered channels  
panic on writing to closed or nil channels  
reading from closed channel returns default value, false  
##### Sync
Mutex, RWMutex  
sync.Map  
atomic  
##### Interfaces
Type that satisfies an interface is said to implement it  
Interface default value is nil  
Empty interface is any type  
```go
switch val.(type){
case int:
default:
}
```
type assertion
```go
func assertInt(val interface{}) {
    i, ok := val.(int) // if false without ok - panics
}
```
interface assertion
```go
var _ Interface = (*Type)(nil)
```
##### Memory
go runtime lies in OS heap  
go works in stack, but if a function returns a pointer, then go creates it in heap  
Read(p []byte) (n int, err error) - reduces amount of allocations  
Arena (64MB) -> Span (66 classes) -> Pages  
STW - > mark heap objects (tricolor) -> STW. Actual GC during allocation  
when: by time (minutes), by mem limit (GOGC), by call runtime.GC()  
##### Scheduler
Pool of goroutines -> local pools (set of g per thread) -> global pool -> work stealing (half)  
Queue: FIFO + 1-el LIFO  
FIFO hunger: timer for goroutines chain (~10ms)  
goroutines hunger: timer for long runs (sysmon) + park g on syscall  
global queue hunger: every 61 tick local queue takes a g from global queue  
network poller - a pad between OS network management and goroutines  


```go
import (
    "fmt"
    "time"
    "sort"
    "math/rand"
    "sync"
)
```


```go
func sleep(n int) {
    <-time.After(time.Duration(n) * time.Second)
}
```


```go
var a, b = 1, 4
func swap(x, y *int) {
    *x, *y = *y, *x
}
fmt.Printf("before swap a: %d, b: %d\n", a, b)
swap(&a,&b)
fmt.Printf("after swap a: %d, b: %d\n", a, b)
```

    before swap a: 1, b: 4
    after swap a: 4, b: 1





    22 <nil>




```go
type Something struct {
    Name string
}
s := []Something{{"aba"}, {"george"}, {"bob"}}
sort.Slice(s, func(i,j int) bool {
    return s[i].Name < s[j].Name
})
for _, elem := range s {
    fmt.Printf("%s\n", elem.Name)
}
```

    aba
    bob
    george



```go
func sliceIntersectionMemory(a, b []int) []int {
    am := make(map[int]struct{})
    l := len(a)
    if len(a) < len(b) {
        l = len(b)
    }
    c := make([]int, 0, l)
    for _, elem := range a {
        am[elem] = struct{}{}
    }
    for _, elem := range b {
        if _, ok := am[elem]; ok {
            c = append(c, elem)
        }
    }
    return c
}

func sliceIntersectionTime(a, b []int) []int {
    l := len(a)
    if len(a) < len(b) {
        l = len(b)
    }
    c := make([]int, 0, l)
    sort.Slice(a, func(i, j int) bool { return a[i] < a[j] })
    sort.Slice(b, func(i, j int) bool { return b[i] < b[j] })
    j := 0
    INTERSECTION:
    for _, el := range a {
        for {
            if j == len(b) {
                break INTERSECTION
            }
            if el == b[j] {
                c = append(c, el)
                j++
                break
            }
            j++
        }
    }
    return c
}

a := []int{1,5,4,3,7,9}
b := []int{1,2,3,4,5,10}
for _, el := range sliceIntersectionTime(a,b) {
    fmt.Printf("%d, ", el)
}
fmt.Println()
for _, el := range sliceIntersectionMemory(a,b) {
    fmt.Printf("%d, ", el)
}
```

    1, 3, 4, 5, 
    1, 3, 4, 5, 


```go
func randIntGenerator(n, m int) <-chan int {
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    out := make(chan int)
    go func() {
        for i := 0; i < n; i++ {
            out <- r.Intn(m)
        }
        close(out)
    }()
    return out
}

for i := range randIntGenerator(20, 1000) {
    fmt.Printf("%d, ", i)
}
```

    614, 331, 935, 487, 464, 299, 324, 952, 463, 447, 469, 180, 210, 124, 725, 966, 271, 373, 740, 553, 


```go
func mergeChannels(chans ...<-chan int) <-chan int {
    merged := make(chan int)
    go func() {
        wg := sync.WaitGroup{}
        for _, ch := range chans {
            wg.Add(1)
            go func(ch <-chan int) {
                defer wg.Done()
                for i := range ch {
                    merged<- i
                }
            }(ch)
        }
        wg.Wait()
        close(merged)
    }()
    return merged
}

a := make(chan int)
b := make(chan int)
go func() {
    for _, i := range []int{1,2,3} {
        a <- i
    }
    close(a)
}()
go func() {
    for _, i := range []int{4,5,6} {
        b <- i
    }
    close(b)
}()
ch := mergeChannels(a, b)
for i := range ch {
    fmt.Printf("%d, ", i)
}
```

    4, 5, 6, 


```go
func pipeline(in <-chan int, out chan<- int) {
    for input := range in {
        out<- input*2
    }
    close(out)
}
in := make(chan int)
out := make(chan int)
go func() {
    for _, i := range []int{1,2,3,4,5,6,7,8,9} {
        in<- i
    }
    close(in)
}()
go pipeline(in, out)
for i := range out {
    fmt.Printf("%d, ", i)
}
```

    2, 4, 6, 8, 10, 12, 14, 16, 18, 


```go
func worker(id int, fn func(int) int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        result := fn(job)
        results <- result
        fmt.Printf("worker %d has done job %d with result %d\n", id, job, result)
    }
}

workersLimit := 3
jobs := make(chan int)
results := make(chan int)
fn := func(i int) int { return i * 3 }
go func() {
    wg := sync.WaitGroup{}
    for i := 0; i < workersLimit; i++ {
        i := i
        wg.Add(1)
        go func() {
            defer wg.Done()
            worker(i, fn, jobs, results)
        }()
    }
    wg.Wait()
    close(results)
}()
go func() {
    for _, i := range []int{1, 2, 3, 4, 5, 6, 7, 8, 9} {
        jobs <- i
    }
    close(jobs)
}()
for i := range results {
    fmt.Printf("%d\n", i)
}
```

    worker 1 has done job 3 with result 9
    9
    3
    6
    12
    worker 1 has done job 4 with result 12
    worker 0 has done job 1 with result 3
    15
    worker 1 has done job 5 with result 15
    worker 2 has done job 2 with result 6
    worker 1 has done job 7 with result 21
    21
    24
    18
    27
    worker 2 has done job 8 with result 24
    worker 1 has done job 9 with result 27
    worker 0 has done job 6 with result 18



```go
type WG struct {
    sema chan struct{}
    len  int
    mx sync.Mutex
}

func NewWG() *WG {
    return &WG{sema: make(chan struct{})}
}

func (wg *WG) Add(i int) {
    wg.mx.Lock()
    defer wg.mx.Unlock()
    wg.len += i
}

func (wg *WG) Done() {
    wg.mx.Lock()
    defer wg.mx.Unlock()
    wg.len--
    if wg.len == 0 {
        close(wg.sema)
    }
}

func (wg *WG) Wait() {
    <-wg.sema
}

wg := NewWG()

for i:=0; i < 20; i++ {
    wg.Add(1)
    go func(i int){
        defer wg.Done()
        fmt.Printf("%d, ", i)
    }(i)
}

wg.Wait()
```

    1, 9, 2, 7, 0, 10, 11, 12, 13, 14, 15, 16, 4, 17, 18, 8, 5, 6, 3, 19, 


```go
import (
    "fmt"
    "strconv"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    ch := make(chan string)
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            ch <- fmt.Sprintf("Goroutine %s", strconv.Itoa(i))
        }(i)
    }
    go func() {
        wg.Wait()
        close(ch)
    }()


    {
    FOR:
        for {
            select {
            case q, ok := <-ch:
                if !ok {
                    break FOR
                }
                fmt.Println(q)
            }
        }
    }
}
```
