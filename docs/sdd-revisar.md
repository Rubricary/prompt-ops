# SDD — Rubricary Prompt Ops

**Versão** 0.1 · **Data** 13/09/2026 · **Escopo** Fases 0 e 1 (`1_2-modelo-de-negocio.md`)
Complementa `prd.md`. A arquitetura de referência está em `0-como-montar-um-hub.md` §Arquitetura em camadas — este documento só decide a implementação concreta de cada camada e não repete a justificativa.

Decisões vindas do `prd.md` §Anexo A citadas como **D1**…**D12**.

---

## 1. Princípio que governa a arquitetura

**Custo fixo zero até existir cliente pagante.** Cada peça abaixo foi escolhida por caber num plano gratuito com uso comercial permitido, e por ter um caminho de saída que não exige reescrita.

Corolário prático: **nada que precise ficar rodando 24/7.** Sem VPS, sem container, sem processo de servidor git, sem fila dedicada. Tudo é arquivo estático, função sob demanda ou job agendado.

## 2. Mapa das camadas — genérico → concreto

| Camada (`0`) | Genérico | Aqui, no MVP | Saída futura |
|---|---|---|---|
| 1 Site | Next.js/Vercel | Next.js via `@opennextjs/cloudflare` em Cloudflare Workers (**D3**) | mesmo |
| 2 API + CLI | REST + npx | Rotas do Worker + pacote npm `rubricary` | mesmo |
| 3 Índice | Postgres | Supabase free (Postgres + FTS) | Supabase Pro |
| 4 Inteligência | fila + workers | GitHub Actions (Fase 0) → Cloudflare Queues/Cron (Fase 1) | fila dedicada |
| 5 Servidor git | Gitea/Forgejo | **nenhum** — bare estático no R2, dumb HTTP (**D4**) | Forgejo em VPS |
| 6 Armazenamento | bucket | Cloudflare R2, egress zero | mesmo |

---

## 3. Formato canônico (congelar antes de produzir)

Dois arquivos por repositório (**D1**). Mudar isto depois de 50 prompts custa uma semana.

```
prompts/{setor}/{caso-de-uso}.git
├── prompt.md      # frontmatter YAML + corpo do prompt
└── evals.yaml     # 8–15 casos, chaves em inglês (D8)
```

Campos do frontmatter: ver `1_1-taxonomia-e-semeadura.md` §Parte 4, com dois ajustes:

- `model:` passa a significar **modelo recomendado**, não modelo de teste (**D12**).
- novo campo `eval_matrix: [modelo-a, modelo-b, modelo-c]` — a matriz contra a qual a nota é medida.

A nota **não vive no repositório**. É resultado derivado, gerado pelo pipeline e guardado no índice, chaveado por `sha256(prompt.md + evals.yaml + modelo)`. Motivo: nota é função do modelo, e o modelo muda sem o prompt mudar. Colocá-la no git geraria commit espúrio a cada re-avaliação.

---

## 4. Camada 6 — armazenamento

- Um bucket R2, prefixo `repos/{org}/{nome}.git/`.
- Texto puro comprime a 10–20%; 300 prompts com histórico ficam abaixo de 100 MB. O tier gratuito (10 GB) só aperta em outra ordem de grandeza.
- **Egress zero** é o que torna o clone público viável sem receita.
- Verdade durável = o bucket. Não há disco anexado no MVP porque não há máquina.

## 5. Camada 5 — git sem servidor

O ponto mais incomum da arquitetura, e o que zera o custo.

**Leitura (clone público):** o protocolo *dumb HTTP* do git precisa apenas de arquivos estáticos servidos por HTTPS. Bastam, dentro do repo bare:

- `info/refs` e `objects/info/packs`, gerados por `git update-server-info`
- os packfiles em `objects/pack/`

`git clone https://git.rubricary.com/rubricary/nome.git` funciona sem nenhum processo rodando. Domínio próprio apontado ao bucket R2 (custom domain, gratuito).

**Requisito operacional:** rodar `git gc --aggressive && git update-server-info` **antes de cada sincronia**. Sem empacotar, o clone vira milhares de requisições de objeto solto — o que degrada a latência e consome operações Class B do R2. Com pack, é meia dúzia de requisições por clone.

**Escrita:** não passa por git (**D5**). O caminho é:

```
CLI / editor web → POST /v1/prompts → job de avaliação → commit local no runner
→ git gc + update-server-info → upload do repo para o R2 → upsert no índice
```

O runner é o GitHub Actions na Fase 0 e um Worker + Queue na Fase 1. Em nenhum momento existe smart HTTP.

**Migração futura:** `git push --mirror` para o Forgejo. O formato nunca deixou de ser git — é exatamente a aposta do `0` §Por que usar git.

## 6. Camada 4 — inteligência (o diferencial)

É o único código que ninguém copia rápido. Três funções:

**6.1 Runner de evals.** Lê `evals.yaml`, executa cada caso contra cada modelo do `eval_matrix`, aplica o julgamento e devolve `{passed, total, score, por_caso}`. Dois tipos de caso:

- `type: exact` / `regex` / `json_schema` — verificação determinística, custo zero de token extra.
- `type: judge` — outro modelo corrige segundo o `criteria`. Usar o modelo mais barato disponível como juiz; o custo do juiz é ~metade do custo total da suite.

Isto é **interno ao Prompt Ops**, não um serviço externo (**D7**). O formato de caso vem de `2-hub-de-evals.md`; a execução mora aqui.

**6.2 Estimador de custo.** Conta tokens de entrada do prompt renderizado + média de saída observada nos casos, multiplica pela tabela de preço por modelo. Alimenta o campo "custo por 1.000 chamadas" da ficha — que é o dado que `1-prompt-ops.md` §Funcionalidades chama de detector de regressão de custo.

**6.3 Cache por hash.** Toda avaliação é chaveada por `sha256(prompt + evals + modelo + versão-do-runner)`. Re-executar um prompt inalterado custa zero. Sem isto, re-avaliar o acervo inteiro a cada modelo novo é uma conta de API desnecessária.

**Idempotência:** o job precisa ser seguro para re-execução — o GitHub Actions vai falhar no meio de lotes de 50 prompts.

## 7. Camada 3 — índice

Supabase free. Só metadados; o conteúdo nunca entra no banco (`0` §regra de ouro).

| Tabela | Conteúdo essencial |
|---|---|
| `orgs` | slug, nome, tipo (plataforma/externa) |
| `prompts` | org_id, slug, setor, caso_de_uso, locale, jurisdiction, licença, sha do commit atual, versão `production` |
| `versions` | prompt_id, semver, sha, autor, data, mensagem |
| `evaluations` | version_id, modelo, score, passed/total, custo_1k, hash_cache, data |
| `events` | tipo, ator, alvo, payload — base da futura trilha de auditoria |
| `quota` | chave (IP ou conta), janela, contador — ver §9 |

Busca: `tsvector` nativo do Postgres sobre nome + descrição + caso de uso. Sem Elasticsearch, sem embeddings no MVP.

**Armadilha conhecida:** o projeto Supabase free é pausado após 7 dias sem tráfego e volta com ~30 s de cold start. Na Fase 0 é irrelevante; na Fase 1, um Cron Trigger do Cloudflare fazendo um ping diário resolve. A partir do primeiro cliente pagante, é motivo suficiente para o plano Pro.

## 8. Camada 2 — API e CLI

API pública, sem autenticação, cacheada agressivamente na borda:

```
GET  /v1/prompts?sector=&use_case=&locale=&jurisdiction=&q=
GET  /v1/prompts/:org/:slug
GET  /v1/prompts/:org/:slug@:version
GET  /v1/prompts/:org/:slug/evaluations
POST /v1/validate          # validador anônimo — com cota
```

Autenticada (token com escopo), só a partir de RF-07:

```
POST /v1/prompts           # cria/atualiza → dispara job
POST /v1/auth/token
```

CLI: pacote npm `rubricary` (já reservado). `npx rubricary add <slug>` detecta o projeto e escreve os arquivos; `push` chama a API. Nunca um protocolo proprietário de transporte — o repositório continua clonável por git puro.

A API de leitura sem autenticação é estratégica, não conveniência: é o que faz agentes e outras ferramentas dependerem do hub (`0` §Camada 2).

## 9. Controle de cota do validador

O validador anônimo é a maior despesa variável do projeto (`1_2` §Fase 1). Três camadas de defesa, todas dentro do plano gratuito:

1. **Turnstile** (Cloudflare, gratuito) antes do POST — elimina o abuso automatizado, que é o grosso.
2. **Contador por IP** em Cloudflare D1 — janela deslizante, ex. 3 validações/hora. KV não serve: o limite de 1.000 escritas/dia do tier gratuito estoura antes do de leitura.
3. **Teto de tamanho e de modelo**: prompt truncado em N tokens, avaliação anônima sempre no modelo mais barato, suite reduzida a ~5 casos gerados.

Sem isto, o custo do produto cresce linearmente com o sucesso da aquisição.

## 10. Camada 1 — site

Next.js em Cloudflare Workers (**D3**). Conteúdo é texto pequeno e majoritariamente público — cache na borda por padrão, revalidação disparada pelo job de publicação.

Rotas mínimas do MVP (**telas ainda não desenhadas — isto é só o mapa de URLs**):

```
/                         vitrine + entrada do validador
/validate                 validador anônimo
/prompts                  busca com filtros
/prompts/:org/:slug       ficha do prompt
/prompts/:org/:slug/diff  comparação entre versões
/:org                     página da organização
```

Atenção a duas armadilhas do `@opennextjs/cloudflare`: `next/image` deve ser delegado ao Cloudflare Images, e ISR se comporta de forma diferente do runtime da Vercel — preferir cache explícito na borda a revalidação implícita.

---

## 11. Infraestrutura — MVP de custo zero

| Serviço | Plano | Limite relevante | Uso comercial? |
|---|---|---|---|
| Cloudflare Workers/Pages | Free | 100k req/dia | **sim** |
| Cloudflare R2 | Free | 10 GB, 1M ops Class A/mês, **egress zero** | sim |
| Cloudflare D1 | Free | 100k escritas/dia | sim |
| Cloudflare Turnstile | Free | ilimitado | sim |
| Supabase | Free | 500 MB, pausa após 7 dias ociosos | **sim** |
| GitHub Actions | Free | ilimitado em repo público | sim |
| Brevo | Free | e-mail transacional | sim |
| **Vercel** | Hobby | — | **não** — uso comercial proibido, fora por decisão (**D3**) |

**Custo fixo mensal: US$ 0.**

Desembolso real:

| Item | Quando | Ordem de grandeza |
|---|---|---|
| Domínios (já comprados) | — | ~US$ 25/ano |
| Tokens para pontuar os 50 prompts | Fase 0, uma vez | dezenas de dólares |
| Tokens do validador anônimo | Fase 1 em diante | cresce com o sucesso — por isso §9 |

## 12. Gatilhos de migração

Nenhuma peça acima é escolhida para sempre. Cada troca tem um gatilho observável, não uma data:

| Gatilho | Mudança |
|---|---|
| Supabase pausando em horário de tráfego | Supabase Pro (US$ 25) |
| >100k req/dia | Workers Paid (US$ 5) |
| Primeiro cliente pagante | VPS Hetzner + Forgejo, `git push` direto |
| ~1.000 repositórios | índice dedicado, cache mais agressivo |
| ~10.000 repositórios | `git-http-backend` próprio (`0` §Camada 5, opção A) |
| Avaliação > 30% do custo | BYOK obrigatório acima da cota |
| Primeiro Enterprise | ambiente isolado, região de dados, DPA |

## 13. Riscos técnicos

1. **Dumb HTTP mal empacotado** — repositório sem `git gc` torna o clone lento e caro. Deve ser passo obrigatório do pipeline, não manual.
2. **Formato congelado tarde** — mudar `prompt.md`/`evals.yaml` depois dos 50 prompts custa uma semana de retrabalho. Congelar antes do primeiro lote é requisito da Fase 0, não recomendação.
3. **Não-determinismo do juiz** — o mesmo prompt pode receber notas diferentes em execuções distintas. Mitigação: temperatura 0 no juiz, critério escrito de forma verificável, e nota exibida com a data e o modelo, nunca como número absoluto.
4. **Cold start acumulado** — Worker + Supabase pausado + primeira avaliação pode estourar os 30 s do critério de aceite. O ping agendado é obrigatório desde o dia do lançamento.
5. **Reescrita de histórico** — como o commit é feito pelo runner e não pelo usuário, um bug no job pode sobrescrever histórico no R2. Upload sempre por prefixo novo + troca de ponteiro, nunca sobrescrita in-place.
