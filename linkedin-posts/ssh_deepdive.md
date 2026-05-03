I've SSHed into EC2 instances hundreds of times.

ssh -i geek-ec2-kp.pem ec2-user@54.226.167.59

It works. I get a shell. I move on.

But I never paid attention to that fingerprint prompt:

```
The authenticity of host can't be established.
ED25519 key fingerprint is SHA256:WPHp1jPiPTm...
Are you sure you want to continue connecting (yes/no)?
```

I'd type yes without thinking.

Then this "Building Trust on the Internet" series got me curious.

I already had a key — that's what the .pem file was for.
So what was SSH asking me to trust?

Turns out, SSH uses two completely separate keypairs.

The .pem file I downloaded? That's my user authentication keypair.
It proves I'm authorized to log in.

The fingerprint SSH was showing me? That's the server's host keypair.
It proves the server is who it claims to be.

Two different keys. Two different trust questions.

SSH verifies the server's identity before I ever prove who I am.
Secures the pipe first, then checks authorization.

I ran experiments to see what triggers warnings:

• Reboot: same fingerprint, silent login
• Stop/start: new IP, same fingerprint
• Terminate/launch: different fingerprint
• Elastic IP reuse: same IP, different instance → MITM WARNING

That last one is what SSH protects against.

An attacker can steal my IP.
But they can't generate the same host keypair.

known_hosts isn't "servers I trust."
It's "this server has this public key."

SSH tracks by cryptographic identity, not hostname.
If the key changes, SSH refuses to connect.

This is the opposite of TLS.

TLS trusts a hierarchy — DigiCert, Let's Encrypt, 100+ root CAs.
SSH has no global authority. You are the trust anchor.

First connection: you verify the fingerprint (or blindly trust).
After that: SSH enforces consistency.

This is Part 3 of "Building Trust on the Internet."

Read here:
🔗 Building Trust on the Internet — Part 3: SSH's Two-Keypair Authentication

https://lnkd.in/gN9VBgvV: Building Trust on the Internet — Part 3: SSH's Two-Keypair Authentication
