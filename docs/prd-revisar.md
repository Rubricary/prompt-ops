# PRD — Rubricary Prompt Ops

**Versão** 0.1 · **Data** 13/09/2026 · **Status** rascunho para scaffold
**Repo** `github.com/Rubricary/prompt-ops` · **Domínios** `rubricary.com`, `rubricary.dev`

## Documentos de referência

Este PRD não repete conteúdo. Ele aponta.

| Doc | O que contém | Autoridade sobre |
|---|---|---|
| `0-como-montar-um-hub.md` | arquitetura genérica em 6 camadas | vocabulário de camadas |
| `1-prompt-ops.md` | produto, dores, fluxos, funcionalidades | comportamento do produto |
| `1_1-taxonomia-e-semeadura.md` | 10 setores, template, método de produção | conteúdo do acervo |
| `1_2-modelo-de-negocio.md` | fases 0–4, preços, infraestrutura por fase | cronograma e comercial |
| `sdd.md` | arquitetura concreta do MVP | implementação |

Onde houver conflito, vale a ordem: `sdd.md` > `1_2` > `1_1` > `1` > `0`. Os conflitos conhecidos estão resolvidos no Anexo A.

---

## 1. Produto em uma frase

O prompt de produção deixa de ser texto solto dentro do código e vira um artefato versionado, testado e endereçável — `org/prompt@versão` — com vitrine pública de prompts que já vêm com prova.

## 2. Por que este produto existe

Dor: `1-prompt-ops.md` §A dor.
Brecha de mercado: `1_2` §O que pode dar errado — os concorrentes (Braintrust, Langfuse, Promptfoo, Humanloop) avaliam, mas nenhum tem vitrine pública de prompts provados.
Evidência de que o acervo não tem concorrência: `1_1` §Parte 1 — 0% dos 2.720 prompts públicos levantados têm bateria de testes.

## 3. Usuários

| # | Persona | Dor principal | O que ela faz no produto |
|---|---|---|---|
| U1 | Visitante anônimo (dev ou PM) | não sabe se o prompt dele é bom | cola no validador, recebe nota |
| U2 | Autor do prompt (não-programador) | edita direto em produção, sem revisão | edita no navegador, abre proposta |
| U3 | Dev integrador | prompt hardcoded exige deploy | instala SDK, lê `production` |
| U4 | Aprovador / compliance | não sabe qual prompt gerou qual resposta | revisa diff + nota, aprova, audita |

**No MVP só U1 e U3 são atendidos.** U2 e U4 entram na Fase 3 (`1_2`). Isso é deliberado: o que o MVP precisa provar é tráfego para o validador, não governança.

## 4. Escopo

### Fase 0 — acervo em silêncio (sem produto público)

Definida em `1_2` §Fase 0. Entrega:

- **E0.1** Formato canônico congelado (Anexo A, decisão D1).
- **E0.2** 50 prompts de lançamento — Support 20, Sales 15, Engineering 15 (`1_1` §Parte 3), cada um com `evals.yaml` e nota contra 3 modelos.
- **E0.3** Pipeline de avaliação rodando em GitHub Actions.
- **E0.4** Repositórios bare no formato final, versionados desde o primeiro commit.

Métrica única: prompts prontos com nota.
Teto: **90 dias ou 50 prompts com nota, o que vier primeiro** (Anexo A, D2).

### Fase 1 — MVP público, tudo grátis

| ID | Requisito | Prio |
|---|---|---|
| RF-01 | Validador anônimo: cola um prompt, recebe nota, sem cadastro, com cota por IP | P0 |
| RF-02 | Busca por setor, caso de uso, `locale`, `jurisdiction` | P0 |
| RF-03 | Ficha do prompt: texto, variáveis, nota por modelo, custo/1k chamadas, licença, versão, histórico | P0 |
| RF-04 | `git clone` funcionando para todo item público | P0 |
| RF-05 | API de leitura pública sem autenticação (`GET /v1/prompts/:org/:nome`) | P0 |
| RF-06 | SDK npm `rubricary` — `get(nome, {version})` com cache local | P0 |
| RF-07 | CLI `npx rubricary push \| add \| login` | P1 |
| RF-08 | Diff entre duas versões de um prompt | P1 |
| RF-09 | Página de organização/autor | P2 |
| RF-10 | Cadastro de conta (só para quem quer publicar) | P2 |

Métrica única: **visitantes que colam um prompt no validador** e, destes, quantos criam conta.

### Fora de escopo no MVP

Cobrança, planos, prompts privados, organizações, papéis, SSO, trilha de auditoria, editor com aprovação, teste A/B, marketplace, selo, `git push` direto do usuário, execução de eval do cliente.

Os três primeiros só entram quando aparecer **pedido espontâneo por prompt privado** — gatilho comportamental definido em `1_2` §Fase 2.

## 5. Regras de produto inegociáveis

1. Nenhum prompt entra no acervo sem `evals.yaml` (`1_1` §Parte 4). É a única coisa que separa isto de uma lista de prompts.
2. O acervo público é grátis e clonável para sempre. O que se vende é privacidade, governança e computação (`1_2`).
3. Curadoria e selo são da plataforma, não pessoais, na fase 1 (`1_1` §Parte 5).
4. Todo item importado ou derivado carrega crédito à origem e licença compatível — só CC0 e MIT (`1_1` §Parte 1).
5. O validador tem cota rígida desde a primeira hora. É a maior despesa variável do projeto.

## 6. Critérios de aceite do MVP

- Um visitante que nunca ouviu falar do produto cola um prompt e recebe nota em menos de 30 s, sem cadastro.
- `git clone https://git.rubricary.com/rubricary/refund-request-outside-policy.git` funciona de uma máquina limpa.
- `npx rubricary add refund-request-outside-policy` escreve os arquivos no projeto local.
- A ficha do prompt responde em 5 s: o que faz, se é confiável, quem mantém, como usar (`0` §Camada 1).
- Custo fixo mensal de infraestrutura: US$ 0 (só domínio e tokens).

---

## Anexo A — Conflitos entre os documentos e decisões

Levantados na leitura dos cinco documentos.

| # | Conflito | Onde | Decisão |
|---|---|---|---|
| D1 | Formato: `1_2` diz 3 arquivos (`prompt.md` + `config.yaml` + `evals.yaml`); `1_1` §Parte 4 diz 2 arquivos com o config no frontmatter; `1` §Fluxo diz `prompt.md` + `config.yaml` | 1 / 1_1 / 1_2 | **Dois arquivos** (decidido, congelado). `prompt.md` com frontmatter YAML + `evals.yaml`. Razão decisiva: o aprovador precisa ver mudança de texto e mudança de configuração no mesmo diff. Spec em `sdd.md` §3 |
| D2 | Volume de lançamento: `1` e `1_2` dizem 150–300 prompts; `1_1` §Parte 3 diz ~50 (20+15+15) | 1_1 vs 1_2 | **50 é o teto da Fase 0**, 150–300 é a meta do primeiro ano. Os 50 cobrem 3 setores por inteiro, que é o que faz a prateleira parecer cheia; 150 rasos em 10 setores não. Decidido — substitui o teto de 150 do `1_2` |
| D3 | Hospedagem: `0` §Camada 1 diz "Next.js na Vercel"; `1_2` diz "fora da Vercel, por decisão" (Hobby proíbe uso comercial) | 0 vs 1_2 | **Cloudflare.** `0` é documento genérico e foi escrito antes; `1_2` prevalece |
| D4 | Servidor git: `0` §Camada 5 recomenda "B (Gitea/Forgejo) para o MVP"; `1_2` diz bare estático no R2 nas fases 0–2 e Forgejo só na Fase 3 | 0 vs 1_2 | **R2 estático.** Forgejo é custo fixo (VPS) e a regra é custo fixo zero até haver cliente pagante |
| D5 | Escrita: `0` §Camada 2 diz "por baixo a CLI faz um `git push` comum"; `1_2` diz que a escrita passa pela API, não por smart HTTP | 0 vs 1_2 | **API.** O fluxo de aprovação exige que toda mudança entre por proposta. `git push` direto volta na Fase 3 |
| D6 | Cronograma: `0` §Ordem de construção põe cobrança na semana 11 de 12; `1_2` só cobra na Fase 2 (mês 6+) | 0 vs 1_2 | **`1_2`.** As 12 semanas do `0` são ordem de construção genérica, não cronograma comercial |
| D7 | Dependência: `1` §Fluxo diz que os testes rodam "ver Hub 2", mas o Hub de Evals não será construído | 1 | **O runner de evals é parte do Prompt Ops**, não um serviço externo. Formato de caso vem do `2-hub-de-evals.md`; a execução é interna (ver `sdd.md` §Camada 4) |
| D8 | Idioma do schema: `evals.yaml` usa chaves em português (`entrada`, `criterio`, `tipo`, `peso`) num produto em inglês para os EUA | 2 / 1_1 | **Inglês** (decidido): `input`, `criteria`, `type`, `weight`. Schema canônico em `sdd.md` §3; os demais hubs adotam o mesmo se forem construídos |
| D9 | Preços: `1` §Monetização não tem tier Pro e põe Time em US$ 29; `1_2` tem Pro US$ 19 (Fase 2) e Team US$ 29 (Fase 3) | 1 vs 1_2 | **`1_2`.** Fora do escopo do MVP de qualquer forma |
| D10 | Referência quebrada: `1_2` §Fase 0 cita `9-taxonomia-e-semeadura.md`; o arquivo é `1_1-taxonomia-e-semeadura.md` | 1_2 | Corrigir no `1_2` |
| D11 | Moeda: tabelas misturam `R$ 0` no tier grátis com `US$` nos pagos, num produto para os EUA | 1 | Padronizar em US$ |
| D12 | `1_1` §Parte 4 fixa `model: claude-sonnet-4-6` no template, mas a nota é medida contra 3 modelos | 1_1 | `model` é o modelo **recomendado**; a matriz de avaliação é campo separado (`sdd.md` §Formato) |
