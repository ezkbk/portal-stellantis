# Portal Stellantis — Runbook de Manutenção

> Este arquivo é lido automaticamente pelo Claude/Cowork ao abrir esta pasta.
> Ele explica **o que é o portal, como atualizar os números todo mês, como gerenciar
> usuários e como publicar**. Serve tanto para a pessoa responsável quanto para o Claude dela.

---

## 1. O que é / onde fica

- **Portal Stellantis**: painel de energia do cliente Stellantis (volumes, custos, orçado × realizado,
  PLD por submercado e eficiência da produção). É um **único arquivo estático**: `index.html`.
- **Repositório GitHub**: `ezkbk/portal-stellantis` (branch `main`).
- **No ar (Netlify)**: https://portal-stellantis.netlify.app/ — **publica sozinho a cada push** no GitHub.
- **Não tem backend nem build.** Tudo (dados + código) vive dentro do `index.html`.

## 2. Fluxo de trabalho (SEMPRE o mesmo)

1. Abrir esta pasta no Claude/Cowork e pedir a mudança.
2. O Claude edita o `index.html`.
3. **Validar** antes de publicar (ver seção 7).
4. No **GitHub Desktop**: escrever um resumo curto → **Commit** → **Push**.
5. O Netlify republica em 1–2 min. Conferir no link acima.

> Regra de ouro: **nunca publicar dado inconsistente**. É um portal que o cliente vê.
> Se um número não bate, parar e conferir — melhor atrasar do que publicar errado.

## 3. Estrutura de dados dentro do index.html

Todos são objetos JavaScript no `<script>` principal:

| Constante | O que guarda |
|---|---|
| `DS.mon["AAAA-MM"][UNIDADE]` | Volumes e custos mensais por unidade (lp, cp, rlp, rcp, rdem, rdist, renc, cme, clp, ccp, cmd, enc, dist_rmwh, dpl, dfpl, dpc, dfpc). **É o que alimenta a maior parte do dashboard.** |
| `ORC_DATA[UNIDADE]` | Orçado × Realizado 2026 (vol_orc, vol_real, dev_vol, cost_orc, cost_real, dev_cost — arrays de 12 meses, Jan=0 … Dez=11). Tem a chave especial **`BRASIL`** (consolidado). |
| `PLD_HIST["AAAA-MM"]` | PLD histórico por submercado `{SE, S, NE, N}` (com imposto). |
| `PLD_PREV["AAAA-MM"]` | PLD **previsto**, 3 cenários por submercado: `{SE:[pes,rea,oti], S:[...], NE:[...], N:[...]}`. |
| `PROD_DATA[UNIDADE]` | Volumes de produção por planta (veículos/toneladas/horas). |
| `USERS` | Logins (ver seção 6). |

**Chaves de unidade** (usar exatamente assim): `TEKSID_C`, `AUTO_C`, `MEC_C`, `PORTO REAL_C`,
`GOIANA_C`, `ITAUNA_C`, `CAMPO LARGO_C`, `MOPAR_C`, `IGARAPÉ_C`, `JABOATAO_C`, e `BRASIL` (consolidado).

## 4. Atualização mensal dos números (o mais importante)

Origem: a planilha **"Report Mensal Stellantis vX.xlsx"** que o cliente/time envia todo mês.
Ao adicionar um mês novo, atualizar **quatro** estruturas. **Todos os valores de R$ e GWh entram divididos por 1000** (a planilha vem em unidade cheia).

### 4.1 `DS.mon["AAAA-MM"]` (por unidade)
Da planilha, por unidade:
- **Volumes_Energia**: `lp = coluna LP / 1000`, `cp = coluna CP / 1000` (viram GWh).
- **Custos_Fornecimento**: `rlp`, `rcp` (÷1000); `clp`, `ccp` = R$/MWh direto.
- **Custos_Demanda**: `rdem = (ponta + fora-ponta) / 1000`; `dpl, dfpl, dpc, dfpc`.
- **Custos_Encargos_Distribuidora**: `rdist = (col ponta + fora-ponta) / 1000`; `dist_rmwh`.
- **Encargos_CCEE**: `renc = rateio CCEE / 1000 + rdist`.
- Derivados: `cme = (clp*lp + ccp*cp)/(lp+cp)`; `cmd = rdem/(lp+cp)`; `enc = renc/(lp+cp)`.

### 4.2 `PLD_HIST["AAAA-MM"]`
Aba **PLD**, bloco "PLD HISTÓRICO" (**com imposto**): SE/CO, SUL, NORDESTE, NORTE.

### 4.3 `ORC_DATA` (Orçado × Realizado)
Aba **"Orçado x Realizado 2026"**. Cada unidade tem um bloco (Consumo Projetado/Realizado, Custo
Projetado/Realizado, Desvio %). Preencher `vol_real`, `cost_real`, `dev_vol`, `dev_cost` no índice do mês
(Jan=0 … Dez=11). Meses sem realizado ficam `dev = -1` e valor `0`.

> **⚠️ Cuidado com o BRASIL (consolidado).** O gráfico do total lê `ORC_DATA["BRASIL"]`.
> Convenção: **`BRASIL.vol_real` = soma das unidades** (bate com os volumes medidos). O custo do BRASIL
> vem da linha **"CONSOLIDADO FÁBRICAS"** da planilha. **Sempre conferir** que
> `BRASIL.vol_real[mês] == soma das 10 unidades`. Já houve mês em que a célula CONSOLIDADO da planilha
> estava quebrada (mostrava metade do valor) — nesse caso o certo é a **soma das unidades**.

### 4.4 `PLD_PREV` (previsão, 3 cenários)
Aba **PLD**, bloco "PLD PREVISTO": data na coluna da previsão, e **3 colunas por submercado** =
os 3 cenários. Convenção adotada: **coluna mais alta = Pessimista, meio = Realista, mais baixa = Otimista**
→ `PLD_PREV[mês][submercado] = [pessimista, realista, otimista]`.

### 4.5 Não mexer (a menos que venha arquivo)
`PROD_DATA` (produção) só muda quando vier o arquivo de volumes de produção. Se não vier, manter no
último mês existente.

## 5. Cenários de PLD e Eficiência (features prontas)

- **PLD por submercado**: histórico sólido + previsão em 3 linhas tracejadas (Pessimista/Realista/Otimista).
  Fonte: `PLD_PREV`. Botões de submercado filtram; a previsão é de mercado (igual entre submercados).
- **Eficiência da Produção**: seletor **"Denominador: Produção / MWh"** → mostra R$/unidade produzida ou R$/MWh.
  R$/Produção só faz sentido por planta (não soma veículos + toneladas).

## 6. Gerenciar usuários (logins)

Login = **e-mail da pessoa**; a autenticação é a constante `USERS` no `index.html`:

```js
const USERS = {
  "<base64 de 'email:senha'>": { "nome": "Fulano", "plantas": ["AUTO_C"] },
  ...
  "<base64 de 'admin:energia2026!@#'>": { "nome": "Administrador", "plantas": "ALL" }
};
```

- `"plantas": ["CHAVE_DA_UNIDADE"]` → a pessoa vê **só** aquela planta (sem Brasil consolidado, sem a aba
  "Comparativo entre Unidades", tabela de eficiência filtrada).
- `"plantas": "ALL"` → **login global**, vê tudo.
- A chave do objeto é `btoa("email:senha")` (base64 de `email` + `:` + `senha`). Peça ao Claude para gerar.
- A lista atual de logins/senhas está em **`ACESSOS_Portal_Stellantis.md`** — manter esse arquivo em dia
  ao criar/alterar usuários.

> **⚠️ Segurança (importante saber ao criar usuários):** o login **esconde** as outras plantas na tela,
> mas **todos os dados ainda estão dentro do `index.html`** — alguém com conhecimento técnico consegue
> ver as demais plantas pelo "ver código-fonte". É separação visual, **não** sigilo real. Para isolamento
> verdadeiro seria um arquivo por planta ou login com backend (falar com o Israel antes de prometer sigilo
> total ao cliente).

## 7. Validação antes de publicar (obrigatório)

Peça ao Claude para, após qualquer edição:
1. **Checar a sintaxe** de cada `<script>` do arquivo (`node --check`). Módulos devem ser checados como `.mjs`.
2. **Carregar o arquivo no jsdom** (com um stub do Chart.js) e confirmar **0 erros de runtime**.
3. Ao mexer em usuários, **testar um login de planta** (vê só a dela) e um **global** (vê tudo).
4. Ao mexer em números, **conferir BRASIL = soma das unidades** no mês novo.

> Lição que vale ouro aqui: "a verificação que não verifica é pior que nenhuma". Se o Claude disser que
> validou, peça para mostrar a saída real dos testes.

## 8. Publicar

GitHub Desktop → repositório **portal-stellantis** → Commit (mensagem curta do que mudou) → Push.
Netlify republica sozinho. **Não** há passo manual no Netlify para publicar conteúdo.

## 9. Pendências abertas (confirmar com o cliente/Israel)

- **CAPR** e **CMA** (da tabela de e-mails) não existem como unidade no portal — a que correspondem?
- **Pátio Igarapé** existe no portal mas não tinha e-mail — quem acessa?
- **Ordem dos cenários de PLD** — confirmar se alto=Pessimista, baixo=Otimista.
- **Julho/2026** do PLD: há um vão entre o histórico (jun) e a previsão (ago) — some quando entrar o relatório de julho.
