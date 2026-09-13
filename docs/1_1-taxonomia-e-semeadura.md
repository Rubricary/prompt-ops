# Taxonomia e Semeadura do Acervo — Hub Prompt Ops

Define **o que escrever** e **em que ordem**, com base em um levantamento do que já existe público.

Mercado primário: EUA, B2B, em inglês. `locale` e `jurisdiction` são metadados de filtro, não linhas de produto separadas.

> **Precedência.** `prd.md` e `sdd.md` são canônicos onde houver conflito. A Parte 4 abaixo já foi atualizada com o formato congelado.

---

## Parte 1 — O que o scrap encontrou

### Fontes levantadas (13/09/2026)

| Repositório | Itens | Licença |
|---|---|---|
| `f/awesome-chatgpt-prompts` | 2.169 | CC0 1.0 para o conteúdo dos prompts (código em MIT) |
| `linexjlin/GPTs` | 280 | **nenhuma licença declarada — inutilizável** |
| `danielmiessler/Fabric` (patterns) | 255 | MIT |
| `ai-boost/awesome-prompts` | 16 | GPL-3.0 — contaminante, não usar |
| **Total inventariado** | **2.720** | |

Aproveitável para reuso direto: apenas o CC0 e o MIT. Na prática, **~2.420 itens**, e mesmo esses servem como fonte de *ideia*, não de conteúdo — pelas razões abaixo.

### Qualidade do corpus

| Sinal | Itens | % |
|---|---|---|
| Tem variável parametrizada (`{{x}}`) | 508 | 18,7% |
| Define formato de saída | 536 | 19,7% |
| Tem **as duas coisas** | 88 | **3,2%** |
| Menciona configuração (modelo/temperatura) | 92 | 3,4% |
| Menciona caso de teste ou critério de avaliação | 73 | 2,7% |
| Com bateria de testes executável | **0** | **0%** |

Mediana de 1.105 caracteres; 18,9% têm menos de 500. São, em sua maioria, prompts de uma linha do tipo *"act as a…"*.

**Conclusão operacional:** não existe concorrência de acervo. Existe uma pilha de texto sem parâmetro, sem formato, sem configuração e sem prova. O diferencial do doc 1 — *"aqui vem provado"* — está intacto.

### Saturado vs. vazio

Dos 2.720 itens, apenas **419 (15,4%)** sequer nomeiam uma função B2B no título. A distribuição:

| Setor | Itens | % do corpus | Leitura |
|---|---|---|---|
| Engineering | 172 | 6,3% | **Saturado** — e é onde estão os poucos itens decentes |
| Marketing | 55 | 2,0% | Médio, mas genérico e de baixa qualidade |
| Data | 46 | 1,7% | Médio |
| HR | 37 | 1,4% | Médio, quase tudo "resume/cover letter" (lado do candidato, não da empresa) |
| Finance | 31 | 1,1% | **Vazio** na prática — a maioria é auditoria de *código*, não financeira |
| Legal | 30 | 1,1% | **Vazio** |
| Sales | 20 | 0,7% | **Vazio** |
| Product | 12 | 0,4% | **Vazio** |
| Healthcare | 9 | 0,3% | **Vazio** |
| Customer Support | 7 | 0,3% | **O mais vazio de todos** |
| *Sem função B2B no título* | 2.301 | 84,6% | Criativo, consumo, educação, roleplay |

*Margem de erro: a classificação é por palavra-chave no título e tem falsos positivos (ex.: "Accessibility Auditor" caiu em Finance). Os números servem para ordem de grandeza, não para precisão decimal. A direção, porém, é inequívoca.*

### O achado que orienta tudo

**Customer support — o maior gasto corporativo com IA em 2026 — tem 7 prompts públicos em 2.720.** Sales tem 20. Product tem 12.

O que existe em abundância é o que desenvolvedor escreve por diversão (interpretadores, terminais, geradores de regex). O que falta é exatamente o que empresa paga para ter funcionando.

---

## Parte 2 — Taxonomia completa (10 setores)

Estrutura de duas camadas: setor → caso de uso. O caso de uso é o que vira repositório.

**1. Customer Support** — triagem e roteamento de ticket · resposta de primeira linha · pedido de reembolso (dentro e fora da política) · fluxo de cancelamento com retenção · rastreio de pedido · escalonamento para humano · resumo de conversa para handoff · detecção de sentimento e urgência · resposta a avaliação pública negativa · geração de artigo de base de conhecimento a partir de tickets

**2. Sales** — qualificação de lead (BANT/MEDDIC) · cold email de primeiro contato · sequência de follow-up · resumo de call com próximos passos · tratamento de objeção · redação de proposta · pesquisa de conta antes da reunião · higienização de dado de CRM · resumo de pipeline para gestor · e-mail de renovação e upsell

**3. Engineering** — descrição de pull request · code review por convenção do time · mensagem de commit · resumo de incidente · rascunho de post-mortem · documentação de API a partir do código · geração de teste unitário · explicação de código legado · triagem de log de erro · nota de release

**4. Marketing** — brief de SEO · seção de landing page · e-mail de nutrição · post por rede social · adaptação para voz da marca · press release · estudo de caso a partir de entrevista · linha de assunto com variações para teste · resumo de análise competitiva · reaproveitamento de conteúdo entre formatos

**5. Data & Analytics** — SQL a partir de linguagem natural · explicação de query existente · interpretação de variação de métrica · resumo executivo de dashboard · documentação de dicionário de dados · relatório de qualidade de dados · geração de hipótese para teste A/B · narrativa de resultado de experimento

**6. Legal & Compliance** — resumo de contrato · triagem de NDA (semáforo verde/amarelo/vermelho) · extração de cláusula de risco · checagem de política de privacidade · classificação de requisição de titular de dados (DSAR) · resumo de mudança regulatória · resposta a questionário de fornecedor

**7. HR & Recruiting** — descrição de vaga · triagem de currículo por critério · roteiro de entrevista estruturada · resumo de avaliação de candidato · plano de onboarding · rascunho de feedback de desempenho · resposta a política interna · síntese de pesquisa de clima

**8. Finance & Operations** — extração de dados de invoice · categorização de despesa · exceção de reconciliação · resumo de variação orçamentária · triagem de documento de fornecedor · análise de cobrança em atraso · comentário de fechamento mensal

**9. Product** — PRD a partir de problema · história de usuário com critério de aceite · síntese de pesquisa com usuário · agrupamento de feedback por tema · nota de release para cliente · brief de priorização · resumo de sessão de teste de usabilidade

**10. Healthcare Admin & Insurance** — resumo de sinistro · rascunho de autorização prévia · sugestão de código de procedimento · triagem de correspondência de paciente · verificação de elegibilidade · resumo de negativa de cobertura
*(apenas administrativo — nada clínico, nada diagnóstico)*

---

## Parte 3 — Os três setores de lançamento

**Support, Sales, Engineering.** Cerca de 50 prompts, aproximadamente 20 + 15 + 15.

Por quê esses:

| Setor | Vazio no público? | Comprador com cartão? | Caso de teste objetivo? |
|---|---|---|---|
| **Support** | sim, extremo (7 itens) | sim, é o maior orçamento | sim — política é regra verificável |
| **Sales** | sim (20 itens) | sim, orçamento próprio e rápido | médio — precisa de rubrica de juiz |
| **Engineering** | não (172 itens) | sim | sim — saída estruturada, verificável |

Engineering entra apesar de saturado por dois motivos: é o setor de quem visita o site no lançamento, e o que existe público é raso o bastante para a comparação ser favorável. Support é a aposta central — vazio e caro. Sales é o setor onde o comprador decide sozinho, sem passar por compras.

Ficam de fora do lançamento: healthcare (regulado, exige revisor especialista), legal (mesmo motivo), finance, HR, product, marketing, data. Entram por lote conforme o acervo cresce.

---

## Parte 4 — Template por prompt (formato congelado)

Dois arquivos por repositório. Texto em markdown, dado em YAML. Congelado antes do primeiro lote — mudar o esquema com 50 itens prontos custa uma semana de retrabalho.

**`prompt.md`**

```markdown
---
name: refund-request-outside-policy
sector: support
use_case: refund
locale: en-US
jurisdiction: none
license: CC0-1.0
version: 1.0.0
derived_from: null
variables: [customer_name, order_date, policy_window_days, order_total]
model: claude-sonnet-4-6        # modelo recomendado em produção
temperature: 0.2
eval_matrix: [modelo-a, modelo-b, modelo-c]   # contra quem a nota é medida
---

You are a customer support agent for {{company_name}}...
```

`sector` e `use_case` viram slug de diretório e de URL: valores restritos à lista fechada da Parte 2, validados no pipeline. Senão aparecem `support` e `customer-support` no mesmo acervo.

**`evals.yaml`** — 8 a 15 casos por prompt. Chaves em inglês, porque o produto é em inglês:

```yaml
- id: outside-window-polite-refusal
  input: "I bought this 40 days ago and want my money back"
  criteria: "Must deny the refund citing the 30-day window, offer store credit, and stay courteous"
  type: judge        # judge | exact | regex | json_schema
  weight: 2
```

O `2-hub-de-evals.md` usa as mesmas chaves em português (`entrada`, `criterio`, `tipo`, `peso`). O canônico é este; os outros hubs adotam o inglês se forem construídos.

A **nota não vive no repositório** — é função do modelo, não do prompt, e geraria commit espúrio a cada re-avaliação. Ela é resultado derivado, guardado no índice (`sdd.md` §6).

Regra: **nenhum prompt entra no acervo sem `evals.yaml`.** É o que separa o hub de mais uma lista de prompts.

---

## Parte 5 — Método de produção

Lotes de 10 a 15 por sessão: eu escrevo o lote completo, você lê e marca o que refazer, eu reviso, fecha o lote. Cerca de 4 sessões cobrem os 50 do lançamento; ~20 sessões chegam aos 300.

O scrap entra **só** como checagem de cobertura: antes de fechar um setor, confiro se a lista de casos de uso cobre o que aparece nos 2.420 itens aproveitáveis, para não deixar buraco óbvio.

**Curadoria e selo são da plataforma**, não pessoais. Nos EUA o nome que vende é o do hub: *"verified · tested against 3 models"*. Assinatura individual de autor entra na fase 2, quando especialistas externos começarem a publicar — aí o crédito nominal é justamente o que os atrai.
