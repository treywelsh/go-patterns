# Bounded Parallelism Pattern

[Bounded parallelism](https://blog.golang.org/pipelines#TOC_9.) is similar to [parallelism](parallelism.md), but allows limits to be placed on allocation.

```go
    func doSomething(in <-chan string, out chan<- result) {
	    for item := range in {
            doProcessing(item)
        }
    }

    func boundedParallelism() {
        var wg sync.WaitGroup
	    in := make(<-chan inT)
        out := make(chan<- result)

        addWork(in)

        const cnt = 20
        wg.Add(cnt)
        for i := 0; i < cnt; i++ {
            go func() {
                doSomething(in, out)
                wg.Done()
            }()
        }
        go func() {
            wg.Wait()
            close(out)
        }()

        for res := range out {
            ...
        }

    }

```

# Implementation and Example

An example showing implementation and usage can be found in [bounded_parallelism.go](bounded_parallelism.go).
