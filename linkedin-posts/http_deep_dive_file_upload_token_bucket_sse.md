When an 8GB file uploads to a server, where does the data actually sit?

I built a file upload service to trace the data's journey and answer three questions:

𝟭. 𝗪𝗵𝗲𝗿𝗲 𝗱𝗼𝗲𝘀 𝘂𝗽𝗹𝗼𝗮𝗱 𝗱𝗮𝘁𝗮 𝗮𝗰𝘁𝘂𝗮𝗹𝗹𝘆 𝗹𝗶𝘃𝗲?

An 8GB file moves from browser to disk.
"In memory" is too vague.

Where in RAM? How much? For how long?

Turns out: ~160KB total.
 • TCP socket buffer: ~128KB (kernel)
 • io.Copy buffer: 32KB (user-space, reused)

The rest is either on the wire, on disk, or in the OS page cache.

Go's io.Copy allocates a 32KB buffer once and reuses it in a loop.
The buffer size doesn't grow with file size.

Streaming doesn't mean "read faster."
It means "never accumulate."

𝟮. 𝗛𝗼𝘄 𝗱𝗼𝗲𝘀 𝘁𝗵𝗲 𝗰𝗹𝗶𝗲𝗻𝘁 𝗸𝗻𝗼𝘄 𝗵𝗼𝘄 𝗺𝗮𝗻𝘆 𝗯𝘆𝘁𝗲𝘀 𝗴𝗼𝘁 𝘄𝗿𝗶𝘁𝘁𝗲𝗻 𝘁𝗼 𝗱𝗶𝘀𝗸?

The upload handler is writing to disk.
The client is sitting somewhere else, waiting.

How does progress reach the client in real-time?

That's where Server-Sent Events came in.

Turns out SSE is just HTTP:
 • Content-Type: text/event-stream
 • Transfer-Encoding: chunked
 • Flush() after every event

No websocket upgrade. No special protocol.
Just a long-lived GET that writes strings and flushes immediately.

The key: two separate HTTP connections.
HTTP/1.1 is half-duplex.
One connection uploads the file.
The other streams progress events.

A wrapped writer intercepts every write to disk, updates a progress store.
The SSE handler polls that store every 200ms, streams events back.

𝟯. 𝗛𝗼𝘄 𝗱𝗼 𝘆𝗼𝘂 𝘁𝗵𝗿𝗼𝘁𝘁𝗹𝗲 𝗱𝗮𝘁𝗮 𝗳𝗹𝗼𝘄 𝘁𝗵𝗿𝗼𝘂𝗴𝗵 𝗶𝗼.𝗖𝗼𝗽𝘆?

The upload finished too fast to observe — under a millisecond for an 838-byte file.

To actually see progress events stream back, I built a token bucket rate limiter.

Not because the service needed throttling.
Because understanding how to control data flow by wrapping the io.Reader turned out to be the clearest way to understand streaming itself.

Each case handles a different flow control scenario.

Full write-up: https://lnkd.in/gPTfqeKf: HTTP Deep Dive - HTTP Deep Dive - HTTP File Upload: Memory Flow, SSE Progress, and Token Bucket Rate Limiting