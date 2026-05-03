For a long time, I attached IAM roles to Lambda functions without really thinking about what happened next.

Role attached. Works. Move on.

But at work, I ran into a situation where a Step Function in one AWS account needed to trigger a Lambda in a completely different account.

And suddenly "attach a role" wasn't enough of an answer.

I had to actually understand what was happening underneath.

Turns out, every time a Lambda hits S3, or any AWS service talks to another:

• No password is configured
• No key is hardcoded
• No secret is shared between the two services

Yet the request is authenticated, authorized, and completed.

The mechanism behind that is surprisingly elegant.

It starts with a role — not tied to a person, tied to a purpose.
A trust policy defines who can assume it.
A permission policy defines what they can do.

When the Lambda is invoked, AWS calls STS before the handler even starts.
STS returns short-lived credentials — valid for an hour, useless after.
The SDK picks them up silently, signs every request with HMAC, and sends it.
The receiving service recomputes the signature independently to verify.

Nothing static. Nothing permanent. Nothing worth stealing.

The architecture is designed so there is nothing static to steal.

This is Part 2 of my "Building Trust on the Internet" series.
Part 1 was about TLS — how a browser trusts a server it has never met.
This one is about what happens inside AWS once that channel is established.

Covered in the article:
• IAM Roles, trust policies, permission policies
• The full STS flow — from role attachment to signed API call
• What SigV4 actually signs and why it uses HMAC, not asymmetric crypto
• EC2 IMDS and IMDSv2 — and the SSRF gap it closed
• Cross-account access — the same STS story, one level up

Read here: Building Trust on the Internet — Part 2: AWS IAM and AWS STS
