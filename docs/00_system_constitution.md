# 00 — Constituição do Sistema

**Projeto:** Neptune Commerce Operating System (Neptune COS)
**Implementação inicial:** Mellmak
**Estado:** Ativo — regras obrigatórias

## Propósito

Este documento define as regras invioláveis que governam o Neptune Commerce
Operating System. Todas as camadas do sistema — conectores, domínio, serviços
e interface — devem respeitar estas regras sem exceção. Nenhum KPI, cálculo
ou recomendação pode contradizer esta constituição. Qualquer alteração a este
documento requer aprovação explícita antes de ser refletida em código.

---

## 1. Redicom — Fonte Oficial

Redicom é a fonte oficial (sistema de registo) para:

- Faturação
- Encomendas
- Ticket médio (AOV)
- Devoluções
- Produtos vendidos
- Recuperação de carrinho
- Funil comercial

Todo o KPI financeiro ou comercial apresentado ao utilizador tem de ter
origem em dados reconciliados da Redicom. Nenhum outro conector pode
substituir ou sobrepor-se a estes valores.

## 2. GA4 — Fonte Secundária

GA4 é fonte secundária, usada exclusivamente para:

- Sessões
- Utilizadores
- Comportamento de navegação
- Páginas
- Origem de tráfego
- Diagnóstico de tracking

## 3. Regra de Isolamento de Receita

**Nunca usar a receita reportada pelo GA4 como faturação oficial.** GA4 pode
divergir da Redicom devido a bloqueadores de anúncios, falhas de tracking,
ITP/cookies, duplicação de eventos ou perda de conversões. A receita GA4 só
pode ser exibida como métrica de diagnóstico e tem de estar explicitamente
rotulada como "não oficial".

## 4. Meta Ads e Google Ads — Fontes Oficiais de Investimento

Meta Ads e Google Ads são fontes oficiais para:

- Investimento
- Impressões
- Alcance
- Cliques
- CPM
- CPC
- CTR
- Frequência
- Custo por resultado

## 5. Regra de Confiança nas Recomendações

Nenhuma recomendação pode ser apresentada como certeza quando:

- Existirem falhas de tracking ativas;
- Houver dados incompletos no período;
- A Redicom não estiver reconciliada;
- O período ainda estiver em processamento.

Nestes casos, o sistema tem de rotular explicitamente a recomendação como
"baixa confiança", indicando a causa e — sempre que possível — a métrica
afetada.

## 6. Regra de Autonomia Limitada (Human-in-the-loop)

O sistema sugere ações mas **nunca** altera campanhas, orçamentos, produtos
ou qualquer sistema de produção sem aprovação humana explícita. Não existe
execução automática a partir de recomendações — toda a escrita em sistemas
externos passa por confirmação humana.

---

## Aplicabilidade

Estas regras aplicam-se a:

- Todos os conectores em `/src/connectors`;
- Todos os KPIs definidos em `/docs/02_kpi_master.md`;
- Todas as regras de negócio em `/docs/03_business_rules.md`;
- Todas as decisões do Decision Engine em `/docs/04_decision_engine.md`;
- Toda a interface em `/src/ui`.

Qualquer módulo que viole uma destas regras é considerado um bug crítico.
