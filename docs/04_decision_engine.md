# 04 — Decision Engine

**Estado:** Parcial — três regras já confirmadas com dados reais da Mellmak
(abaixo). O conjunto completo (MER, conversão, bloqueio por tracking
crítico, etc., conforme Fase 6 do plano original) continua pendente de
formalização.

Todas as regras abaixo devolvem uma sugestão/alerta — nunca executam uma
alteração em campanhas ou produção (Regra 6 da `00_system_constitution.md`).

---

## Regras confirmadas

### DE-1 — Trajetória face à meta (dupla meta)

Aplicar em separado para a **meta formal** e para a **meta operacional**
quando esta última estiver definida (ver `03_business_rules.md` §1):

```
forecast_mensal = faturação_acumulada / dias_decorridos * dias_do_mês

se forecast_mensal >= meta:        estado = NO CAMINHO
senão se forecast_mensal >= 0.90 * meta: estado = EM RISCO
senão:                              estado = FORA DE TRAJETÓRIA
```

Output esperado (Executive): um estado por meta ativa. Exemplo real,
18-08-2026: **NO CAMINHO** face à meta formal (254.869,90€ — já superada);
**EM RISCO** face à meta operacional (500.000€ — forecast run-rate de
479.640,61€ ≈ 95,9% da meta, dentro do intervalo 90-99,99%).

### DE-2 — Regra Nuno Forte (teto de investimento em desconto)

```
se existe linha ativa em CAMPANHAS-META&GOOGLE com Data_início <= hoje <= Data_fim
   E o Brief/Nome contém indicador de desconto (%, "SALDOS", "SALES", "FLASH SALE"):

    % investido = (investimento_meta + investimento_google) / faturação

    se % investido > 10%:
        alerta = "Investimento acima do teto de época de descontos"
```

**Pré-condição de fiabilidade:** esta regra só deve correr sobre a fonte de
investimento manual/diária reconciliada — as fontes automáticas do workbook
mestre (`DAILY_CHANNEL_META_ADS`, `DAILY_CHANNEL_GOOGLE_ADS`,
`2026 Custos publicidade`) estão desatualizadas para o período atual (ver
`05_backlog.md` B.1/B.2) e não devem alimentar este cálculo até serem
corrigidas.

**Em aberto** (ver `05_backlog.md` A.8, A.9): teto fora de época de
descontos, e se o limite é sobre o mês ou sobre o período da campanha.

### DE-3 — Oportunidade prioritária de recuperação de carrinho

```
taxa_recuperacao = Recuperados / (Abandonados + Recuperados)   [painel Redicom]

se taxa_recuperacao < 12%:
    oportunidade = "Recuperação de carrinho abaixo do saudável — prioridade"
```

**Fonte confirmada em 19-08-2026:** dashboard nativo da Redicom Commerce
Cloud (`mellmak.redicom.cloud`, painel "Recuperação de Carrinho"),
disponível em tempo real por intervalo de datas. Valor observado no
momento da verificação (19-08, 15:51, dados do próprio dia): **5%** — bem
abaixo do limiar, regra ativa. Substitui a dependência do campo "Taxa
abandono carrinho" do workbook mestre, que nunca esteve preenchido (ver
`05_backlog.md` B.3, agora resolvido).

**Complementar — funil de checkout (mesma fonte):** o painel "Funil do
Checkout por Utilizador" da Redicom reporta as taxas de passagem
Carrinho → Página de Pagamento → Resumo do Carrinho → Confirmação de
Encomenda. No momento da verificação, a maior perda relativa estava entre
Carrinho e Página de Pagamento (50% de passagem). Esta fonte deve
alimentar a página Funnel do MVP (Fase 4) diretamente, substituindo a
estimativa indireta usada até aqui (Sessões → Add to cart → Checkout, via
GA4).

---

## Regras ainda pendentes (Fase 6 original)

- Alerta de MER: queda > 25% face à média dos últimos 3 dias
- Verificação de checkout: queda de conversão > 20% com sessões estáveis/altas
- Bloqueio de recomendações de escala quando tracking crítico está ativo
- Meta aumenta investimento sem crescimento proporcional de receita → não
  escalar novamente

Estas regras mantêm-se por formalizar com o mesmo padrão de rigor das duas
acima: só entram aqui com fórmula exata e fonte de dados confirmada.
