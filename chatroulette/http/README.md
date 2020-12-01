# Learning notes

Code taken from Andrew Gerrand's talk [Go: code that grows with grace](https://talks.golang.org/2012/chat.slide#1)

Install the `golang.org/x/net/websocket` package
```bash
$ go get golang.org/x/net/websocket
```
Start the server
```bash
$ go run chat_final.go
```

`chat_final.go`
  * uses http to serve UI and websocket for chat
  * associates the `/` endpoint with a handler function
  * starts a server and blocks until an error happens (`err := http.ListenAndServe(listenAddr, nil)`)
  * the websocket connection shuts down when the handler returns, but we want to keep the connection open
  * therefore we have to keep the socket handler running until it is closed by wrapping the connection (which implements the `io.ReadWriter` interface) into a `socket` type
  * run the same `match` function as in the tcp example in a goroutine
  * the `socketHandler` is blocked until the `done` channel receives something
```go
type socket struct {
    io.ReadWriter
    done chan bool
}

func (s socket) Close() error {
    s.done <- true
    return nil
}

func socketHandler(ws *websocket.Conn) {
    s := socket{conn: ws, done: make(chan bool)}
    go match(s)
    <-s.done
}
```
 * the client can send and receive data from the server like this
```javascript
var sock = new WebSocket("ws://localhost:4000/");
sock.onmessage = function(m) { console.log("Received:", m.data); }
sock.send("Hello!\n")
```
