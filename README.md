# WebSocket Experiment
An attempt to implement part of the RFC webscoket server spec in Go.

## Summary
- Receive an http `GET` request when the js `WebSocket` instance is created
- Get the `Sec-WebSocket-Key` to do the upgrade handshake as follows
    - concat the value with the magic string "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"
    - hash the result encode to Base64
    - write to response header `Sec-WebSocket-Accept`
    - set http status to `101 Switching Protocols`
- after the handshake hijack the tcp socket to write and read directly using it
- websocket data is represented in "frames" with encoded meta data in taking up to 10 bytes from in server-client frames and up to 14 bytes in client-server frames due to XOR masking for security
- which leads to having to decode the client payload by XORing mask bytes using the following op `dec[i] = enc[i]  ^ mask[i%4]`

## My App
> The app doesn't use db and everything happens in memory so the data model isn't well thought out.

A small canvas frontend that generates few shapes and has camera dragging, I generate a room id on the client if the user visits `/` and store the room id once the WS connection happens and map the user to it, if an exisitng room is visited I just store the user id and map it to the existing room, I synchronize camera dragging between users in the same room by broadcasting the update event so the write goroutines of connected clients can check and send the new state, this of course doesn't account for multiple users dragging at once as there's no conflic resolution but I assume this will be a case of last-write-wins.

**the read and write loops happen in two seperate goroutines fired after the handshake**

### Run it
```sh
npm run build
go run .
```