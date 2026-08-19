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

**Causa raiz confirmada em 20-08-2026 (auditoria técnica de checkout, ver
`05_backlog.md` F.11):** os 5% não são sinal de campanha fraca — são um
teto estrutural. O email do cliente só é capturado no passo de
Identificação (2.º passo do checkout real); quem abandona antes disso —
exatamente onde está a maior perda absoluta do funil — é anónimo para o
sistema de recuperação (Mautic). **A ação correta não é otimizar o
conteúdo do email, é antecipar a captura de email para a gaveta do
carrinho ou o 1.º passo do checkout.** Só faz sentido reavaliar esta regra
depois dessa mudança ser feita.

**Correção a uma nota anterior:** esta versão do documento continha uma
leitura errada do funil ("maior perda relativa entre Carrinho e Página de
Pagamento, 50% de passagem") — essa é, na verdade, a transição **mais
eficiente** das três (50% é a melhor taxa de passagem do funil). As
transições mais fracas são as seguintes (44% e 45% de passagem),
correspondentes no site real aos passos de Identificação e Pagamento —
ver `05_backlog.md` F.0.

**Ressalva de fiabilidade (F.0, F.10):** a legenda do painel "Funil do
Checkout por Utilizador" não corresponde à ordem real do checkout, e um
erro de JavaScript no GTM (`$ is not defined`, 6x no passo de
Identificação) pode estar a subcontar eventos. Confirmar com a Redicom o
mapeamento exato de cada barra e alargar o intervalo para 30 dias antes
de tratar os valores percentuais como definitivos — a existência da
priorização (identificação/pagamento mais fracos que carrinho) é mais
robusta do que os números exatos.

---

## Regras ainda pendentes (Fase 6 original)

- Alerta de MER: queda > 25% face à média dos últimos 3 dias
- Verificação de checkout: queda de conversão > 20% com sessões estáveis/altas
- Bloqueio de recomendações de escala quando tracking crítico está ativo
- Meta aumenta investimento sem crescimento proporcional de receita → não
  escalar novamente

Estas regras mantêm-se por formalizar com o mesmo padrão de rigor das duas
acima: só entram aqui com fórmula exata e fonte de dados confirmada.
