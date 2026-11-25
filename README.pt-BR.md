# App educativo de IA na Cloudflare (plano gratuito)

Criado por [Adriano Meirinho](https://meirinho.com.br) · Licença MIT · [Read in English](README.md)

![Capa em fundo grafite com borda cinza de 2 px, o título «App de IA no plano gratuito da Cloudflare» à esquerda, a assinatura Adriano Meirinho no alto, o selo m no canto inferior esquerdo, a marca C v2 no canto inferior direito, a barra lima na base e um botão de play à direita.](docs/video-cover.jpg)

Um chat mínimo num Worker só: uma página e uma rota `/api/chat` que chama um modelo do Workers AI. O código cabe no editor do painel da Cloudflare e não tem build nem dependências.
O objetivo é didático: mostrar um app de IA pequeno feito do jeito certo, com limite de uso, verificação anti-robô, proteção contra prompt injection e nenhum dado guardado. Os princípios estão em [SECURITY.md](SECURITY.md) e o passo a passo para ensinar está em [SKILL.md](SKILL.md), ambos em inglês.

O código e a documentação estão em inglês. Os textos que o app mostra na tela estão em português do Brasil, porque o público é brasileiro.

## O que tem aqui

| Arquivo | Para quê |
| --- | --- |
| `src/worker.js` | O Worker inteiro (página + API). É este arquivo que você cola no painel. |
| `.dev.vars.example` | Modelo das variáveis, com valores de exemplo. |
| `wrangler.toml.example` | Modelo para quem publica com wrangler. |
| `SECURITY.md` | Princípios de segurança e como relatar um problema. |
| `SKILL.md` | O padrão em formato de skill e o roteiro «How to teach it on video». |
| `README.pt-BR.md` | Este README em português. |
| `LICENSE` | Licença MIT. |

## Antes de começar

- Uma conta Cloudflare no plano Workers Free. Nada aqui exige plano pago.
- Toda configuração real (limites, chaves, prompt e endereço) fica em variáveis e secrets do Worker, nunca no código nem no repositório.
- Se faltar alguma configuração obrigatória, o app recusa as perguntas. Ele nunca funciona sem limite.

## Configuração

| Nome | Tipo no painel | Obrigatório | O que é |
| --- | --- | --- | --- |
| `AI` | Binding Workers AI | sim | Acesso ao modelo. |
| `STATE` | Binding KV namespace | sim | Contadores efêmeros, que expiram sozinhos. |
| `RATE_LIMITER` | Binding Rate Limiting | não | Camada extra, só configurável com wrangler. |
| `RATE_LIMIT_PER_MINUTE` | Texto | sim | Perguntas por pessoa por minuto (proteção curta). |
| `WEEKLY_LIMIT_PER_IP` | Texto | sim | Perguntas por pessoa por semana. |
| `DAILY_QUESTION_CAP` | Texto | sim | Teto global de perguntas por dia. |
| `DAILY_NEURON_CAP` | Texto | sim | Teto global estimado em neurônios por dia. |
| `MAX_TOKENS` | Texto | sim | Tamanho máximo da resposta. |
| `MAX_INPUT_CHARS` | Texto | sim | Tamanho máximo da pergunta. |
| `ALLOWED_ORIGIN` | Texto | sim | Endereço exato da página, por exemplo `https://your-app.your-account.workers.dev`. |
| `TURNSTILE_SITE_KEY` | Texto | sim | Chave pública do widget Turnstile. |
| `TURNSTILE_SECRET_KEY` | Secret | sim | Chave secreta do widget Turnstile. |
| `HASH_SALT` | Secret | sim | Texto aleatório longo, usado para não guardar identificadores em claro. |
| `SYSTEM_PROMPT` | Secret | não | O prompt real. Sem ele, vale um exemplo genérico. |
| `APP_DISABLED` | Texto | não | `1` desliga o app na hora. |
| `AI_GATEWAY_ID` | Texto | não | Id de um AI Gateway seu, para mais uma camada de limite. |

Escolha os números pensando na cota gratuita do Workers AI (10 mil neurônios por dia) e no preço do modelo em neurônios, publicado na [página de preços](https://developers.cloudflare.com/workers-ai/platform/pricing/). Deixe uma folga grande, ainda mais se a mesma conta tiver outros apps usando Workers AI: a cota é da conta inteira.

## Publicar pelo painel (sem Git)

Os caminhos abaixo seguem a documentação oficial da Cloudflare.

1. **KV.** Em **Workers KV**, selecione **Create instance**, dê um nome e selecione **Create** ([doc](https://developers.cloudflare.com/kv/get-started/)).
2. **Binding do KV.** No seu Worker, abra a aba **Bindings**, selecione **Add binding** > **KV namespace**, use o nome de variável `STATE`, escolha o namespace e selecione **Add binding**.
3. **Binding do Workers AI**, se ainda não existir: em **Bindings**, adicione um binding de Workers AI com o nome `AI`.
4. **Turnstile.** Na página **Turnstile**, selecione **Add widget**, informe o nome, a região, o hostname exato do seu Worker e o modo **Managed**, e selecione **Create**. Copie a sitekey. A secret key vai direto para o passo 5, sem passar por chat, e-mail nem arquivo ([doc](https://developers.cloudflare.com/turnstile/get-started/widget-management/dashboard/)).
5. **Variáveis e secrets.** No Worker, em **Settings** > **Variables and Secrets** > **Add**, crie cada item da tabela acima com o tipo certo (Texto ou Secret) e selecione **Deploy** ([variáveis](https://developers.cloudflare.com/workers/configuration/environment-variables/), [secrets](https://developers.cloudflare.com/workers/configuration/secrets/)).
6. **Código.** Selecione **Edit Code**, troque todo o conteúdo pelo de `src/worker.js` e publique com **Deploy**.

## Publicar com wrangler

1. Copie `wrangler.toml.example` para `wrangler.toml` e preencha o id do namespace KV. O `.gitignore` já deixa esse arquivo fora do Git.
2. Crie os secrets com `npx wrangler secret put NOME` (o valor é digitado e não aparece na tela).
3. Crie as variáveis de texto pelo painel. O `keep_vars = true` do exemplo faz o wrangler mantê-las.
4. Publique com `npx wrangler deploy`.

Para testar na sua máquina, copie `.dev.vars.example` para `.dev.vars` e rode `npx wrangler dev`. As chaves de teste do Turnstile da documentação sempre passam, mas o hostname que elas devolvem pode não bater com `ALLOWED_ORIGIN`. Nesse caso, crie um widget de teste com o hostname `localhost`.

## Desligar em emergência

- Crie ou mude a variável `APP_DISABLED` para `1` e selecione **Deploy**. O app passa a responder «fora do ar» em todas as rotas.
- Ou, em **Settings** > **Domains & Routes**, desative o endereço `workers.dev` do Worker ([doc](https://developers.cloudflare.com/workers/configuration/routing/workers-dev/)). Isso não desativa as URLs de prévia de versões.

## Licença

Este repositório usa a licença MIT. Veja [LICENSE](LICENSE).
