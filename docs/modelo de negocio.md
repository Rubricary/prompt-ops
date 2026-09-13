# Modelo de Negócio — Hub Prompt Ops
 
Documento de estratégia comercial. Complementa o `0-como-montar-um-hub.md` (arquitetura) e o `1-prompt-ops.md` (produto).
 
**Mercado primário: Estados Unidos, B2B, em inglês.** Português brasileiro existe como coleção segmentada por filtro de idioma/jurisdição, não como posicionamento.
 
---
 
## O princípio que governa tudo
 
O acervo é grátis. O que se vende é **privacidade, governança e computação.**
 
Foi assim nos dois casos de referência:
 
| | Grátis | Onde vem o dinheiro |
|---|---|---|
| **GitHub** | repositórios públicos, por ~10 anos | Enterprise US$ 21/usuário/mês, Team US$ 4, Copilot US$ 10–39, Actions (computação medida), Marketplace. Estimado acima de US$ 2 bi/ano. Deu prejuízo até por volta de 2023. |
| **Hugging Face** | 1M+ modelos, datasets, Spaces | Inference Endpoints por GPU-hora (~40% da receita), Enterprise Hub US$ 20/usuário/mês (repos privados, SSO, controle de acesso, log de auditoria, ~35%), Pro US$ 9/mês. ~US$ 150M de ARR em 2026, partindo de ~US$ 30M em 2023. |
 
Duas lições que definem o cronograma abaixo:
 
1. **Nenhum dos dois cobrou no dia 1.** O Hugging Face só começou a monetizar em 2021, anos depois de nascer. Cobrar cedo mata o acervo, que é o ativo real.
2. **A receita pode ser pequena e o valor, enorme.** O Hugging Face foi avaliado a ~87× a receita e acabou comprado pela Nvidia por US$ 12,9 bi. Quem detém o acervo canônico de uma categoria vira alvo de aquisição de quem quer a distribuição.
---
 
## Fase 0 — Acervo em silêncio (meses 0–3)
 
**Objetivo: ter o que mostrar. Sem site público, sem divulgação, sem cadastro.**
 
A decisão é deliberada: um hub de prompts com trinta itens medianos não tem segunda visita. A primeira impressão só acontece uma vez, e ela é definida pela prateleira, não pela interface.
 
O que se constrói nesta fase:
 
- **Formato canônico fechado.** `prompt.md` + `config.yaml` + `evals.yaml`. Congelar o formato antes de produzir em escala — mudar o esquema com 200 itens prontos custa uma semana de retrabalho.
- **150–300 prompts em inglês**, cada um com bateria de testes e nota contra 3 modelos (ver `9-taxonomia-e-semeadura.md`).
- **O pipeline que gera a nota**, rodando em GitHub Actions. É a mesma camada de inteligência que depois roda em produção — construída aqui, testada contra o próprio acervo.
- **Repositórios bare já no formato final**, versionados desde o primeiro commit (ver infraestrutura abaixo).
O que **não** se constrói: site, busca, conta de usuário, cobrança, organização.
 
Métrica única desta fase: **prompts prontos com nota**. É a única coisa que importa.
 
### O prazo é parte da decisão
 
Adiar o lançamento tem custo real, e ele está listado em "o que pode dar errado": Braintrust, Langfuse, Promptfoo e Humanloop já fazem avaliação e nenhum tem vitrine pública de prompts provados. Essa brecha não fica aberta indefinidamente.
 
Por isso o silêncio precisa de teto: **90 dias ou 150 prompts com nota, o que vier primeiro.** Passado o teto, lança-se com o que existe. "Ainda não está bom" é o argumento que mantém projeto em gaveta por dois anos.
 
---
 
## Fase 1 — Lançamento público, tudo grátis (meses 3–6)
 
**Objetivo: tráfego e primeiros publicadores. Receita zero, por decisão.**
 
O que entra no ar:
 
- O acervo da Fase 0, com busca por setor, caso de uso, **locale** (`en-US` / `pt-BR`) e **jurisdiction** (`US` / `BR` / `none`).
- Ficha do prompt: o que faz, variáveis, nota por modelo, custo estimado por 1.000 chamadas, licença, versão.
- **Ferramenta grátis sem cadastro**: "cole seu prompt e receba a nota". É a porta de entrada, o equivalente ao PageSpeed no SEO — e a única peça que justifica sozinha o lançamento.
- `git clone` funcionando para todo item público.
- SDK de leitura e CLI (`push`, `add`, `login`).
O que **não** existe ainda: cobrança, organizações, privado, SSO.
 
Métrica única: **visitantes que colam um prompt no validador** e, destes, quantos criam conta. Nada mais importa.
 
**O validador é a maior despesa variável do projeto.** Cada colagem anônima gasta token seu. Cota rígida por IP e modelo barato no plano grátis, desde a primeira hora — senão a ferramenta de aquisição vira o buraco no caixa.
 
---
 
## Fase 2 — Primeira receita: privado (meses 6–10)
 
O gatilho para começar a cobrar é comportamental, não de calendário: **quando aparecerem pedidos espontâneos para tornar um prompt privado.** Antes disso, não há demanda, há suposição.
 
| Plano | Preço | O que entrega |
|---|---|---|
| Free | US$ 0 | Prompts públicos, 1 usuário, cota de avaliação por mês |
| Pro | US$ 19/usuário/mês | Prompts privados, histórico ilimitado, cota maior de avaliação |
 
Por que privado é o primeiro degrau: é a fronteira mais óbvia e menos negociável do produto. No momento em que a empresa coloca o system prompt de produção no hub, ele não pode estar na vitrine. É a mesma conversão que levou o GitHub de público para pago.
 
**Aqui também entra o BYOK** (*bring your own key*): a organização pluga a chave de API dela e as avaliações rodam por conta dela. Isso resolve o risco de custo da Fase 4 antes que ele apareça, e vira argumento de venda — o dado não sai do fornecedor que ela já aprovou.
 
---
 
## Fase 3 — Onde está o dinheiro: governança (meses 10–20)
 
Esta é a camada que sustenta o negócio. Sai do orçamento de **compliance e risco**, não do de ferramentas — é o que permite o preço.
 
| Plano | Preço | O que entrega |
|---|---|---|
| Team | US$ 29/usuário/mês | Organização, proposta de mudança, aprovador, papéis, editor para não-programador |
| Enterprise | US$ 1.500+/mês | SSO/SAML, trilha de auditoria exportável, ambiente isolado, SLA, retenção configurável, DPA |
 
O argumento de venda é uma pergunta que hoje ninguém consegue responder: *"qual prompt estava no ar quando este cliente recebeu esta resposta?"*. Em setor financeiro, saúde e qualquer empresa sob auditoria, isso é requisito, não conveniência.
 
Aqui também entra o dado que só o hub tem: **qual prompt regrediu, em que tipo de caso, agregado entre clientes.** É o equivalente do dado de uso do Hugging Face — não se copia, se acumula.
 
---
 
## Fase 4 — Computação e escala (meses 20+)
 
Margem sobre algo que você roda. É a linha que cresceu mais rápido nos dois casos de referência (Inference Endpoints, GitHub Actions).
 
| Linha | Cobrança | Observação |
|---|---|---|
| Entrega em runtime | US$ por 1.000 chamadas acima da cota | O SDK consultando a versão `production` |
| Execução de avaliação | margem sobre o custo de tokens | Para quem não usa BYOK e prefere não gerenciar chave |
| Teste A/B | incluído no Enterprise, medido no Team | 10% do tráfego na versão nova |
| API de leitura pública | por consulta, acima da cota | Agentes e ferramentas consumindo o acervo — é o que cria dependência |
 
---
 
## Linhas complementares (não priorizar antes da fase 3)
 
- **Verificação/selo** — auditoria de um prompt ou de uma suite, renovável. Quem vende software com IA quer o selo.
- **Marketplace** — prompts pagos de especialistas, com comissão. Só faz sentido depois que houver autores externos publicando de graça.
- **Coleção pt-BR** — não é linha de receita separada; é diferencial de busca. O filtro por jurisdição é o que torna defensável um prompt de contrato brasileiro contra um americano.
---
 
## Infraestrutura e custo por fase
 
O princípio: **custo fixo zero até existir cliente pagante.** Cada degrau de infraestrutura é pago por uma receita que já entrou, não por uma que se espera.
 
| Fase | Infraestrutura | Custo mensal |
|---|---|---|
| 0 — silêncio | Repositórios bare locais + R2 (10 GB grátis) · Supabase free · GitHub Actions como worker | **US$ 0** |
| 1 — lançamento | + Cloudflare Workers/Pages (100k req/dia) · R2 público via CDN para clone e runtime | **US$ 0** |
| 2 — privado | + Workers Paid (US$ 5) · Supabase Pro (US$ 25, remove a pausa por inatividade e liga backup) | **~US$ 30** |
| 3 — governança | + VPS Hetzner (~€4) com Forgejo, servidor git próprio | **~US$ 35** |
| 4 — escala | + fila dedicada, réplica, região isolada por contrato Enterprise | conforme contrato |
 
**Fora da Vercel, por decisão.** O plano Hobby proíbe uso comercial de forma explícita — vale desde o primeiro dia, não a partir de certo faturamento. Cloudflare Workers e Supabase permitem uso comercial no plano gratuito. Next.js roda no Cloudflare via `@opennextjs/cloudflare`, com atenção a `next/image` (delegar ao Cloudflare Images) e a ISR.
 
### Camada 5 — git sem servidor, desde o dia 1
 
O formato git entra na Fase 0, não depois. O que muda é só quem serve:
 
- **Fases 0–2:** repositório *bare* servido como **arquivo estático no R2**, atrás de domínio próprio. `git clone https://git.seuhub.com/org/prompt.git` funciona sem nenhum processo rodando, com egress zero. A escrita não passa por smart HTTP: a CLI e o editor web chamam a API, e o job que já roda a bateria de testes faz o commit, o `git update-server-info` e a sincronia para o bucket.
- **Fase 3 em diante:** Forgejo na VPS, com `git push` direto do usuário. A migração é `git push --mirror`, porque o formato nunca deixou de ser git.
Não haver push direto do usuário nas primeiras fases não é limitação neste hub: o fluxo de aprovação exige que toda mudança entre pela proposta, não por push cru.
 
### Custo variável — o único desembolso real
 
| Item | Quando | Ordem de grandeza |
|---|---|---|
| Domínio | Fase 0 | ~US$ 12/ano |
| Tokens para pontuar o acervo semeado | Fase 0, uma vez | dezenas de dólares, com modelos baratos |
| Tokens do validador anônimo | Fase 1 em diante | **cresce com o sucesso** — exige cota rígida |
| Tokens de avaliação de cliente | Fase 2 em diante | zero, se BYOK |
 
---
 
## Cronograma de migração técnica
 
| Momento | Gatilho | Mudança |
|---|---|---|
| Fim da Fase 0 | acervo pronto | Sobe o site no Cloudflare Workers; R2 vira público |
| Primeiro cliente pagante | receita recorrente existe | Supabase Pro, Workers Paid, VPS Hetzner com Forgejo |
| ~1.000 repos | busca lenta | Índice dedicado, cache agressivo na borda |
| ~10.000 repos | Forgejo vira gargalo | `git-http-backend` próprio (opção A do doc 0) |
| Primeiro Enterprise | exigência contratual | Ambiente isolado, região de dados, DPA, log de auditoria exportável |
| Avaliação > 30% do custo | conta de API subindo | BYOK obrigatório acima da cota, cache de resultado por hash do prompt |
 
A ordem do doc 0 permanece válida: **semeadura antes de cobrança.** Ninguém paga por prateleira vazia — e, na versão revisada, ninguém sequer visita.
 
---
 
## O que pode dar errado
 
- **Ficar em silêncio além do teto.** A Fase 0 é uma aposta com prazo. Sem o corte de 90 dias, ela vira adiamento permanente disfarçado de capricho.
- **Cobrar cedo demais.** Mata o acervo e o acervo é o ativo.
- **Concorrente com acervo melhor.** Braintrust, Langfuse, Promptfoo, Humanloop já fazem avaliação. Nenhum tem vitrine pública de prompts provados — essa é a brecha, e ela fecha.
- **Custo de avaliação sem cota.** O plano grátis vira prejuízo linear ao sucesso. BYOK resolve para cliente identificado; para o validador anônimo, só cota resolve.
- **Enterprise cedo demais.** Um contrato grande no mês 6 consome o ano inteiro em requisitos de compliance e paralisa o produto.
- **Pausa do Supabase free.** Projeto sem tráfego por 7 dias é suspenso e volta com 30 segundos de cold start. Na Fase 0 é irrelevante; na Fase 1, um ping agendado resolve; a partir da Fase 2, é motivo suficiente para o plano Pro.