# Segurança

Este app é pequeno de propósito, mas segue os mesmos princípios de um app de IA em produção. Este documento explica os princípios, não os parâmetros. Os valores reais e os detalhes de configuração ficam fora do repositório.

## Princípios

1. **Configuração fora do código.** Limites, chaves, prompt e endereço permitido vêm de variáveis e secrets do Worker. O repositório só traz exemplos.
2. **Falha fechada.** Se faltar configuração, se a verificação anti-robô falhar ou se um contador não puder ser atualizado, o app recusa a pergunta. Ele nunca passa a funcionar sem limite.
3. **Limite de uso em camadas.** Há limite por pessoa em janela curta, limite por pessoa por semana e um teto global diário bem abaixo da cota gratuita, para sobrar recurso para outros apps. As camadas se sobrepõem, para que a falha de uma não deixe o app aberto.
4. **Verificação anti-robô.** Cada pergunta exige um token do Cloudflare Turnstile, validado no servidor antes de qualquer gasto. O token vale uma vez só.
5. **Mesma origem.** A API só aceita pedidos vindos da própria página, com corpo JSON pequeno e no formato esperado.
6. **Proteção contra prompt injection.**
   - O system prompt fica no servidor, junto de regras fixas que nenhum texto do usuário pode mudar.
   - A pergunta vai num campo separado, delimitado e limitado, e é tratada como dado.
   - O histórico não vem do navegador: cada pergunta é avaliada sozinha.
7. **Saída sempre como texto.** A resposta do modelo entra na página como texto puro, nunca como HTML. A página tem Content Security Policy restritiva com nonce, bloqueio de iframe externo e cabeçalhos de segurança.
8. **Nenhum dado guardado.** O app não registra perguntas nem respostas. Os contadores guardam só números, sob chaves derivadas com sal secreto, e expiram sozinhos.
9. **Mensagens educativas e genéricas.** Os erros lembram que este é um app educativo de demonstração e dizem o que a pessoa pode fazer, sem detalhar números nem como a regra funciona.
10. **Interruptor de emergência.** Uma variável desliga o app na hora, sem mexer no código.
11. **Custo zero.** Tudo roda no plano gratuito. Se uma cota gratuita acaba, o serviço para em vez de gerar cobrança.

## Limitações conhecidas

- Contadores distribuídos são aproximados por natureza. Por isso há várias camadas e um teto global com folga.
- Nenhuma defesa contra prompt injection é perfeita. As regras reduzem o risco, e a saída como texto impede que uma resposta manipulada vire código na página.

## Como relatar um problema

Encontrou uma falha? Não abra issue pública. Use o recurso de relato privado de vulnerabilidade do GitHub neste repositório (aba Security), se estiver ativo, ou fale com o mantenedor por um canal privado.
