Fan-Out Messaging Pattern
=========================
Fan-Out is a messaging pattern used for distributing work amongst workers (producer: source, consumers: destination).

We can model fan-out using the Go channels.

## Implementation

```go
// Split a channel into n channels that receive messages in a round-robin fashion.
func fanOut(in <-chan time.Time, buflen int, count int) []<-chan time.Time {
	out := make([]chan time.Time, count)
	outRead := make([]<-chan time.Time, count) //Can't return out as []<-chan time.Time

	for i := range out {
		out[i] = make(chan time.Time, buflen)
		outRead[i] = out[i]
	}

	spread := func(in <-chan time.Time) {
		for v := range in {
			for _, ch := range out {
				ch <- v
			}
		}
        
        //When in closed, all output chans are closed
		for _, ch := range out {
			close(ch)
		}
	}

	go spread(in)

	return outRead
}
```
## Usage

```go
func main() {
	timer := time.NewTicker(2 * time.Millisecond)

	chans := fanOut(timer.C, 5, 10)
	for i, c := range chans {
		go func(i int, c <-chan time.Time) {
			for v := range c {
				fmt.Println("ch", i, "time", v)
			}
		}(i, c)
	}
	fmt.Println("Wait and close")
	time.Sleep(2 * time.Second)
	timer.Stop()
}
```

The `Split` function converts a single channel into a list of channels by using 
a goroutine to copy received values to channels in the list in a round-robin fashion.
