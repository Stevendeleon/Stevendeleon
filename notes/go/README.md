# Go (Golang) Notes


### Templates
* These are wonderful, they work with all sorts of files and not limited to html
    * Has helped me a ton with automation work

### Testing
* Race detector is a helpful way for identifying potential concurrency issues within your program
    * can be used as: 
        ```shell
        $ go test -race mypkg    // to test the package
        $ go run -race mysrc.go  // to run the source file
        $ go build -race mycmd   // to build the command
        $ go install -race mypkg // to install the package
        ```
    * In the past, I've built some concurrent functionality, ran some unit + integration tests along with some benchmarks and everything checked out well and passing. Issues went unnoticed until I re-ran using the `--race` flag. Very useful!
    * Resources:
        * [race detector](https://go.dev/doc/articles/race_detector)

### Godoc
* This is a nice tool for auto generating documentation for your packages based on comments and naming conventions.
* I do not see this used often enough for codebases I have worked on but something I try to sprinkle in if teams are up to it.
* Resources:
    * [godoc](https://pkg.go.dev/golang.org/x/tools/cmd/godoc)
    * [Godocs - Effortless documentation for your go packages](https://www.youtube.com/watch?v=80VT3xexcWs) - (YT Video)

### Scraping
* net/http and [goquery](github.com/PuerkitoBio/goquery) is a combo I like.
* if you need to maintain a session between redirects, [net/http/cookiejar](https://pkg.go.dev/net/http/cookiejar) works wonders

### Channels
* Great for transferring data between goroutines
* by default, channels have no space inside and so you cannot temporarily store a value in it, it acts more as a portal.
* **when unbuffered**, you need to simultaneously have **1** thread to send data to channel and another to receive:
    ```go
    // unbuffered
    numbersChannel := make(chan int)
    go func() {
        numbersChannel <- 555
    }()
    n := <-numbersChannel
    fmt.Printf("n = %d\n", n)
    ```
* **when buffered**, you can send/receive under a single thread:
    ```go
    numbersChannel := make(chan int, 1)
    numbersChannel <- 555
    n := <-numbersChannel
    fmt.Printf("n = %d\n", n)
    ```

### Mutexes
* Otherwise known as mutual exclusions
* Helpful when you do not need to worry about communication between goroutines
* also for when you want to **safely** write to a single spot in memory (ensuring the space in memory is only written to by one thing)
    * locking, doing the write, then unlocking
    * Simple counter example:
    ```go
    type counter struct {
        numberMap map[string]int
        mutex     sync.Mutex
    }

    func (c *counter) add(key string, num int) {
        c.mutex.Lock()
        defer c.mutex.Unlock()
        c.numberMap[key] = num
    }

    var wg sync.WaitGroup

    func main() {
        c := counter{
            numberMap: make(map[string]int),
        }

        for i := 0; i < 1000; i++ {
            wg.Add(1)
            go func(i int) {
                defer wg.Done()
                key := fmt.Sprintf("key%d", i)
                c.add(key, i)
            }(i)
        }
        wg.Wait()

        for key, num := range c.numberMap {
            fmt.Printf("Key: %s, Value: %d\n", key, num)
        }
    }
    ```