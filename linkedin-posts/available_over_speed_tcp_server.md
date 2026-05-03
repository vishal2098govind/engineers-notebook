I used to think Go concurrency was about making things faster.

Then I wrote a tiny TCP server.

What broke wasn’t performance — it was 𝗮𝘃𝗮𝗶𝗹𝗮𝗯𝗶𝗹𝗶𝘁𝘆.

A single blocking `Accept()` call meant:
• one client connected
• everyone else waited

Adding a goroutine didn’t make the server faster. 
It made the server 𝘂𝘀𝗮𝗯𝗹𝗲.

That’s when concurrency stopped feeling like a language feature and started feeling like a design decision.

I wrote about this realization while evolving a raw TCP server in Go:

https://lnkd.in/g7uC7fdZ