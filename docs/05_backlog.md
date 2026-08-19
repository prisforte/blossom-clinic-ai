# 05 — Backlog

**Estado:** Ativo — primeira versão, com base no briefing operacional de
19-08-2026 e na inspeção do ficheiro
`MM_Ecommerce_Performance_Tracking_2026.xlsx`.

Convenção de estado: `bloqueado` (depende de acesso/decisão externa) ·
`por validar` (dono humano tem de confirmar) · `pronto para implementar`
(claro o suficiente para virar código/config) · `concluído`.

---

## 0. CRÍTICO — a Redicom tem 3 valores de faturação diferentes para o mesmo dia

**Encontrado em:** 20-08-2026, durante extração de dados na Redicom para
18-08-2026.

| Relatório (Redicom) | Faturação 18-08-2026 |
|---|---:|
| Análise da Receita Diária | 9.790,96 € |
| Resumo de Encomendas | 9.229,89 € |
| Vendas Globais | 9.148,00 € |
| *Sheet manual "MELLMAK" (usada em todas as análises anteriores)* | *9.311,00 €* |

**Nenhum dos 3 relatórios da Redicom bate com o valor que temos vindo a
tratar como "faturação oficial" em todas as análises de fecho de dia até
agora.** Isto significa que:

- A definição de "faturação oficial" na Regra 1 da `00_system_constitution.md`
  ainda não está resolvida ao nível de qual relatório da Redicom usar.
- A sheet manual "MELLMAK" parece usar um 4º critério, ainda não identificado.
- **Toda a "Confiança dos dados" reportada como "Média" nas análises de
  18-08 e no briefing operacional deve ser lida como "Baixa"** até isto
  ficar resolvido — não é um gap de fonte secundária, é uma ambiguidade na
  própria fonte oficial.

**Atualização 20-08-2026 — reconciliado a 50%.** Extração linha-a-linha da
Redicom (encomendas de 1-18 agosto, nível de item) permitiu explicar 2 dos
4 valores para 18-08:

| Relatório | Valor | Metodologia encontrada nos dados brutos | Confirmado? |
|---|---:|---|---|
| Resumo de Encomendas | 9.229,89 € | `SUM(EncomendaValorTotal)` por encomenda única, Venda+Troca, agrupado pela **data de pagamento** (`EncomendaPaga`) | ✅ exato, ao cêntimo |
| Vendas Globais | 9.148,00 € | `SUM(EncomendaValorTotal)` por encomenda única, só Tipo="Venda", agrupado pela **data da encomenda** (`EncomendaData`) | ≈ quase exato (calculado: 9.149,07€, diferença de 1,07€) |
| Análise da Receita Diária | 9.790,96 € | Testado por data de fatura e por data de pagamento, com/sem trocas — nenhuma combinação bate | ❌ ainda não reconciliado |
| Sheet manual "MELLMAK" | 9.311,00 € | Nenhuma combinação testada bate | ❌ ainda não reconciliado |

**Causa raiz identificada para metade da divergência:** não é um erro de
dados — é uma diferença de **critério de data** (data da encomenda vs. data
de pagamento) e de **inclusão de trocas**. Isto tem de virar uma decisão
formal de política: qual data é a "oficial" para reporting diário de
faturação (impacta `02_kpi_master.md` e todo o Decision Engine).

**Ação em curso:**
1. Perguntar à Redicom/quem gere o sistema: que campo de data e que
   filtros (loja física incluída? tipo de encomenda?) usa "Análise da
   Receita Diária" — é o único ainda sem hipótese.
2. Perguntar a quem preenche a sheet manual "MELLMAK" de onde tira o valor
   — pode ser um export manual de um destes relatórios a uma hora de corte
   diferente do dia.
3. Confirmar com o financeiro/gestão qual data-base (encomenda vs.
   pagamento) deve ser a oficial para a Regra 1 da constituição.

**Dono:** financeiro/gestão Mellmak (definição do critério oficial) +
pessoa que preenche a sheet manual (origem do valor atual).

---

## A. Pontos por validar (origem: briefing operacional, secção 10)

| # | Item | Estado | Dono |
|---|---|---|---|
| 1 | Hora exata em que os artigos de 30% foram repostos na Mellmak (11-08) | por validar | operações/compras |
| 2 | Lista final e validada de artigos a 30%, 50% e 70% | por validar | compras |
| 3 | Confirmação de que todos os URLs de campanhas antigas continuam funcionais | **respondido 20-08** — 4/4 anúncios ativos testados carregam e têm produto (ver secção E) | marketing/dev |
| 4 | Confirmação de que o catálogo Meta e o catálogo Google estão atualizados | **parcialmente respondido, negativo** — ver secção E | marketing |
| 5 | Definição final da política de portes em Portugal (2,99€ vs 3,99€) e limite de portes grátis | por validar | comercial |
| 6 | Reconciliação de devoluções, sem stock e cancelamentos | bloqueado | Redicom (fonte oficial, ver `00_system_constitution.md`) |
| 7 | Correção das discrepâncias entre atribuição Redicom/GA4 e faturação oficial | bloqueado — causa provável já identificada em auditoria de junho (ver `01_data_sources.md`), execução do lado Redicom por confirmar | ver item B.2 abaixo |
| 8 | Teto de investimento de 10% (Regra Nuno Forte) fora de época de descontos: existe? qual? | por validar | Nuno Forte |
| 9 | Limite de 10% é sobre o mês acumulado ou sobre o período da campanha? | por validar | Nuno Forte |
| 10 | Efeito isolado da alteração de portes (2,99€) sobre conversão/ticket médio | por validar | análise futura, precisa de mais dados pós-mudança |
| 11 | Estado de execução do plano de estabilização de tracking preparado na auditoria de junho de 2026 (remoção do GA4 nativo da Redicom, revisão do Purchase, normalização de eventos) — foi executado, parcialmente, ou ainda não? | **por validar, urgente** | quem geriu a auditoria de junho + Redicom |
| 12 | Consent Mode a assumir consentimento aceite por defeito (auditoria de junho, item 8) — corrigido? | por validar, possível risco de conformidade | dev/marketing |
| 13 | Configuração cross-domain entre plataformas (auditoria de junho, item 9) — revista? | por validar | dev |

## B. Data Health — gaps encontrados no workbook mestre

Achados da inspeção de `MM_Ecommerce_Performance_Tracking_2026.xlsx`
(41 folhas). Relevantes porque explicam *porque* várias métricas oficiais
não estão disponíveis para julho/agosto de 2026 apesar de existirem como
colunas definidas no sistema.

| # | Gap | Detalhe | Impacto |
|---|---|---|---|
| B.1 | Conectores automáticos de custo de anúncios parados | `DAILY_CHANNEL_META_ADS`, `DAILY_CHANNEL_GOOGLE_ADS`, `DAILY_BUDGET_TRACKET`, `CHANNEL_RESUME` não têm dados depois de 31-12-2024 | A fórmula oficial de "% Budget gasto" no `TRADING TRACKING 2026` devolve 0% em todo o mês de agosto — o valor real só existe na sheet manual diária |
| B.2 | Folha "2026 Custos publicidade" desatualizada | Últimos dados em 03-06-2026; sem julho/agosto | Mesma causa raiz do B.1 para o período mais recente |
| B.3 | "Taxa abandono carrinho" nunca preenchida no workbook mestre | Coluna existe na estrutura mestre (`TRADING TRACKING 2026`) desde sempre, mas todos os meses têm valor vazio | **Resolvido em 19-08-2026** — fonte encontrada: dashboard nativo da Redicom Commerce Cloud (`mellmak.redicom.cloud`, painel "Recuperação de Carrinho") reporta a taxa em tempo real (5% no momento da verificação). A regra do Decision Engine deixa de estar bloqueada — ver `04_decision_engine.md` DE-3. O campo do workbook mestre continua vazio e não deve ser usado. |
| B.4 | "CAMPAIGNS 2026" não ligado | Coluna de campanha por dia no workbook mestre está vazia para 2026, apesar de a campanha "SALES" estar ativa e registada na folha `CAMPANHAS - META & GOOGLE 2026` | O gatilho de "época de descontos" da Regra Nuno Forte não pode assumir que este campo está fiável — usar antes a folha de campanhas diretamente |
| B.5 | Gap de reconciliação faturação por família | Ex.: 18-08, faturação por família (Mulher+Homem+Criança+Unisexo) soma 8.466,48€ vs. faturação total oficial 9.311,00€ — gap de 844,52€ | Por resolver antes de usar dados por família/categoria em decisões de merchandising |
| B.6 | `Custos publicidade` (folha legada) com erros `#REF!` | Referências quebradas na folha antiga de custos | Não usar como fonte; folha a descontinuar |
| B.7 | Valor de compra (purchase value) do Google Ads/GA4 subavaliado na Mellmak | Ticket médio implícito Google ≈26€/compra vs. Meta ≈80€/compra (mesma loja, mesmo período); na Forte Store os dois batem certo (≈67€ vs. ≈67€). Aponta para configuração incorreta do parâmetro `value` no evento de compra do GA4/Google Ads da Mellmak, não para diferença real de eficiência | "Valor conv./custo" do Google para a Mellmak (10,02 em julho, 13,87 em 1-15 ago) está artificialmente inflacionado — não usar em decisões nem publicar externamente sem esta ressalva (ver Regra 5 da constituição). **Decisão de comunicação (20-08):** internamente mantém-se registado como suspeita forte de configuração incorreta (evidência acima); em comunicação externa a um terceiro (ver secção F), optou-se por "em validação interna" em vez de afirmar o diagnóstico como confirmado. **Atualização 20-08 (auditoria de junho encontrada):** este achado NÃO é um problema novo — corresponde diretamente aos itens 6 ("Purchase disparado no momento errado") e 7 ("Enhanced Conversions mal configuradas, trigger incorreto") de uma auditoria técnica interna de junho de 2026, ainda sem confirmação de correção do lado da Redicom. Ver `01_data_sources.md`. |
| B.8 | Sheets manuais "CALCULOS FS E MM" (Meta+Google) não batem com a própria célula "TOTAL INVESTIMENTO" | Julho: Forte Store soma diária 21.385,33€ vs. célula 20.124,21€ (~6%); Mellmak soma diária 19.897,49€ vs. célula 18.954,01€ (~5%). Padrão idêntico nas duas sheets — suspeita de fórmula ligada a intervalo desatualizado, como já visto no workbook mestre (B.1/B.2) | Confiar na soma diária (validada contra Ads Manager, ver C.16) em vez da célula de total até se confirmar a causa |

## C. Backlog de implementação (Neptune COS)

| # | Item | Fase | Estado |
|---|---|---|---|
| C.1 | Conector Redicom (mock) | Fase 5 | pronto para implementar |
| C.2 | Conector GA4 (mock) | Fase 5 | pronto para implementar |
| C.3 | Conector Meta Ads (mock) | Fase 5 | pronto para implementar |
| C.4 | Conector Google Ads (mock) | Fase 5 | pronto para implementar |
| C.5 | KPI Master completo (fórmulas + fontes) | Fase 3 | pendente |
| C.6 | Regra dupla meta (formal 254.869,90€ vs. operacional 500.000€) no Decision Engine | Fase 6 | pronto para implementar (regra já definida em `03_business_rules.md` §1) |
| C.7 | Regra Nuno Forte (teto 10% em desconto) no Decision Engine | Fase 6 | pronto para implementar, com nota de fiabilidade (ver B.1/B.2) |
| C.8 | Gatilho "época de descontos" lido de calendário de campanhas | Fase 6 | pronto para implementar como config/dados, ver `03_business_rules.md` §2 |
| C.9 | Data Health: sinalizar fontes automáticas desatualizadas (B.1, B.2, B.6) | Fase 4 (UI) + Fase 6 (regra) | pronto para implementar |
| C.10 | Data Health: reconciliação faturação total vs. por família (B.5) | Fase 4/6 | pronto para implementar como verificação, não como bloqueio |
| C.11 | Métrica "vendas por categoria prioritária" (T-shirts, malas, sapatilhas, jeans) | Fase 3/4 | pendente — precisa de fonte estruturada, hoje só existe no briefing como números pontuais |
| C.12 | Auditoria de catálogo/stock/preço/URL, catálogos Meta/Google, links de campanha | fora do Neptune COS | **concluído parcialmente em 20-08**, via sessão do utilizador com acesso direto (ver secção E) — Google Merchant Center continua bloqueado por falta de permissão |
| C.13 | Bloquear propostas de aumento de investimento até resolver stock crítico nos destinos de anúncios ativos (secção E.3) | Fase 6 (Decision Engine) | pronto para implementar — condição objetiva: ativo se ≥1 anúncio ativo aponta para produto com ≤2 unidades disponíveis |
| C.14 | Corrigir/investigar 4.291 produtos com erro no feed "% de Descontos" (Meta) | fora do Neptune COS | bloqueado — a executar por quem gere o feed/catálogo Meta |
| C.15 | Obter acesso ao Google Merchant Center 2107046214 (Mellmak) na conta usada para auditoria | fora do Neptune COS | bloqueado — pedir acesso a quem administra a conta Google Ads/Merchant Center da Mellmak |
| C.16 | Corrigir o parâmetro `value` do evento de compra GA4/Google Ads na Mellmak (B.7) | fora do Neptune COS | bloqueado — a executar por quem gere o tracking/GTM da Mellmak; validar depois nas duas plataformas |

## E. Auditoria de catálogo e anúncios (20-08-2026, via acesso direto do utilizador)

Fonte: extração direta em Meta Ads Manager, Meta Ad Library, Google Ads/
Windsor.ai e Google Merchant Center. Só leitura — nada foi alterado
(campanhas, orçamentos ou catálogos).

**E.1 — Investimento diário (validação cruzada com a sheet manual):**
confirma que os valores de investimento da sheet manual são fiáveis —
Google bate ao cêntimo, Meta tem um desvio de 0,2% (atribuição). Ao
contrário da faturação (item 0), aqui não há problema de fonte.

**E.2 — Catálogo Meta não corresponde às listas de desconto nomeadas.**
O conjunto de produtos em uso nos anúncios ("30% e 50% INV 22 21 20 19 18
17 16", 10,5 mil produtos) foi criado em 18-11-2023 — não é a "Lista 30%
Mellmak" / "Lista 50% Mellmak" (11-8 a 6-9-2026) referida em
`03_business_rules.md` §3. Não é possível confirmar que os descontos atuais
estão refletidos no catálogo publicitário. Feed complementar de descontos
tem 4.291 produtos com erro de carregamento (de um total de 5.133).

**E.3 — Risco de stock em anúncios ativos.** Dos 4 anúncios ativos
testados com destino a páginas de saldos, 3 têm ≤2 unidades disponíveis no
tamanho existente ou a maioria dos tamanhos esgotados. Há investimento
ativo a gerar tráfego pago para produtos quase esgotados.

**E.4 — Divergência criativo vs. página.** Os 4 criativos testados
anunciam "até 70%"; as 4 páginas de destino estão todas a -50% (um deles
até tem a inconsistência dentro do próprio anúncio: título 70%, descrição
50%). Nenhum dos destinos testados chega a 70%.

**E.5 — Google Merchant Center inacessível.** A conta Google usada na
auditoria só tem acesso ao MC da Forte Store (131851149), não ao da
Mellmak (2107046214, o associado à campanha ativa). Catálogo Google não
pôde ser auditado — ver C.15.

**E.6 — Conector Meta sem API disponível.** O Windsor.ai só tem Google Ads
ligado para esta conta; os dados Meta vieram da UI do Ads Manager
(relatório manual, não persistente). Confirma que o conector Meta em
`/src/connectors/meta` (Fase 5) terá de prever, para já, um modo de
importação manual/CSV além (ou em vez) de uma API direta.

## F. Auditoria do funil de checkout (20-08-2026, teste direto no site)

Fonte: percurso real como visitante anónimo em mellmak.com, com inspeção
de DOM/CSS/rede/consola. Nenhuma compra concluída, nenhuma definição
alterada. Testado num único dia de dados de funil (2026-08-19, a decorrer)
— os números absolutos não devem ser generalizados sem alargar o
intervalo.

**F.0 — O mapeamento do funil do dashboard não corresponde ao fluxo real
do site.** A legenda mostra "Adicionou ao Carrinho → Página de Pagamento →
Resumo do Carrinho → Confirmação Encomenda"; o fluxo real é "Resumo →
Identificação → Pagamento → Imprimir Comprovativo". O passo
"Identificação" (onde estão as piores fricções, ver F.4/F.5) não aparece
como passo próprio. **Antes de usar estes números para decisões,
confirmar com a Redicom a que evento corresponde cada barra.**

**F.1 — Prioridade #1, esforço baixo, sem dependência externa: zona morta
de layout entre 768px e 1130px.** `header`, `.containerFull` e `.steps`
têm `min-width: 1130px` fixo; o breakpoint mobile só entra a 767px. Nessa
faixa (iPad horizontal, portáteis Windows com escala 125-150%, janela de
browser não maximizada), **os preços e o total ficam fora do ecrã** — o
cliente é convidado a avançar sem ver quanto vai pagar. Confirmado
também na Forte Store (mesma plataforma Redicom) — correção beneficia as
duas lojas.

**F.2 — Falha silenciosa na submissão de "Identificação" (guest
checkout).** Em 2 de 3 tentativas, o POST a `actions_registo.php` ficou
pendente e o cliente foi redirecionado para a homepage, perdendo nome,
morada, código postal e localidade — sem mensagem de erro. Carrinho
mantém-se, progresso não. Variável exata do gatilho ainda não isolada —
precisa de log do lado servidor para medir frequência real.

**F.3 — URLs intermédios do checkout ligados à sessão.** Reentrar num
URL de checkout válido minutos antes redireciona para a homepage sem
aviso — afeta botão "voltar", refresh, novo separador ou link guardado.
Combinado com F.2, qualquer desvio ao caminho linear perfeito custa o
checkout inteiro.

**F.4 — Passo de Identificação funciona como muro de login, com registo
pré-selecionado.** "Já é Cliente Registado?" ocupa a posição primária de
leitura; o guest checkout está na coluna secundária; "Pretende
registar-se?" vem com "Sim" pré-selecionado — o cliente tem de reparar e
desmarcar ativamente.

**F.5 — Passo de Identificação não mostra preço nem total, e o botão diz
"Finalizar".** Cliente introduz morada às cegas quanto ao valor, e
"Finalizar" sugere fim de processo quando ainda faltam 2 passos e o
método de pagamento.

**F.6 — Passo de Pagamento pede 28 campos, repetindo toda a morada** já
pedida no passo anterior.

**F.7 — Botão "INSERIR UM DESCONTO" duplicado com a secção "Código
promocional"** no passo de Resumo, ambos com mais destaque visual que o
total — padrão clássico que manda o cliente sair do site à procura de
cupão. Não existe na Forte Store (mesma plataforma) — é específico da
Mellmak.

**F.8 — Mini-carrinho sem portes/previsão de entrega e sem CTA direto
para pagamento**, e ausência de limiar de portes grátis (a alavanca mais
direta para subir ticket médio e reduzir o atrito dos 2,99€ num carrinho
de 20€). Nota positiva: os portes E o total já aparecem corretamente no
passo 1 do checkout em si — não é um caso de "portes escondidos até ao
fim".

**F.9 — Widget "Assistente IA" sobrepõe-se a campos obrigatórios** do
passo de Identificação, mais grave na zona morta de F.1.

**F.10 — Erro de JavaScript no checkout pode estar a subcontar eventos
do funil.** `ReferenceError: $ is not defined` disparado 6x no contentor
GTM-WW3NKZ9 (tag dependente de jQuery em falta) durante a submissão do
passo de Identificação; chamadas ao Google Analytics/Ads em paralelo
devolveram 503. Reforça F.0 — as barras do dashboard podem não refletir
o funil real. Não afeta o cliente diretamente, afeta a fiabilidade dos
números usados para decidir.

**F.11 — Explica a regra "recuperação de carrinho < 12%" do Decision
Engine (ver `04_decision_engine.md` DE-3, pendente).** O email do cliente
só é capturado no passo de Identificação — quem abandona antes disso
(precisamente onde está a maior perda absoluta, F.0) é anónimo para o
sistema de recuperação (Mautic). Os ~5% de recuperação não são sinal de
uma campanha fraca — é o teto matemático de um sistema que só consegue
identificar uma fração dos abandonos. A alavanca correta não é melhorar
o email, é capturar o email mais cedo no funil (gaveta do carrinho ou
passo 1).

**F.12 — Campo de morada limitado a 28 caracteres**, insuficiente para
muitas moradas portuguesas (ex.: "Rua Dr. António Bernardino de
Almeida" tem 37) — risco de entregas falhadas.

**F.13 — Infraestrutura Redicom a 80% de espaço utilizado** (aviso nativo
do próprio dashboard) e tempo de resposta da listagem de produtos com
variação grande (84ms–1,6s) entre leituras. Não é causa direta de
abandono no checkout, mas merece acompanhamento.

**Não testado:** viewport mobile real (<768px, ferramenta de teste não
permitiu forçar); conteúdo/atraso reais do email/SMS de recuperação (por
acordo, só diagnóstico técnico, ver F.11); variável exata do gatilho de
F.2 (precisa de logs de servidor).

**Ordem de ataque original (esforço/impacto, SUPERADA — ver secção G):**
~~1. Remover `min-width: 1130px` / criar breakpoint intermédio (F.1)~~
~~2. Instrumentar `actions_registo.php` para medir frequência de F.2~~
3. Confirmar com a Redicom o mapeamento do funil (F.0) e alargar o
   intervalo do dashboard para 30 dias antes de decidir com estes números
4. Capturar o email mais cedo no funil (resolve F.11)
5. Mostrar total no passo de Identificação; renomear "Finalizar" (F.5)
6. Remover botão de desconto duplicado (F.7); CTA direto de pagamento no
   mini-carrinho (F.8)
~~7. Usar Microsoft Clarity para confirmar F.1/F.2/F.3~~ → **feito, ver G**

## G. Quantificação real via Microsoft Clarity (20-08-2026, 30 dias)

Fonte: Microsoft Clarity, projeto `h7znnsrur0` (Mellmak), últimos 30 dias
até 19-08-2026. Só leitura. **Corrige a priorização da secção F** — os
volumes reais mudam o que é urgente.

**Funil real (30 dias, muito mais robusto que o único dia do dashboard
Redicom usado em F.0):** 1.520.259 sessões no site → 64.206 chegam ao
checkout (4,22%) → 50.493 ao Resumo → 8.151 à Identificação → 6.388 ao
Pagamento → **2.560 à Confirmação (3,99% das sessões de checkout)**. Nota:
~5.100 das 6.388 que chegam ao Pagamento saltam a Identificação —
já estavam autenticadas. F.4/F.5 (fricções da Identificação) afetam
sobretudo tráfego não autenticado, não a totalidade do checkout.

**G.1 — F.2 é maior e diferente do que se pensava: não é redirecionamento
para a homepage, é perda de sessão/cookie, concentrada em webviews.**
Existe um estado de erro explícito e silencioso (`err=2`) que devolve o
utilizador ao formulário de login vazio, sem mensagem. Afeta **1.316
sessões — 16,1% de todas as que chegam à Identificação**. Assinatura nas
gravações: a sessão do Clarity começa já na página de erro (entrada =
saída, 1 página, sessão nova) e concentra-se em **FacebookApp (19,17%),
InstagramApp (12,67%) e MobileSafari (13,88%) — 38% de todo o tráfego de
checkout.** Padrão clássico de bloqueio de cookies/storage em webviews
in-app e Safari ITP, não de código partido. **Passa a ser a prioridade
nº1** — é o único dos problemas testados com impacto direto e mensurável
na conversão em volume real.

**G.2 — Novo achado, não estava na auditoria original: erro
`meta[itemprop=productid]` é 81% de todos os erros JS do checkout.**
Um script (provavelmente de tracking/analytics) assume a existência de
`meta[itemprop=productid]` em páginas que não são de produto, incluindo o
checkout — 54.080 ocorrências em 30 dias, entre os dois erros dominantes:
`cannot read properties of null (reading 'getattribute')` (57,72%) e a
variante do `meta[itemprop]` (23,50%). Correção trivial (guard/null
check) e potencialmente ligada aos mesmos problemas de tracking já
documentados em `01_data_sources.md`.

**G.3 — Novo achado: `validatephone_status is not defined`** — erro de
JS do próprio checkout (validação de telefone), 1,8-5% dos erros
consoante o segmento. Ao contrário de G.4, este é mesmo do checkout e
merece investigação.

**G.4 — F.10 estava mal atribuído: o erro `$ is not defined`
não é um problema de checkout.** Apenas 1 sessão de checkout em 30 dias
(0,002%) o dispara. As 432 sessões afetadas no site inteiro concentram-se
em fichas de produto, tráfego mobile vindo de `m.facebook.com` (57,4%).
**Retirar da lista de prioridades de checkout** — reportar separadamente,
se relevante, como problema de páginas de produto.

**G.5 — F.1 (zona morta 768-1130px) é real mas de volume marginal.**
0,44% das sessões de checkout por segmento de dispositivo (Tablet, proxy
mais limpo — o filtro por largura de documento está poluído porque
sessões "Celular" aparecem com largura >767px, sinal de que a métrica de
largura do Clarity já reflete o próprio overflow que queríamos medir).
Confirmado visualmente em heatmap de tablet: bloco de totais colide com o
rodapé. Continua a valer a pena corrigir (é barato), mas **deixa de ser a
prioridade nº1** — G.1 tem 16-36× mais volume.

**G.6 — Novo achado: existe um segundo checkout, `/checkout/p1/`, nunca
auditado.** 7.661 page views em 30 dias, fora do âmbito de tudo o que
foi testado até agora. Estado desconhecido.

**Limitações do próprio Clarity, registadas para não repetir o erro:**
sem filtro de intervalo de viewport (só >/</=), sem métrica de scroll
horizontal, heatmap só em 3 larguras fixas — nenhuma delas cobre
1024-1130px diretamente.

**Nova ordem de ataque (substitui a de F, por volume real):**
1. `err=2` — perda de sessão/cookie na Identificação, foco em Safari ITP
   e webviews Facebook/Instagram (G.1) — 16,1% de quem chega ao passo
2. Guard/null-check no script que assume `meta[itemprop=productid]`
   (G.2) — 81% dos erros JS do checkout, correção trivial
3. Investigar `validatephone_status is not defined` (G.3)
4. Auditar `/checkout/p1/`, nunca testado (G.6)
5. Zona morta 768-1130px (F.1/G.5) — barato, mas volume baixo
6. Restantes itens de F (F.3 a F.9, F.12) por ordem de esforço
7. `$ is not defined` (G.4) — não é checkout, reportar à parte ou ignorar

## D. Fora de âmbito nesta sessão

Os pontos 1–4 da secção 11 do briefing operacional (auditoria de catálogo em
tempo real, proteção de links, catálogos Meta/Google, merchandising ao vivo
no site) requerem acesso direto a sistemas de produção (Redicom/PIM, Meta
Business Manager, Google Merchant Center, CMS da Mellmak) que este ambiente
não tem — e que, mesmo que tivesse, exigiriam aprovação humana antes de
qualquer alteração (Regra 6 da constituição). O Neptune COS pode estruturar
as regras e o checklist (feito acima) para que, quando esses conectores
existirem, as verificações corram automaticamente.
