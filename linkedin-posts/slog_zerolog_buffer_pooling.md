I thought logging was one line of code.

log.Info("request ended")
Done.

I'd seen memory spiking in CloudWatch when traffic hit 1,000+ requests/min.
Understanding how logging works under concurrency made allocation pressure visible.

𝗧𝗵𝗲 𝗽𝗿𝗼𝗯𝗹𝗲𝗺:
Two goroutines writing to stdout simultaneously:
{"level":"INFO","msg":"Request en{"level":"ERROR","msg":"upload failed"}ded","duration":"8.2s"}

Not valid JSON. Log parsers break. Grafana shows garbage.

𝗧𝗵𝗲 𝗳𝗶𝘅 (slog's approach) :
Mutex around the write.
But the mutex covers format + write. Under 1000 concurrent goroutines, 999 wait while 1 formats.
The expensive part, assembling the JSON string, is serialized even though it could happen in parallel.

𝘇𝗲𝗿𝗼𝗹𝗼𝗴'𝘀 𝗮𝗽𝗽𝗿𝗼𝗮𝗰𝗵:
 • Format outside the mutex.
 • Each goroutine gets its own scratch buffer from sync.Pool.
 • Format in parallel (no lock)
 • Lock only for the write syscall

Mutex window shrinks from milliseconds to microseconds.

𝗧𝗵𝗲 𝗯𝗶𝗴𝗴𝗲𝗿 𝗿𝗲𝗮𝗹𝗶𝘇𝗮𝘁𝗶𝗼𝗻:
Understanding buffer pooling made serialization cost visible everywhere.

Every microservice call pays serialization cost twice (marshal + unmarshal).

Teams have measured 30-40% of request latency in some architectures being pure serialization overhead.

Not business logic. Not database queries. Just converting data between bytes and structs.

𝗪𝗵𝗮𝘁 𝘁𝗵𝗲 𝗮𝗿𝘁𝗶𝗰𝗹𝗲 𝗰𝗼𝘃𝗲𝗿𝘀:
• Concurrent writes and mutex contention 
• Buffer pooling and sync.Pool mechanics (per-P slots, victim cache, lock-free design) 
• Why the pattern appears in net/http, encoding/json, database drivers 
• The serialization tax on every microservice boundary 
• Why "let's add an API" is a serialization decision

Read here: https://lnkd.in/gvjNk_er: I Thought Logging Was One Line Until I Saw the Cost of Serialization Through Buffer Pooling