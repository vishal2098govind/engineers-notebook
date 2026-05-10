I thought connection pooling was just caching connections to avoid the handshake cost.



Then I built one from scratch.



Pooling isn't about caching.

It's about two separate contracts:



𝟭. 𝗔 𝗳𝗿𝗮𝗺𝗶𝗻𝗴 𝗰𝗼𝗻𝘁𝗿𝗮𝗰𝘁



How does the client know when a response ends?



Return a connection to the pool mid-response and the next caller gets a connection with leftover bytes in the recv buffer.

Corrupted data.



Every system solves this differently:

• Postgres: sends ReadyForQuery frame after every response

• HTTP chunked: 0\r\n terminating chunk

• HTTP with Content-Length: client reads exactly N bytes



Without the sentinel, the pool can't know when a connection is safe to reuse.



𝟮. 𝗔 𝘀𝘆𝗻𝗰𝗵𝗿𝗼𝗻𝗶𝘇𝗮𝘁𝗶𝗼𝗻 𝗰𝗼𝗻𝘁𝗿𝗮𝗰𝘁



How many connections should exist?



Three knobs control this:

• MaxConnsPerHost (semaphore) — caps total connections per server, absorbs bursts

• MaxIdleConnsPerHost — caps stale idle accumulation

• MaxIdleConns (global) — prevents fd exhaustion when talking to 50 hosts



The semaphore is pre-filled with tokens.

Dial a connection → consume token.

Return to pool → no change (connection still exists).

Close connection → release token.



When MaxConnsPerHost is exhausted, new requests park on the semaphore.

As in-flight connections complete and close, parked goroutines unblock in waves.



𝗪𝗵𝗮𝘁 𝘁𝗵𝗲 𝗮𝗿𝘁𝗶𝗰𝗹𝗲 𝗰𝗼𝘃𝗲𝗿𝘀:



• Building a raw TCP pool from buffered channels to shared-host pools

• Why pooling starts with a server decision (the for sc.Scan() loop)

• IdleConnTimeout and eviction goroutines

• The semaphore invariant that makes shared pools work

• How http.Transport maps to the same knobs

• SQLAlchemy pool tuning (same pattern, different language)

• HTTP/1.1 head-of-line blocking and why HTTP/2 fixes it



The big realization:



Framing and synchronization aren't separate problems.

They're two halves of the same pooling contract.



Read here: https://open.substack.com/pub/vishalageek/p/i-thought-connection-pooling-was?r=n6j9g&utm_campaign=post&utm_medium=web&showWelcomeOnShare=true



Part of my HTTP Protocol Deep Dives series.