```
(JavaScript进行WebSocket字节流通讯示例)[https://www.cnblogs.com/coloc/p/8127268.html]

Read RFC 6455, especially 5. Data Framing, and you can find answers there.

In the WebSocket Protocol, data is transmitted using a sequence of frames (and not base64-encoded). Frame types defined in RFC 6455 are as follows.

Continuation Frame (opcode = 0x0)
Text Frame (opcode = 0x1)
Binary Frame (opcode = 0x2)
Close Frame (opcode = 0x8)
Ping Frame (opcode = 0x9)
Pong Frame (opcode = 0xA)
Decent WebSocket libraries (both client-side ones and server-side ones) can handle all of these frames and provide functions/methods to send/receive them. So, you don't have to worry about how data are encoded regardless of whether data are text or binary (unless you choice a bad WebSocket library).

Note that both clients and servers can send/receive both text frames and binary frames.
```
