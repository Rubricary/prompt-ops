# Como Montar um Hub na Prática (o modelo Hugging Face)

Documento técnico comum aos sete hubs. A arquitetura é a mesma; muda só o que é armazenado e a camada de inteligência por cima.

> **Precedência.** Para o Prompt Ops (Rubricary), `prd.md` e `sdd.md` são canônicos onde houver conflito com este documento. As decisões já refletidas aqui: hospedagem fora da Vercel (Camada 1) e git bare estático como opção padrão de MVP (Camada 5).

---

## Glossário antes de tudo

- **Git**: o sistema de versionamento usado pelo mundo inteiro. Guarda o histórico completo de arquivos, quem mudou o quê e quando.
- **Repositório (repo)**: a pasta versionada. Cada item do seu hub será um repositório.
- **Repositório *bare***: um repositório sem pasta de trabalho, só com o histórico. É o formato usado no servidor, porque ninguém edita arquivos lá — só envia e recebe.
- **Push / clone**: enviar suas alterações / baixar uma cópia.
- **Commit**: uma alteração registrada, com autor, data e mensagem.
- **Smart HTTP**: o modo como o git trafega por HTTPS. Quem serve isso é um programa chamado `git-http-backend`, que já vem com o git.
- **Hook**: um script que o git dispara automaticamente em certos momentos (ex.: logo após receber um envio).
- **Object storage / bucket**: armazenamento de arquivos na nuvem cobrado por GB (S3, Cloudflare R2). Barato e infinito.
- **Egress**: o custo de tráfego de saída, cobrado quando alguém baixa dados da sua nuvem. É o que quebra hubs de arquivo grande — e o motivo de escolher o Cloudflare R2, que não cobra.
- **CDN**: rede que espalha cópias dos arquivos pelo mundo para entregar rápido e barato.
- **Git LFS**: extensão do git para arquivos grandes (modelos, vídeos). **Você não vai precisar**, e essa é a sua vantagem: só texto.

---

## Por que usar git por trás

Você não constrói versionamento. É problema resolvido há vinte anos, e:

- O público-alvo já sabe usar (`git push` é reflexo).
- Você ganha de graça: histórico, diff, branches, propostas de mudança, autoria, assinatura criptográfica, integridade por hash.
- O item do seu hub passa a ser clonável, o que dá liberdade ao usuário — e liberdade é o que faz as pessoas confiarem o acervo a você. O Hugging Face cresceu assim: cada modelo é literalmente um repositório git.
- Migrar depois para git, tendo começado com banco de dados, é um projeto doloroso. O contrário nunca é preciso.

**Quando não usar git:** se o item for uma linha de dados que muda mil vezes por dia (uma métrica, um log). Aí é banco. Nos sete casos deste documento, o item é um documento de texto que muda poucas vezes por semana — git é o encaixe exato.

---

## Arquitetura em camadas

```
┌─────────────────────────────────────────────┐
│ 1. Site (Next.js na Vercel)                 │  vitrine, busca, editor,
│                                             │  painel, cobrança
├─────────────────────────────────────────────┤
│ 2. API + CLI                                │  npx seuhub push/add
├─────────────────────────────────────────────┤
│ 3. Índice (Postgres + busca textual)        │  METADADOS apenas
├─────────────────────────────────────────────┤
│ 4. Camada de inteligência (workers)         │  ← SEU DIFERENCIAL
│    validação, nota, análise de segurança    │
├─────────────────────────────────────────────┤
│ 5. Servidor git (git-http-backend)          │  push/clone
├─────────────────────────────────────────────┤
│ 6. Armazenamento (bucket R2)                │  repositórios bare
└─────────────────────────────────────────────┘
```

A regra de ouro: **o conteúdo nunca entra no banco de dados**. O banco guarda nome, descrição, autor, tags, contadores, notas e o hash do commit atual. O texto vive no git, no bucket. É isso que mantém seu custo de infraestrutura desprezível.

---

## Camada 6 — Armazenamento

- Um repositório bare por item, em `s3://hub/repos/{org}/{nome}.git`.
- **Cloudflare R2**: preço por GB semelhante ao S3, mas **sem custo de egress**. Como seu produto é distribuir arquivos, isso muda o negócio, não só a conta.
- Texto puro comprime a 10–20% do tamanho. Cem mil itens de 50 KB dão ~5 GB brutos. Custo mensal: alguns dólares. Este é o motivo de você ter escolhido texto.
- Estratégia prática: disco anexado (volume) para os repositórios ativos, sincronizado com o bucket, que é a verdade durável. Git em bucket direto é possível, mas exige cuidado extra com desempenho.

## Camada 5 — Servidor git

Não escreva um servidor git. Use o que existe:

**Opção A:** um contêiner rodando `nginx` + `fcgiwrap` + `git-http-backend`. São ~30 linhas de configuração. Autenticação por token via um endpoint de autorização que consulta sua API. Roda numa VPS de US$ 20.

**Opção B:** Gitea ou Forgejo (servidores git prontos, de código aberto) rodando escondidos atrás da sua aplicação. Você ganha permissões, usuários e propostas de mudança prontos, e usa a API deles a partir do seu site. Reduz meses de trabalho; o custo é depender do modelo de dados deles — e um custo fixo de VPS desde o dia 1.

**Opção C:** usar o próprio GitHub como armazenamento no MVP, via API, e migrar depois. Serve para validar em semanas, mas amarra você a limites de taxa e some com o seu controle. Use só se o objetivo for testar a ideia rápido.

**Opção D (recomendada para o MVP): nenhum servidor.** O protocolo *dumb HTTP* do git precisa apenas de arquivos estáticos servidos por HTTPS: `info/refs`, `objects/info/packs` e os packfiles, todos gerados por `git update-server-info`. Um repositório bare sincronizado para o bucket, atrás de domínio próprio, faz `git clone` funcionar sem nenhum processo rodando e com egress zero. A escrita não passa por git: a API e o worker que já roda a análise fazem o commit e a sincronia. Exige `git gc` antes de cada sincronia — sem empacotar, o clone vira milhares de requisições de objeto solto.

Recomendação: **D enquanto não houver receita, B quando houver cliente pagante (traz `git push` direto e propostas de mudança prontas), A quando a escala justificar**. A migração de D para B é `git push --mirror` — o formato nunca deixou de ser git.

A opção D só não serve se o seu hub precisar de `git push` direto do usuário desde o começo. Onde houver fluxo de aprovação (toda mudança entra por proposta), ela não é limitação.

O gancho essencial é o `post-receive`: assim que alguém envia conteúdo, esse hook chama sua API, que dispara a reindexação e a análise. É a ponte entre o git e o produto.

## Camada 4 — A camada de inteligência

**É aqui que está o negócio.** As camadas 5 e 6 são infraestrutura comoditizada; qualquer um monta. O que ninguém copia rápido é o julgamento automático sobre o conteúdo:

| Hub | O que a camada faz |
|---|---|
| Prompt Ops | roda a suite de testes no diff, mede custo, compara versões |
| Evals | executa suites contra N modelos, monta placar, detecta contaminação |
| Skills | mede acionamento, detecta conflito, analisa risco de segurança |
| MCP | analisa permissões, detecta envenenamento, testa vitalidade |
| llms.txt | rastreia, valida, mede deriva, testa percepção nos modelos |
| Legislação | ingere, estrutura, detecta alteração, calcula vigência |
| i18n | casa com a memória, verifica consistência, previne quebra |

Implementação: fila de trabalhos (Redis, ou a fila da própria Cloudflare) + workers. O resultado vira metadado no índice e aparece na vitrine.

## Camada 3 — Índice

- **Postgres** (Supabase, Neon) com busca textual nativa. Não precisa de Elasticsearch no começo, e provavelmente nunca.
- Tabelas: `orgs`, `repos`, `versoes`, `analises`, `usuarios`, `permissoes`, `eventos`.
- Busca semântica (por significado, não por palavra) é um adicional posterior: guarde os vetores numa extensão do Postgres, não num serviço novo.

## Camada 2 — API e CLI

- API REST simples. Autenticação por token com escopo.
- A CLI é obrigatória e precisa ser boa: `npx seuhub push` tem que funcionar de primeira, sem instalação global. Publique no npm.
- Por baixo, a CLI faz um `git push` comum. Nada de protocolo proprietário.
- Ofereça também **um endpoint de leitura sem autenticação** para os itens públicos. É assim que agentes e outras ferramentas passam a depender de você — e dependência é fosso.

## Camada 1 — Site

- Next.js em Cloudflare Workers (via `@opennextjs/cloudflare`). O conteúdo é texto pequeno e majoritariamente público: cacheie tudo agressivamente na borda.
- **Não use a Vercel no plano Hobby**: ele proíbe uso comercial de forma explícita, desde o primeiro dia e não a partir de certo faturamento. Cloudflare Workers e Supabase permitem uso comercial no plano gratuito.
- Páginas obrigatórias: busca, ficha do item, diff entre versões, perfil de autor/organização, painel da organização, cobrança.
- A ficha do item é o produto visível. Ela precisa responder em cinco segundos: o que é, se é confiável, quem mantém, como instalar.

---

## Ordem de construção (12 semanas, uma pessoa)

| Semanas | Entrega |
|---|---|
| 1–2 | Modelo de dados, autenticação, repositório bare no bucket (opção D), clone funcionando |
| 3–4 | CLI (`push`, `add`, `login`) e API de leitura |
| 5–6 | Site: busca, ficha do item, perfil. Indexação via hook |
| 7–8 | **Camada de inteligência** — a análise específica do seu hub |
| 9 | Semeadura do acervo (ver o plano de cada documento) |
| 10 | Organizações, privado, permissões |
| 11 | Cobrança (Stripe) e limites por plano |
| 12 | Lançamento com o relatório/ranking de tração |

Observe que a semeadura vem antes da cobrança. Ninguém paga por prateleira vazia.

---

## Princípio geral sobre o acervo inicial

Os sete hubs se dividem em dois tipos, e a estratégia é diferente:

**Acervo gerável por máquina** (llms.txt, MCP, legislação, skills públicas, i18n): você nasce cheio, rastreando ou importando o que já é público, com crédito à origem. O acervo não é seu diferencial — **a camada de julgamento sobre ele é**. Lance com um relatório inédito sobre esse acervo; é o que gera imprensa e backlink.

**Acervo que precisa ser criado** (evals, prompts): não há atalho. Produza você mesmo 10 a 50 itens excelentes e trate isso como o produto, não como preparação. Complemente com economia de reputação: especialista assina o que escreve e leva o crédito público.

Em ambos os casos, a alavanca mais eficiente é a mesma: **uma ferramenta grátis, sem cadastro, que entrega valor na hora** (validador, diagnóstico, nota). Ela traz o visitante, prova a competência e o converte em publicador. Foi o padrão de todo registro que venceu.
