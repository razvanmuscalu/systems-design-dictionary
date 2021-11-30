# Golang

- [Basics](#basics)
  - [Array vs Slice](#array-vs-slice)
  - [Pointers vs Copies](#pointers-vs-copies)
- [Concurrency](#concurrency)
  - [Routines](#routines)
  - [Stop Routines](#stop-routines)
  - [Channels](#channels)
    - [Buffered vs Unbuffered](#buffered-vs-unbuffered)
  - [Advanced Concurrency](#advanced-concurrency)
  - [Context](#context)

# Basics

* What's the difference between `var foo []int` and `foo := []int{}`
  * First does not initialise and takes no space in memory
  * Second initializes with zero value and occupies space in memory

## Array vs Slice

* Arrays have fixe sizes
  * And in a contiguous place in memory => therefore more efficient
  * Do use arrays when the size you’re working with is fixed (or you don’t mind having null elements at random locations)
* Slices are windows of arrays and have dynamic sizes
  * They do not hold data, but instead change data on underlying array
  * They are a pointer to an array where the data starts
  * Have length and capacity (so that append can grow the size dynamically)
  

## Pointers vs Copies

  - Copies are pass-by-value
    - They work on copies of data, so methods attached to literals will not change the underlying data
    - They live on the stack
    - Use them for small objects (e.g. primitives)
  -  Pointers are pass-by-reference
    - Methods attached to pointers will change the data at the referenced address
    - They live on the heap
    - Use them for large objects
    - Do NOT return pointers only to be able to return nil
  - See: https://gobyexample.com/pointers
  

# Concurrency

## Routines

* A lightweight thread, first-class citizen of the language
* Executes concurrently (time-slicing) on separate stack, but shared heap
* Everything managed and scaled by go runtime (as opposed to Java where pool executors must be created and configured)
  

## Stop routines

* From sender side, via “done” channel
  * Sender and Receiver communicate via one channel to send actual data across, and another channel to send a “done” signal
  * Receiver listens to both channels (via select) and it can stop when it receives the “done” signal (simply “return”)
* From receiver side only, via timeout channel
  ```
  select {
      case res := <-c1:
          fmt.Println(res)
      case <-time.After(1 * time.Second):
          fmt.Println("timeout 1")
  }
  ```
  

## Channels

* Channels are used to send data between go routines
* By default, they’re blocking (a sender will be blocked if the sender is not ready to receive)
  * The blocking nature can avoided by using a default case in the “select” statement
    

### Buffered vs Unbuffered

* The blocking nature can also be avoided via buffered channels
* A sender can send to a buffered channel even if receiver is not ready
* The channel will keep the data until receivers can receive, or until buffer gets full (at this time, senders will start blocking)
  

## Advanced Concurrency

- Worker pools via separate channels for receiving jobs and sending results (https://gobyexample.com/worker-pools)


- To wait until multiple routines finished executing, use wait groups (https://gobyexample.com/waitgroups)
  - Add routine to wait group (`wg.Add(1)`)
  - Signal when routine is done by deferring `wg.Done()`
  - Wait on main routine (`wg.Wait()`)


- To propagate errors from routines, use err groups


- Rate limiting via time tickers (https://gobyexample.com/rate-limiting)


- Can control safety of shared heap via atomic counters (https://gobyexample.com/atomic-counters) or mutexes (https://gobyexample.com/mutexes)


## Context

* How would you use it? Why would you use it?
  * Context carries deadlines/timeouts that can be accessed via `ctx.Done()`
    * `ctx.Err()` returns an error explaining why the Done channel was closed
    * Set timeout when spawning sub-contexts from a parent context (e.g. when making an HTTP call, when calling a database, etc.)
      * This is done via `.WithTimeout()` function
  * Context also carries user-defined request-scoped values (e.g. user id, etc.)
  * See: https://gobyexample.com/context
  * See: https://go.dev/blog/context