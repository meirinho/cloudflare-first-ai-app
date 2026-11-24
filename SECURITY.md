# Security

This app is small on purpose, but it follows the same principles as a production AI app. This document explains the principles, not the parameters. Real values and configuration details stay out of the repository.

## Principles

1. **Configuration outside the code.** Limits, keys, prompt and allowed address come from Worker variables and secrets. The repository only has examples.
2. **Fail closed.** If configuration is missing, bot verification fails or a counter cannot be updated, the app refuses the question. It never starts running without limits.
3. **Layered usage limits.** There is a short-window limit per person, a weekly limit per person and a daily global cap well below the free allocation, so there is room left for other apps. The layers overlap, so one failing does not leave the app open.
4. **Bot verification.** Every question requires a Cloudflare Turnstile token, validated on the server before anything is spent. Each token works only once.
5. **Same origin.** The API only accepts requests from its own page, with a small JSON body in the expected shape.
6. **Prompt injection protection.**
   - The system prompt lives on the server, together with fixed rules that no user text can change.
   - The question goes in a separate, delimited and size-limited field, and is treated as data.
   - History does not come from the browser: each question is evaluated on its own.
7. **Output always as text.** The model's answer is inserted into the page as plain text, never as HTML. The page has a strict nonce-based Content Security Policy, blocks being framed and sends security headers.
8. **No stored data.** The app does not log questions or answers. Counters store only numbers, under keys derived with a secret salt, and expire on their own.
9. **Educational, generic messages.** Errors remind people this is an educational demo app and say what they can do, without revealing numbers or how a rule works.
10. **Emergency switch.** A variable turns the app off immediately, without touching the code.
11. **Zero cost.** Everything runs on the free plan. If a free allocation runs out, the service stops instead of generating charges.

## Known limitations

- Distributed counters are approximate by nature. That is why there are several layers and a global cap with headroom.
- No defense against prompt injection is perfect. The rules reduce the risk, and rendering output as text keeps a manipulated answer from becoming code on the page.

## Reporting a problem

Found a vulnerability? Please do not open a public issue. Use GitHub's private vulnerability reporting in this repository (Security tab), if enabled, or contact the maintainer through a private channel.
