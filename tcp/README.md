# Learning notes

Code taken from Andrew Gerrand's talk [Go: code that grows with grace](https://talks.golang.org/2012/chat.slide#1)

All example servers can be started with
```bash
$ go run <filename.go>
```
and clients can connect via netcat
```bash
nc localhost 4000
```

1. `hellonet.go`
a. uses the `net` package to create a server/TCPListener with `l, err := net.Listen("tcp", listenAddr)`
b. TCPListener waits for and returns the next connection (`c, err := l.Accept()`)
c. a greeting is written to the connection `fmt.Fprintln(c, "Hello!")`

2. `echoserver.go`
a. same as before, with the difference that instead of a hard-coded greeting, the connection input will be copied as output (hence echo)
b. `io.Copy` copies from src to dst until either EOF is reached on src or an error occurs - meaning it blocks for all other clients connecting to the server until the first connected client closes the connection!

3. `echoserver_concur.go`
a. calling `io.Copy` as a goroutine makes the server concurrent

4. `chat.go`
a. instead of echoing client inputs, client are now able to exchange inputs: `io.Copy` is replaced with a function named `match`
b. `match` uses the `select` statement: If a client connects, and there is no partner client waiting already in the `partner` channel, the client connection itself will be added to the channel (`case partner <- c`). If there already is a partner client waiting in the channel (`case p := <- partner`), the chat is started.
c. the chat function itself copies data from each connection to the other - one copy operation is launched in another goroutine so the two copy operations can happen concurrently. The input parameters `a` and `b` are connections (type `Conn`) and as such implement the `io.ReadWriteCloser` interface that contains Read, Write and Close methods.
d. the issue is that if one client closes the connection, the other client does not get notified!

5. `chat_errhndl.go`
a. when an error happens during the copy operation for any of the two connections, it is added to a channel
b. the line `if err := <-errc; err != nil` in the `chat` function blocks until an err appears, after that both connections are closed
c. interestingly, when a client terminates, the error is nil - so this case doesn't get logged by the line `log.Println(err)`