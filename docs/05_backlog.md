# 05 — Backlog

**Estado:** Ativo — primeira versão, com base no briefing operacional de
19-08-2026 e na inspeção do ficheiro
`MM_Ecommerce_Performance_Tracking_2026.xlsx`.

Convenção de estado: `bloqueado` (depende de acesso/decisão externa) ·
`por validar` (dono humano tem de confirmar) · `pronto para implementar`
(claro o suficiente para virar código/config) · `concluído`.

---

## A. Pontos por validar (origem: briefing operacional, secção 10)

| # | Item | Estado | Dono |
|---|---|---|---|
| 1 | Hora exata em que os artigos de 30% foram repostos na Mellmak (11-08) | por validar | operações/compras |
| 2 | Lista final e validada de artigos a 30%, 50% e 70% | por validar | compras |
| 3 | Confirmação de que todos os URLs de campanhas antigas continuam funcionais | por validar | marketing/dev |
| 4 | Confirmação de que o catálogo Meta e o catálogo Google estão atualizados | por validar | marketing |
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
| C.12 | Auditoria de catálogo/stock/preço/URL, catálogos Meta/Google, links de campanha | fora do Neptune COS | bloqueado — requer acesso direto a Redicom/PIM, Meta Business Manager e Google Ads, que esta sessão não tem; a executar por quem tem esse acesso |

## D. Fora de âmbito nesta sessão

Os pontos 1–4 da secção 11 do briefing operacional (auditoria de catálogo em
tempo real, proteção de links, catálogos Meta/Google, merchandising ao vivo
no site) requerem acesso direto a sistemas de produção (Redicom/PIM, Meta
Business Manager, Google Merchant Center, CMS da Mellmak) que este ambiente
não tem — e que, mesmo que tivesse, exigiriam aprovação humana antes de
qualquer alteração (Regra 6 da constituição). O Neptune COS pode estruturar
as regras e o checklist (feito acima) para que, quando esses conectores
existirem, as verificações corram automaticamente.
