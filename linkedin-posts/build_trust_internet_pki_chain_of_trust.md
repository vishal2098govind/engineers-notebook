Trust on the internet is a strange thing.

Two systems, sitting somewhere across the world, start talking to each other…
without ever having met,
over a network that anyone can listen to.

And yet, they trust.

For a long time, HTTPS just felt like:

🔒 this connection is secure

But that never answered the deeper question:

Where does that trust actually come from?

When I started digging into it, things shifted.

It wasn’t just encryption.
It was a system of:
• proving identity without revealing secrets
• verifying information without prior trust
• building confidence through layers, not assumptions

Concepts that once felt separate started to connect:

• hashing → integrity
• digital signatures → authenticity
• certificates → identity
• PKI → trust distribution
• TLS → trust in motion

At some point, it stopped feeling like “security mechanisms”
and started feeling like a "design for establishing trust in an untrusted world".

I tried to capture that journey, from first principles to HTTPS, in this piece.

This is also where I’m starting a deeper exploration into:

• how identity is represented (JWT)
• how trust is distributed at scale (JWKS)
• how access is delegated (OAuth)
• how decisions are made (RBAC, ABAC, policies)
• and how systems like AWS IAM bring it all together

If you’ve ever paused at that 🔒 icon and wondered what it really stands for,
this might resonate.

Read here:

Building Trust on the Internet — Part 1: From First Principles to HTTPS