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
| 7 | Correção das discrepâncias entre atribuição Redicom/GA4 e faturação oficial | bloqueado | ver item B.2 abaixo |
| 8 | Teto de investimento de 10% (Regra Nuno Forte) fora de época de descontos: existe? qual? | por validar | Nuno Forte |
| 9 | Limite de 10% é sobre o mês acumulado ou sobre o período da campanha? | por validar | Nuno Forte |
| 10 | Efeito isolado da alteração de portes (2,99€) sobre conversão/ticket médio | por validar | análise futura, precisa de mais dados pós-mudança |

## B. Data Health — gaps encontrados no workbook mestre

Achados da inspeção de `MM_Ecommerce_Performance_Tracking_2026.xlsx`
(41 folhas). Relevantes porque explicam *porque* várias métricas oficiais
não estão disponíveis para julho/agosto de 2026 apesar de existirem como
colunas definidas no sistema.

| # | Gap | Detalhe | Impacto |
|---|---|---|---|
| B.1 | Conectores automáticos de custo de anúncios parados | `DAILY_CHANNEL_META_ADS`, `DAILY_CHANNEL_GOOGLE_ADS`, `DAILY_BUDGET_TRACKET`, `CHANNEL_RESUME` não têm dados depois de 31-12-2024 | A fórmula oficial de "% Budget gasto" no `TRADING TRACKING 2026` devolve 0% em todo o mês de agosto — o valor real só existe na sheet manual diária |
| B.2 | Folha "2026 Custos publicidade" desatualizada | Últimos dados em 03-06-2026; sem julho/agosto | Mesma causa raiz do B.1 para o período mais recente |
| B.3 | "Taxa abandono carrinho" nunca preenchida | Coluna existe na estrutura mestre (`TRADING TRACKING 2026`) desde sempre, mas todos os meses têm valor vazio | Impossível aplicar a regra do Decision Engine "recuperação de carrinho < 12% → oportunidade prioritária" sem esta fonte |
| B.4 | "CAMPAIGNS 2026" não ligado | Coluna de campanha por dia no workbook mestre está vazia para 2026, apesar de a campanha "SALES" estar ativa e registada na folha `CAMPANHAS - META & GOOGLE 2026` | O gatilho de "época de descontos" da Regra Nuno Forte não pode assumir que este campo está fiável — usar antes a folha de campanhas diretamente |
| B.5 | Gap de reconciliação faturação por família | Ex.: 18-08, faturação por família (Mulher+Homem+Criança+Unisexo) soma 8.466,48€ vs. faturação total oficial 9.311,00€ — gap de 844,52€ | Por resolver antes de usar dados por família/categoria em decisões de merchandising |
| B.6 | `Custos publicidade` (folha legada) com erros `#REF!` | Referências quebradas na folha antiga de custos | Não usar como fonte; folha a descontinuar |

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

## D. Fora de âmbito nesta sessão

Os pontos 1–4 da secção 11 do briefing operacional (auditoria de catálogo em
tempo real, proteção de links, catálogos Meta/Google, merchandising ao vivo
no site) requerem acesso direto a sistemas de produção (Redicom/PIM, Meta
Business Manager, Google Merchant Center, CMS da Mellmak) que este ambiente
não tem — e que, mesmo que tivesse, exigiriam aprovação humana antes de
qualquer alteração (Regra 6 da constituição). O Neptune COS pode estruturar
as regras e o checklist (feito acima) para que, quando esses conectores
existirem, as verificações corram automaticamente.
