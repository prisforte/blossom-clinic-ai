# 03 — Regras de Negócio (Mellmak)

**Estado:** Ativo — primeira versão, com base no briefing operacional de
19-08-2026 e no ficheiro `MM_Ecommerce_Performance_Tracking_2026`. Sujeito a
revisão contínua conforme novos episódios operacionais.

---

## 1. Dupla meta mensal

A Mellmak opera com duas metas distintas em simultâneo, que o sistema deve
tratar como objetos separados (nunca substituir uma pela outra):

| Meta | Valor (agosto 2026) | Natureza |
|---|---:|---|
| **Meta formal** | 254.869,90 € | Objetivo orçamental oficial do mês |
| **Meta operacional** | 500.000 € | Objetivo comercial ambicioso, definido à parte |

Regras:
- Ultrapassar a meta formal não encerra a análise do mês — o sistema deve
  continuar a reportar o gap face à meta operacional enquanto esta estiver
  definida.
- O estado de trajetória (`NO CAMINHO` / `EM RISCO` / `FORA DE TRAJETÓRIA`,
  ver `04_decision_engine.md`) deve ser calculado **para cada meta em
  separado** — é possível estar "NO CAMINHO" face à meta formal e "FORA DE
  TRAJETÓRIA" face à meta operacional ao mesmo tempo.
- A meta operacional só é válida enquanto houver uma decisão humana explícita
  a defini-la para o período em causa (não é uma regra permanente).

## 2. Regra Nuno Forte — teto de investimento em época de descontos

**Fórmula confirmada** (existente na folha `TRADING TRACKING 2026`, coluna
"% Budget gasto"):

```
% Investido = (Investimento Meta + Investimento Google) / Faturação
```

**Regra:** durante períodos de campanha de saldos/descontos (ver secção 3),
o investimento total (Meta + Google) não deve ultrapassar **10% da
faturação**.

**Gatilho de "época de descontos":** uma linha ativa na folha
`CAMPANHAS - META & GOOGLE 2026` (Data de início ≤ hoje ≤ Data de fim) cujo
nome/brief contenha desconto (ex.: "SALDOS", "SALES", "FLASH SALE", "% NC").

**Pontos ainda por confirmar com o Nuno** (não assumir sem validação):
- Se existe um teto diferente (ou nenhum) fora de época de descontos.
- Se o limite é calculado sobre o acumulado do mês ou sobre o período da
  própria campanha.

**Nota de fiabilidade:** a fórmula oficial deste indicador depende de fontes
de custo que, no workbook mestre, estão desatualizadas desde dezembro de
2024 (`DAILY_CHANNEL_META_ADS`, `DAILY_CHANNEL_GOOGLE_ADS`,
`DAILY_BUDGET_TRACKET`, `CHANNEL_RESUME`) e desde junho de 2026
(`2026 Custos publicidade`). Por isso, para julho/agosto de 2026 o valor
correto só está disponível na folha de controlo diário manual ("MELLMAK"),
não no cálculo automático do workbook mestre. O Decision Engine deve tratar
esta fonte automática como **não fiável para o período atual** até ser
reconciliada.

## 3. Estrutura de desconto e catálogos (época de saldos)

Campanha ativa: **"Últimos Saldos — até 70%"** (renomeada a partir de
"Tudo 50% e 70%" em 11-08-2026 — ver cronologia, secção 5).

Sub-páginas: Saldos 30%, Saldos 50%, Saldos 70%.

| Catálogo | Conteúdo | Regra de uso |
|---|---:|---|
| Lista 30% Mellmak (11-8 a 6-9-2026) | 1.643 artigos | Comunicação e campanhas de 30%, após validar stock e URLs |
| Lista 50% Mellmak (11-8 a 6-9-2026) | 2.465 artigos | Campanhas e páginas de 50% |
| Lista Compras 12/8 (30% até 18-9-2026) | 1.553 artigos | Só após validação final de Compras |
| Catálogo Inverno / visualização | 5.709 artigos | Nunca usar diretamente em campanhas de saldos sem validar preço, stock, coleção e página de destino |

Regras gerais:
- Nunca usar o catálogo genérico "Todos os produtos" em campanhas de saldos
  sem filtros — pode incluir nova coleção, artigos sem a promoção comunicada
  ou sem stock.
- Meta e Google só devem receber artigos com stock, preço promocional
  correto e URL ativa.
- Criativo, título da campanha, desconto comunicado e página de destino têm
  de coincidir sempre.
- URLs de campanhas antigas (50%/70%) mantêm-se ativos enquanto existirem
  anúncios em circulação a apontar para eles — nunca desativar sem
  verificar dependências de anúncios ativos.

## 4. Categorias prioritárias

Famílias identificadas como críticas (maior peso em carrinho):
**T-shirts, malas, sapatilhas e jeans** — 1.303 artigos em carrinho, 45,4%
do peso em carrinhos, 49.345,57 € de valor potencial (53,9% do valor
potencial total em carrinho).

Ordem de prioridade em merchandising, CRM e anúncios:
1. T-shirts
2. Malas
3. Sapatilhas
4. Jeans
5. Polos e camisas
6. Mais vendidos
7. Últimas unidades

Malas e sapatilhas aparecem recorrentemente nos tops de venda — a página de
Mulher, páginas de saldos e páginas de marca devem privilegiar estes artigos
sempre que ativos, com stock e preço corretos.

## 5. Cronologia operacional — episódio de catálogo (agosto 2026)

Registo do episódio que afetou a trajetória do mês, para referência e para
alimentar o Decision Engine com um caso real de "tracking crítico" /
inconsistência de catálogo.

**1–5 de agosto — arranque forte.** Campanha "Últimos Saldos — tudo 50% e
70%", comunicação e páginas construídas sobre artigos de Primavera/Verão
2026. Faturação: 123.093,00 € em 5 dias (média 24.618,60 €/dia).

**5 de agosto — alteração de catálogo.** Ocultação urgente de referências
para transição para Inverno 2026: 970 referências-cor na lista comunicada
nesse momento, 1.520 referências-cor ocultas no acumulado (114 nessa manhã).
Identificado depois que algumas não deveriam ter sido retiradas. A ocultação
propagou-se hierarquicamente (categoria → subcategorias, marcas, páginas de
calçado/malas/Mulher), quebrando coerência entre anúncios ativos, páginas de
destino e produto disponível. Alteração simultânea de portes (3,99 € → 2,99 €
em Portugal, remoção do limite de portes grátis) — efeito isolado ainda não
medido.

**6 de agosto — impacto.** Faturação cai para 13.764,00 € (-52,8% vs. dia 5);
MER cai de 8,41 para 5,86 (-30,3%); investimento Meta também desce 32,4%.
Associação temporal forte com múltiplas alterações simultâneas — não se pode
atribuir a um único fator.

**6–10 de agosto — validação pendente.** Pedido a Compras para confirmar
quais artigos deveriam ficar ocultos; validação não fechada de imediato,
prolongando exposição a disponibilidade inconsistente. Notas de "PROBLEMAS
EXCEL COMPRAS" nestes dias (ver `05_backlog.md`).

**11 de agosto — correção.** Decisão: artigos vendáveis a 30% permanecem
visíveis/compráveis na Mellmak até fim do período de saldos. Ações: repor
artigos a 30%; renomear campanha para "Últimos Saldos — até 70%"; manter
páginas/links de 50%/70% para anúncios antigos; corrigir links quebrados;
atualizar catálogos Meta/Google; registar hora exata da reposição para medir
recuperação; diferenciar 30%/50%/70% na comunicação e feeds. Envio de SMS de
recuperação à base de clientes.

**Impacto agregado (1–5 vs. 6–18 de agosto):**

| Período | Faturação | Média diária |
|---|---:|---:|
| 1–5 agosto | 123.093,00 € | 24.618,60 € |
| 6–18 agosto | 155.408,00 € | 11.954,46 € |

Queda de ~51,4% na média diária após o dia 5. **Não deve ser apresentado
como prejuízo causado exclusivamente pela ocultação** — não há grupo de
controlo, e catálogo, portes, comunicação, mix de produto e sazonalidade da
promoção mudaram ao mesmo tempo. Regista-se como impacto comercial
relevante e temporalmente coincidente.

## 6. Controlo diário obrigatório

O controlo diário da Mellmak (fonte para o Data Health / Executive) deve
incluir, no mínimo:

1. Faturação oficial
2. Investimento Meta e Google
3. MER diário e acumulado
4. Encomendas, ticket médio e conversão
5. Nº de artigos ativos por catálogo de desconto (30% / 50% / 70%)
6. Validação de stock e preço
7. Estado dos links de anúncios, SMS, email, homepage e páginas de categoria
8. Vendas por categoria prioritária (T-shirts, malas, sapatilhas, jeans)
9. Alterações de catálogo, com data, hora, origem da decisão e responsável
   pela validação
10. Devoluções, cancelamentos, falhas de entrega e artigos que continuam
    online após cancelamento

Itens 5, 6, 7 e 9 ainda não têm fonte de dados estruturada — ver
`05_backlog.md`.

## 7. Regra de aprovação humana para investimento

Nenhuma alteração de orçamento ou estrutura de campanha é executada pelo
sistema. Antes de qualquer proposta de aumento de investimento, o sistema
deve validar: cobertura dos catálogos ativos, estado das páginas de destino,
disponibilidade dos produtos anunciados, e os resultados financeiros
oficiais do período. Só depois disso a proposta pode ser apresentada — e
sempre como proposta sujeita a aprovação humana (ver Regra 6 da
`00_system_constitution.md`).
