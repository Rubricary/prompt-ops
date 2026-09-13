# Hub 1 — Prompt Ops
 
## Glossário antes de tudo
 
- **Prompt**: o texto de instrução que uma empresa envia para a IA. Ex.: *"Você é o atendente da Loja X. Nunca prometa reembolso acima de R$ 200."*
- **System prompt**: o prompt fixo, invisível ao usuário final, que define o comportamento do robô. É ele que vive em produção.
- **Produção**: o sistema que os clientes reais estão usando agora (oposto de "teste").
- **Versionamento**: guardar cada alteração de um arquivo com data, autor e motivo, podendo voltar atrás.
- **Diff**: a visualização "o que mudou entre a versão antiga e a nova", linha por linha.
- **Rollback**: voltar para a versão anterior quando a nova deu problema.
- **SDK**: uma bibliotequinha que o programador instala no projeto dele para conversar com o seu serviço.
- **Runtime**: o momento em que o programa está rodando de verdade (oposto de "na hora de programar").
## A dor
 
Hoje, numa empresa média, o system prompt está escrito dentro do código, ou pior: numa variável de ambiente, num painel administrativo, ou colado numa planilha que o time de atendimento edita. Consequências reais:
 
1. Alguém muda uma frase numa sexta-feira, a IA começa a responder errado no sábado e ninguém sabe o que mudou.
2. Não existe aprovação: quem escreve o prompt normalmente não é programador, então ele edita direto, sem revisão.
3. Não existe histórico: não dá para responder "qual prompt estava no ar quando esse cliente recebeu essa resposta?" — o que é um problema jurídico sério.
4. Não dá para testar duas versões e comparar.
Prompt virou código de produção, mas não tem nenhuma das proteções que o código tem.
 
## O que o hub é
 
Um lugar onde o prompt é um arquivo de texto versionado, com fluxo de revisão, e uma API que entrega o prompt certo para o sistema da empresa em tempo real.
 
O prompt deixa de estar "dentro" do software e passa a ser um recurso externo com endereço e versão:
 
```
minhaempresa/atendimento-checkout@v1.4
```
 
## Fluxo — como uma empresa usaria
 
**Entrada (setup, uma vez):**
 
1. A empresa cria uma organização no hub.
2. Cria um "prompt" chamado `atendimento-checkout`. Isso gera um repositório com um arquivo `prompt.md` e um `config.yaml` (modelo usado, temperatura, variáveis esperadas).
3. O programador instala o SDK e troca o texto que estava no código por uma chamada:
   `prompt = hub.get("atendimento-checkout", version="production")`
**Dia a dia (o ciclo que vende o produto):**
 
1. A pessoa de atendimento (não programadora) abre o prompt no site e edita o texto no navegador.
2. Ao salvar, o hub não publica: cria uma **proposta de mudança**, mostrando o diff lado a lado.
3. O hub roda automaticamente a bateria de testes daquele prompt (ver Hub 2) e mostra: "12 de 14 casos passaram; piorou em 'pedido de reembolso'".
4. Um aprovador recebe notificação, olha o diff e o resultado dos testes, e aprova.
5. Ao aprovar, o hub marca a nova versão como `production`. O SDK, que consulta a cada X minutos, passa a entregar a nova versão — **sem novo deploy do software**.
6. Se der problema, um botão "reverter" volta para a versão anterior em segundos.
Esse ponto 5 é o argumento comercial mais forte: hoje mudar uma vírgula no prompt exige um ciclo completo de programação e publicação, que leva horas ou dias.
 
## Funcionalidades exclusivas (o que o GitHub não faz)
 
- **Editor para não-programador**: campo de texto com destaque das variáveis (`{{nome_cliente}}`), sem YAML, sem terminal.
- **Teste automático no momento da mudança**: a proposta já nasce com nota.
- **Comparador A/B**: publica a versão nova para 10% do tráfego e mostra as duas taxas de sucesso lado a lado.
- **Trilha de auditoria**: "em 14/03 às 15:22, a resposta ao cliente #8891 foi gerada com a v1.3" — exigência de compliance, LGPD e setor financeiro.
- **Detector de regressão de custo**: avisa se o novo prompt ficou 40% mais longo e vai aumentar a conta de tokens.
- **Biblioteca pública de prompts testados**, por setor, que qualquer um pode copiar para dentro da sua organização.
## Monetização
 
| Camada | Preço-alvo | O que entrega |
|---|---|---|
| Grátis | R$ 0 | Prompts públicos, 1 usuário, sem aprovação |
| Time | US$ 29/usuário/mês | Prompts privados, aprovação, histórico ilimitado |
| Empresa | US$ 1.500+/mês | SSO, trilha de auditoria exportável, ambiente isolado, SLA |
| Uso | US$ por 1.000 chamadas de API acima da cota | Entrega em runtime, A/B, cache |
 
O dinheiro de verdade está na camada Empresa: auditoria e controle de acesso são itens que passam por compliance e saem do orçamento de risco, não do de ferramentas.
 
## Como não parecer vazio no lançamento
 
O hub é útil para uma empresa mesmo que ninguém mais o use — é ferramenta interna, não rede social. Ainda assim, a vitrine importa:
 
1. **Semeie 150–300 prompts públicos** de qualidade, organizados por caso de uso (atendimento, extração de dados de nota fiscal, classificação de ticket, resumo de reunião). Escreva você mesmo, ou adapte o que já está espalhado em repositórios com licença permissiva — sempre com crédito.
2. Cada prompt público entra **com a bateria de testes junto** e com a nota contra 3 modelos. É isso que diferencia de uma lista de prompts qualquer: aqui vem provado.
3. **Importador de um clique**: "cole seu prompt atual aqui e veja a nota dele". A pessoa entra para usar a ferramenta, e o acervo se enche sozinho com o que ela publica.
4. Publique semanalmente um "estudo de caso": pegue um prompt famoso, mostre o diff de uma melhoria e o ganho medido. Esse é o conteúdo que traz o público certo.
 