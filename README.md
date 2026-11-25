# Educational AI app on Cloudflare (free plan)

By [Adriano Meirinho](https://meirinho.com.br) · MIT License · [Leia em português](README.pt-BR.md)

![Cover on a graphite background with a 2-pixel gray border, the title “App de IA no plano gratuito da Cloudflare” on the left, the name Adriano Meirinho at the top, an m seal in the lower-left corner, the C v2 mark in the lower-right corner, a lime bar along the bottom, and a play button on the right.](docs/video-cover.jpg)

A minimal chat in a single Worker: one page and one `/api/chat` route that calls a Workers AI model. The code fits in the Cloudflare dashboard editor and has no build step and no dependencies.
The goal is educational: show a small AI app done the right way, with usage limits, bot verification, prompt injection protection and no stored data. The principles are in [SECURITY.md](SECURITY.md) and the teaching walkthrough is in [SKILL.md](SKILL.md).

The code and docs are in English. The text the app shows on screen is in Brazilian Portuguese, because the audience is Brazilian.

## What is in here

| File | Purpose |
| --- | --- |
| `src/worker.js` | The whole Worker (page + API). This is the file you paste into the dashboard. |
| `.dev.vars.example` | Template for the variables, with example values. |
| `wrangler.toml.example` | Template for those who deploy with wrangler. |
| `SECURITY.md` | Security principles and how to report a problem. |
| `SKILL.md` | The pattern as a skill and the «How to teach it on video» script. |
| `README.pt-BR.md` | This README in Portuguese. |
| `LICENSE` | MIT License. |

## Before you start

- A Cloudflare account on the Workers Free plan. Nothing here requires a paid plan.
- All real configuration (limits, keys, prompt and address) lives in Worker variables and secrets, never in the code or in the repository.
- If any required setting is missing, the app refuses questions. It never runs without limits.

## Configuration

| Name | Type in the dashboard | Required | What it is |
| --- | --- | --- | --- |
| `AI` | Workers AI binding | yes | Access to the model. |
| `STATE` | KV namespace binding | yes | Short-lived counters that expire on their own. |
| `RATE_LIMITER` | Rate Limiting binding | no | Extra layer, only configurable with wrangler. |
| `RATE_LIMIT_PER_MINUTE` | Text | yes | Questions per person per minute (short-term protection). |
| `WEEKLY_LIMIT_PER_IP` | Text | yes | Questions per person per week. |
| `DAILY_QUESTION_CAP` | Text | yes | Global cap on questions per day. |
| `DAILY_NEURON_CAP` | Text | yes | Global cap on estimated neurons per day. |
| `MAX_TOKENS` | Text | yes | Maximum answer length. |
| `MAX_INPUT_CHARS` | Text | yes | Maximum question length. |
| `ALLOWED_ORIGIN` | Text | yes | Exact address of the page, for example `https://your-app.your-account.workers.dev`. |
| `TURNSTILE_SITE_KEY` | Text | yes | Public key of the Turnstile widget. |
| `TURNSTILE_SECRET_KEY` | Secret | yes | Secret key of the Turnstile widget. |
| `HASH_SALT` | Secret | yes | Long random text, used so identifiers are never stored in the clear. |
| `SYSTEM_PROMPT` | Secret | no | The real prompt. Without it, a generic example is used. |
| `APP_DISABLED` | Text | no | `1` turns the app off immediately. |
| `AI_GATEWAY_ID` | Text | no | ID of your own AI Gateway, for one more limit layer. |

Pick the numbers with the Workers AI free allocation (10,000 neurons per day) and the model's price in neurons in mind, as published on the [pricing page](https://developers.cloudflare.com/workers-ai/platform/pricing/). Leave plenty of headroom, especially if other apps in the same account use Workers AI: the allocation is shared by the whole account.

## Deploy from the dashboard (no Git)

The paths below follow the official Cloudflare documentation.

1. **KV.** In **Workers KV**, select **Create instance**, give it a name and select **Create** ([docs](https://developers.cloudflare.com/kv/get-started/)).
2. **KV binding.** In your Worker, open the **Bindings** tab, select **Add binding** > **KV namespace**, use the variable name `STATE`, pick the namespace and select **Add binding**.
3. **Workers AI binding**, if it does not exist yet: in **Bindings**, add a Workers AI binding named `AI`.
4. **Turnstile.** On the **Turnstile** page, select **Add widget**, enter the name, the region, your Worker's exact hostname and the **Managed** mode, and select **Create**. Copy the sitekey. The secret key goes straight to step 5, never through chat, email or a file ([docs](https://developers.cloudflare.com/turnstile/get-started/widget-management/dashboard/)).
5. **Variables and secrets.** In the Worker, under **Settings** > **Variables and Secrets** > **Add**, create each item from the table above with the right type (Text or Secret) and select **Deploy** ([variables](https://developers.cloudflare.com/workers/configuration/environment-variables/), [secrets](https://developers.cloudflare.com/workers/configuration/secrets/)).
6. **Code.** Select **Edit Code**, replace all the content with `src/worker.js` and publish with **Deploy**.

## Deploy with wrangler

1. Copy `wrangler.toml.example` to `wrangler.toml` and fill in the KV namespace ID. The `.gitignore` already keeps that file out of Git.
2. Create the secrets with `npx wrangler secret put NAME` (you type the value and it is not shown).
3. Create the text variables in the dashboard. The example's `keep_vars = true` makes wrangler keep them.
4. Deploy with `npx wrangler deploy`.

To test on your machine, copy `.dev.vars.example` to `.dev.vars` and run `npx wrangler dev`. The Turnstile test keys from the documentation always pass, but the hostname they return may not match `ALLOWED_ORIGIN`. In that case, create a test widget with the hostname `localhost`.

## Emergency shutdown

- Create or change the `APP_DISABLED` variable to `1` and select **Deploy**. The app then answers «O app está fora do ar no momento.» (out of service) on every route.
- Or, under **Settings** > **Domains & Routes**, disable the Worker's `workers.dev` address ([docs](https://developers.cloudflare.com/workers/configuration/routing/workers-dev/)). This does not disable version preview URLs.

## License

This repository uses the MIT License. See [LICENSE](LICENSE).
