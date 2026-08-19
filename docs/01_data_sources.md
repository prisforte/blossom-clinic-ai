# 01 — Fontes de Dados

**Estado:** Ativo — limitações conhecidas documentadas a partir de uma
auditoria técnica interna de junho de 2026 e de achados de agosto de 2026
(ver `05_backlog.md`). Âmbito e frequência de sincronização por conector
continuam pendentes (dependem dos conectores reais da Fase 5).

Este documento segue a hierarquia de fontes definida em
`00_system_constitution.md`. Para cada fonte: o que é oficial, limitações
conhecidas, e estado de remediação.

---

## Redicom (fonte oficial — ver Regra 1 da constituição)

**Limitações conhecidas:**

1. **Múltiplos relatórios de faturação divergentes para o mesmo dia**,
   sem reconciliação (ver `05_backlog.md` item 0). Causa provável, por
   critério de data (encomenda vs. pagamento) e inclusão/exclusão de
   trocas — parcialmente reconciliado em 20-08-2026.
2. **Evento de Purchase (auditoria de junho, item 1):** não disparava de
   forma fiável na página de sucesso da compra — em alguns casos era
   enviado antes de a compra estar realmente concluída. Isto pode
   contaminar tanto o lado Redicom (registo da venda) como o lado GA4
   (conversões reportadas a Meta/Google).
3. **Dependência técnica (auditoria de junho, item 20):** várias correções
   de tracking não podem ser feitas só via GTM porque estão implementadas
   diretamente pela Redicom — qualquer correção de tracking depende de
   intervenção da própria Redicom, não é resolúvel só do lado do site.

**Estado de remediação:** um conjunto de pedidos técnicos foi preparado
para a Redicom em junho (remoção do GA4 nativo, revisão do Purchase,
revisão dos eventos de ecommerce, normalização do tracking) — **estado de
execução por confirmar**. Os achados de agosto (item 0 e B.7 do backlog)
são consistentes com estes pedidos ainda não estarem fechados.

## GA4 (fonte secundária — ver Regra 2 e 3 da constituição)

**Limitações conhecidas (auditoria de junho):**

- **Implementação duplicada:** GA4 existia em simultâneo via Google Tag
  Manager e via código nativo da Redicom (gtag) → `page_view` duplicado,
  sessões duplicadas, utilizadores duplicados, métricas inflacionadas.
- **+33.000 sessões "Unassigned"** — perda de informação de origem de
  tráfego, dificulta avaliar campanhas. (Este é o campo "Unassigned" já
  previsto na página Data Health da Fase 4 do MVP.)
- **Eventos de ecommerce incompletos/incorretos:** `add_to_cart`,
  `begin_checkout` e `purchase` nem sempre seguiam o modelo recomendado
  pelo GA4.
- **Consent Mode:** consentimento considerado aceite por defeito —
  medições pouco fiáveis e possível não conformidade.
- **Cross-domain:** configuração entre plataformas por rever — risco de
  perda de sessões/atribuição.
- **Eventos redundantes:** vários eventos enviados mais do que uma vez.
- **Dados históricos contaminados:** duplicações, eventos incorretos e
  atribuições erradas tornam parte da informação histórica pouco fiável —
  aplicar com cautela em comparações ano-a-ano.

**Estado de remediação:** limpeza do contentor GTM foi feita (remoção de
etiquetas Universal Analytics/Google Optimize obsoletas); origem da
duplicação do GA4 foi identificada; **a coexistência com implementações
nativas da Redicom continua por rever antes de uma migração completa para
um modelo centralizado em GTM.**

## Meta Ads (fonte oficial de investimento — ver Regra 4 da constituição)

**Limitações conhecidas:**

- **Pixel duplicado** em determinadas páginas (auditoria de junho, item
  14) → duplicação de eventos, pior aprendizagem do algoritmo de
  otimização.
- Campanhas podem estar a **otimizar com dados de conversão incorretos**,
  como consequência direta dos problemas de Purchase/GA4 acima.
- Sem conector API ligado nesta altura (Windsor.ai só tem Google Ads
  ligado); extração atual é manual via UI do Ads Manager, não persistente
  (ver `05_backlog.md` E.6).

## Google Ads (fonte oficial de investimento — ver Regra 4 da constituição)

**Limitações conhecidas:**

- **Enhanced Conversions mal configuradas, associadas a trigger incorreto**
  (auditoria de junho, item 7) → perda de qualidade da otimização.
- **Smart Bidding e Performance Max afetados** pelos mesmos problemas de
  Purchase/tracking (auditoria de junho, item 17).
- **Valor de compra (purchase value) subavaliado na Mellmak** (achado de
  20-08-2026, `05_backlog.md` B.7): ticket médio implícito no Google
  ≈26€/compra vs. ≈80€/compra no Meta, mesma loja/período — a Forte Store
  não apresenta este desvio. Este achado de agosto é consistente com os
  itens 6/7 da auditoria de junho ainda não estarem corrigidos — **não é
  um problema novo, é uma continuação de um problema já identificado há
  2 meses.** Qualquer "valor conv./custo" do Google Ads para a Mellmak
  deve ser lido com esta reserva até correção confirmada.

## Base de dados de clientes / CRM

**Limitação conhecida (auditoria de junho, item 18):** muitos clientes
registados sem email, sobretudo em loja física — impede desconto online
e reduz a qualidade da base para email marketing. Foi proposta a
obrigatoriedade de preenchimento de email em loja — estado de
implementação por confirmar.

---

## Regra de leitura para o Neptune COS

Enquanto os itens acima não estiverem confirmados como corrigidos, o
Decision Engine e a página Data Health devem tratar **GA4 e Google Ads
(lado de valor/conversão) como fontes de confiança reduzida para a
Mellmak**, independentemente do que a Regra 1-4 da constituição definir
como fonte oficial em condições normais — a hierarquia de fontes assume
tracking saudável, o que ainda não está confirmado aqui.
