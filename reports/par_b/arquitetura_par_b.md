# Tarefa 1 — Arquitetura do Par B (Zara Sales & Brand Performance)

**Equipa 3 · Challenge 2 · MEIA/ISEP 2026/27 · Par B (Rômulo Semião Arantes, João Vitor Carneiro Moreira)**
**Versão:** v0.1 (18/11/2026) · **Estado:** proposta para discussão com o João Vitor e os docentes

> **Convenção de etiquetas usada em todo o documento**
> - **[F]** facto verificado (com a fonte indicada).
> - **[P]** pressuposto do projeto (vindo do enunciado/`CONTEXT.md`/`DATA.md` ou assumido por nós e a justificar no paper).
> - **[S]** sugestão minha, sujeita a decisão.
> - **[A DECIDIR]** ponto em aberto (lista na secção 11).

---

## 0. Estado da verificação do dataset (ler primeiro)

| Item | Estado |
|---|---|
| Ficheiro do dataset em `data/dataset2/` | **[F]** Não existe: a pasta só tem `.gitkeep`. Nada do que se segue foi validado contra os dados reais. |
| Página oficial do Kaggle | **Não consegui abri-la** nesta sessão. Os campos abaixo vêm do vosso `DATA.md` e do enunciado, **não** foram confirmados na fonte. |
| Fonte secundária | **[F, secundário]** Uma análise pública em Excel de um estudante (repositório GitHub "ZARA_SALES_ANALYSIS", encontrado por pesquisa) descreve o dataset como **252 produtos**, CSV do Kaggle, **dados transversais (sem dimensão temporal)**. Não é fonte primária: confirmar com `df.shape` e `df.dtypes`. |
| Campos | **[P]** `Product ID, Product Name, Description, Product Category/Section, Price, Currency, Sales Volume, Promotion (binária), Seasonal, Product Position`. |

**Primeira tarefa de código (antes de qualquer modelo):** auditoria do dataset (`notebooks/exploratory_data_analysis/00_audit.ipynb`). Questões que condicionam toda a arquitetura:

1. Quantas linhas? Há duplicados de `Product Name`/`Description` (variantes da mesma peça)?
2. `Price` é preço de tabela ou preço já com promoção aplicada quando `Promotion = Yes`? (condiciona a estimação da elasticidade, ver 3.4.)
3. `Sales Volume` cobre que período? É unidades ou valor?
4. `Seasonal` e `Product Position` — cardinalidade e distribuição; há células vazias?
5. Existem custo, stock ou datas? (Pelo vosso `DATA.md`, não.)

Se (1) der um N muito pequeno, a escolha de modelos (secção 3.3) simplifica-se ainda mais.

---

## 1. Visão geral

**Problema.** Dado um conjunto de peças de uma coleção de moda com stock inicial, decidir *que desconto aplicar em cada semana* de modo a maximizar receita e libertação de capital até ao fim da coleção, respeitando desconto máximo, margem mínima, cadência mínima entre reduções e capacidade de armazém.

**Limitação estrutural [F, secundário + P].** O dataset é transversal e sem stock, custo, dias em loja nem % de desconto. Logo:

- A camada preditiva **não** aprende uma dinâmica temporal; aprende **um nível de procura por produto** e um **efeito médio de promoção** (heterogéneo por categoria).
- A dimensão temporal (semanas, decaimento da procura, stock, margem) é **simulada** com pressupostos explícitos (secção 5), e todos os resultados do paper têm de ser apresentados como **resultados sobre um ambiente simulado calibrado com dados reais**, nunca como ganho real de negócio.

**Posicionamento científico [S].** A contribuição do Par B não é "prever melhor o volume de vendas da Zara" (dataset pequeno), mas sim **a interface ML → otimização sob incerteza forte nos parâmetros** (elasticidade não identificada de forma causal): como propagar a incerteza do preditor para a política de markdown e quão robusta é essa política. Isto cola-se diretamente à pergunta integradora.

```
 Zara CSV (N produtos)
        │  data_loader
        ▼
 ┌─────────────────────────────┐
 │ CAMADA PREDITIVA (AAUTIA)   │  φ_i → λ̂_i (procura base), ε̂_i (elasticidade), r̂_i (risco de encalhe)
 │  baselines + GLM + GBM      │  + incerteza (intervalos / bootstrap)
 └─────────────┬───────────────┘
               │  tensor Q̂[i,t,k] (+ quantis)   ← INTERFACE
               ▼
 ┌─────────────────────────────┐
 │ SIMULADOR (pressupostos A1–A9)│  stock, decaimento g(t), procura ~ NegBin
 └─────────────┬───────────────┘
               ▼
 ┌─────────────────────────────┐
 │ CAMADA PRESCRITIVA (PLNTDIA)│  MILP (principal) · DP exato (validação)
 │                             │  GA/memético · SA/Tabu (comparação)
 │                             │  TOPSIS/AHP (escolha final)
 └─────────────┬───────────────┘
               ▼
   Política de markdown π*: k_{i,t}  +  métricas  +  demo
```

---

## 2. Formulação do problema (espaço de estados / otimização)

### 2.1 Notação (usar igual no paper, no código e nas Tarefas 2–3)

| Símbolo | Significado | Origem |
|---|---|---|
| `i ∈ 𝓘`, `N = |𝓘|` | produtos (peças) | dados |
| `t ∈ {1,…,T}` | semana da coleção; `T` = horizonte | [P] `T = 12` base; cenários 8/12/16 |
| `k ∈ {0,…,K}` | índice do escalão de desconto, `d_0 = 0` | [P] `𝒟 = {0,10,20,30,40,50}%` (`K=5`) |
| `p_i` | preço de tabela | dados (`Price`) |
| `c_i = ρ·p_i` | custo unitário | [P] `ρ = 0,5` base; cenários 0,4–0,6 |
| `S_i` | stock inicial (unidades) | [P] simulado (A3) |
| `V_i` | `Sales Volume` observado | dados |
| `φ_i` | vetor de atributos (Category/Section, Seasonal, Position, Promotion, Price, texto…) | dados |
| `λ_i` | procura base semanal a preço de tabela | estimada (camada preditiva) |
| `ε_i` | elasticidade-preço (constante por segmento) | estimada + [P] (A4) |
| `g(t)` | decaimento da procura com a idade da peça | [P] `g(t)=exp(−δ(t−1))` (A5) |
| `q̂_{i,t}(k)` | procura média prevista na semana `t` com escalão `k` | **interface** |
| `D_{i,t}` | procura aleatória, `D ~ NegBin(q̂, ϑ)` | simulador |
| `y_{i,t}` | vendas, `y = min(I, D)` | simulador |
| `I_{i,t}` | stock no início da semana `t` | estado |
| `h` | custo de posse por semana (fração do custo) | [P] |
| `γ ∈ (0,1]` | fator de desconto semanal (valor temporal do capital) | [P] |
| `σ` | valor residual unitário no fim (fração de `p_i`) | [P] |
| `m_min` | margem mínima sobre o custo | enunciado + [P] |
| `d_max` | desconto máximo | enunciado + [P] |
| `c_min` | cadência mínima (semanas entre reduções) | enunciado + [P] |
| `W_t` | capacidade de armazém admissível na semana `t` | [P] (A8) |

### 2.2 Espaço de estados (por produto; acoplado entre produtos só por `W_t`)

- **Estado:** `s_{i,t} = (I_{i,t}, k_{i,t−1}, τ_{i,t})`, com `τ` = semanas desde a última redução (e `t` implícito).
- **Estado inicial:** `s_{i,1} = (S_i, 0, ∞)`.
- **Ações:** `a_{i,t} = k_{i,t} ∈ 𝒦_i(s)`, o conjunto admissível dado pelas restrições abaixo.
- **Transição:** `I_{i,t+1} = I_{i,t} − y_{i,t}`, com `y_{i,t} = min(I_{i,t}, D_{i,t}(k_{i,t}))`.
- **Teste de objetivo / terminal:** `t = T+1`.
- **Custo de caminho (a minimizar) = −lucro descontado.**

### 2.3 Função objetivo

Maximizar o valor esperado

```
J(π) = E[ Σ_i ( Σ_{t=1..T} γ^t · ( p_i·(1−d_{k_{i,t}})·y_{i,t}  −  h·c_i·I_{i,t} )  +  γ^T · σ·p_i·I_{i,T+1} ) ]
```

Receita descontada, menos custo de posse do inventário, mais valor residual do stock encalhado. A "libertação de capital" entra via `γ` e `h`, e é reportada também como métrica própria (secção 7). Variante multiobjetivo [S]: `J_β = Receita + β·LibertaçãoDeCapital`, com varrimento de `β` para gerar a fronteira de Pareto usada em TOPSIS.

### 2.4 Restrições

| # | Restrição | Forma |
|---|---|---|
| C1 | desconto máximo | `d_{k} ≤ d_max` |
| C2 | margem mínima | `p_i(1−d_k) ≥ (1+m_min)·c_i` ⇒ define `𝒦_i` por produto |
| C3 | só descidas de preço | `k_{i,t+1} ≥ k_{i,t}` |
| C4 | cadência mínima | se `k_{i,t} > k_{i,t−1}`, então `k_{i,t'} = k_{i,t}` para `t' ∈ [t, t+c_min−1]` |
| C5 | espaço de armazém | `Σ_i I_{i,t} ≤ W_t` nos pontos de controlo (restrição **mole** com penalização, para evitar infeasibilidade) |

**Dimensão do espaço de pesquisa [F, cálculo].** Por produto, o número de trajetórias monótonas `k_1 ≤ … ≤ k_T` é `C(T+K, K)`; com `T=12`, `K=5`: `C(17,5) = 6188` (ainda menos com C4). O espaço total é exponencial em `N` (`6188^N`), mas **o problema só se acopla entre produtos por C5**. Consequência importante: sem C5 cada produto resolve-se **exatamente** por enumeração/DP. Isto tem de ser dito com honestidade no paper: as metaheurísticas não são necessárias por razões de tratabilidade; justificam-se pela comparação metodológica e pelas extensões (procura estocástica, preditor como caixa-preta, objetivos não lineares).

---

## 3. Camada preditiva (AAUTIA)

### 3.1 Alvos

| Alvo | Tipo | Definição | Nota |
|---|---|---|---|
| `Y1 = Sales Volume` | regressão (contagem, assimétrica) | `log1p(V_i)` ou modelo Poisson/NegBin | alvo principal |
| `Y2` = efeito de promoção | estimativa de parâmetro | `β_promo` (heterogéneo por segmento) | alimenta `ε` |
| `Y3` = risco de encalhe | **proxy** | `r_i = P(S_i > Σ_t D_{i,t}(k=0))`, calculado a partir de `Y1` + simulador (não é rótulo observado) | não treinar classificador sobre um rótulo inventado sem o dizer; se se quiser classificador, rótulo = `V_i` abaixo do quartil inferior da sua categoria **[S]** e declarar como proxy |

### 3.2 Engenharia de atributos

- **Categóricas:** `Category/Section`, `Seasonal`, `Product Position` (one-hot ou target encoding **dentro** de cada fold).
- **Binária:** `Promotion`.
- **Numéricas:** `log(Price)`; `Price` relativo à mediana da categoria; `Price` relativo ao quantil de preço na categoria.
- **Interações [S]:** `Promotion × Category`, `Promotion × Seasonal`, `Position × Category`.
- **Texto [S, opcional]:** `Description`/`Product Name` → TF-IDF reduzido por SVD, ou palavras-chave (tipo de peça, material, cor), apenas se a auditoria mostrar texto informativo e N suficiente.
- **Grupos:** chave de grupo `g_i` = *stem* do `Product Name` (para validar sem fuga entre variantes).

### 3.3 Modelos candidatos

| Papel | Modelo | Porquê | Risco |
|---|---|---|---|
| Baseline 0 | mediana/média de `V` por categoria | referência mínima | — |
| Baseline 1 | Regressão linear/Ridge em `log1p(V)` | interpretável; dá `β_promo` | — |
| Candidato A | **GLM Poisson/NegBin** com regularização | natural para contagens; coeficientes interpretáveis para a elasticidade | sobredispersão |
| Candidato B | **Random Forest** | robusto com N pequeno, sem tuning pesado | extrapola mal em `Price` |
| Candidato C | **LightGBM/XGBoost** (parâmetros fortemente regularizados) | estado da prática em tabular | overfitting com N pequeno |

**Recomendação [S]:** não ir além destes. Com poucas centenas de linhas, redes neuronais não se justificam. Reportar sempre o ganho **face ao baseline** e com intervalos de confiança; é provável que o GLM fique competitivo com os GBM, e isso é um resultado legítimo.

### 3.4 Elasticidade com promoção binária (ponto crítico)

O dataset diz apenas *se* há promoção, não *quanto*. Procedimento proposto:

1. **Efeito de promoção estimado:** `log V_i = β_0 + β_promo·Promo_i + θᵀφ_i + u_i` (ou GLM log-link). Lift multiplicativo `L = exp(β_promo)`. Heterogeneidade: `β_promo` por categoria/secção (efeitos aleatórios ou interações regularizadas).
2. **Mapeamento para elasticidade** com procura de elasticidade constante `q(p) = q_0·(p/p_0)^{−ε}`:

   ```
   L = (1 − d̄)^{−ε}   ⇒   ε = β_promo / ( −ln(1 − d̄) )
   ```

   onde `d̄` é a **profundidade média de desconto subjacente à flag**, **desconhecida** [P/A4]. Tratar `d̄ ∈ {10, 20, 30, 40}%` como **parâmetro de cenário** e fazer análise de sensibilidade de `ε` e da política a `d̄`.
3. **Corte transversal em `Price`** (variação de preço entre produtos): útil como verificação cruzada, mas **confundido** (preço correlaciona com qualidade/categoria). Usar apenas com efeitos fixos de categoria e não como estimativa causal.
4. **Identificação causal:** `Promotion` não foi aleatorizada e `Product Position` pode ter sido atribuída *em função* da procura esperada (causalidade inversa). **[S]** Reportar como efeito **associativo**; opcionalmente comparar com DML/T-learner apenas como análise de robustez, sem prometer causalidade.
5. **Se `Price` já é o preço promocional** quando `Promotion = 1`, parte do efeito está em `Price`: ver questão 2 da auditoria; muda a especificação.

### 3.5 Validação (dados transversais, sem tempo)

- **Não há fuga temporal** (não há tempo), mas há **outros riscos de fuga**:
  - variantes do mesmo produto em folds diferentes → **`GroupKFold`** (ou `StratifiedGroupKFold`) pelo *stem* do nome;
  - target encoding fora do fold → fazer **dentro** de cada fold (Pipeline);
  - qualquer atributo derivado de `Sales Volume` (incluindo o rótulo de risco) não pode entrar nas *features*.
- **N pequeno:** validação cruzada **repetida** (ex.: 5×10 folds por grupo), conjunto de *hold-out* só no fim, e **bootstrap** para ICs.
- **Métricas:** MAE, RMSE, RMSLE, WAPE, R² (em escala log e original); calibração dos intervalos (cobertura empírica); para `ε`/`β_promo`: IC bootstrap.

---

## 4. Camada prescritiva (PLNTDIA)

### 4.1 Baselines de comparação (obrigatórios)

| ID | Política | Descrição |
|---|---|---|
| B0 | Sem desconto | `k=0` sempre |
| B1 | Desconto fixo | `k=k̄` a partir da semana `t_0` (ex.: `t_0 = ⌈T/2⌉`; varrer `k̄`) |
| B2 | **Regra heurística por sell-through** | se `ST_t < alvo(t)`, subir um escalão (respeitando C1–C4); é o baseline "de negócio" |
| B3 | Guloso míope | em cada semana escolhe o `k` que maximiza o lucro dessa semana |

### 4.2 Abordagens comparadas

| Abordagem | Como se aplica aqui | Prós | Contras |
|---|---|---|---|
| **A. MILP/ILP (OR-Tools)** | variáveis `x_{i,t,k}` binárias; procura tabelada `q̂_{i,t,k}` → modelo **linear** (ver 4.3) | ótimo **global** com garantia (gap); restrições C1–C5 nativas; interpretável; encaixa no stack previsto | assume procura determinística (ou cenários); modelo cresce com `N·T·K`; preditor tem de ser tabelado |
| **D. DP exata por produto (validação)** | estado `(t, I, k, τ)` por produto; ou enumeração das 6188 trajetórias | exata, simples, serve de **prova de ótimo** para validar o MILP e as metaheurísticas | não trata C5 (coupling) sem relaxação Lagrangiana |
| **B. Algoritmo Genético / memético** | cromossoma por produto: `T` genes em `{0..K}` (ou pontos de mudança + níveis); operador de reparação para C3/C4; *fitness* = `J` simulado − penalizações; variante **memética** com pesquisa local | usa o preditor como caixa-preta e a simulação estocástica diretamente; lida com não linearidades; mapeia para a matéria de PLNTDIA | sem garantia de ótimo; muitos hiperparâmetros; custo de simulação |
| **C. SA / Pesquisa Tabu** | vizinhança: deslocar a semana de uma redução ±1, subir/descer um escalão; Tabu com memória de movimentos recentes `(i,t)` | simples, rápido, bom para refinar uma solução; bom ponto de comparação com GA | sensível ao arrefecimento/lista tabu; sem garantias |
| **E. AHP/TOPSIS** | **não otimiza trajetórias**: ordena políticas/pontos de Pareto por critérios (receita, libertação de capital, margem, stock final, nº de mudanças, risco P5) com pesos via AHP | camada de decisão multicritério explícita (DSS); transparente | pesos subjetivos → têm de vir do negócio/docentes; é complemento, não alternativa ao A |

### 4.3 Formulação MILP (abordagem A)

```
max  Σ_i Σ_t Σ_k γ^t · p_i(1−d_k) · y_{i,t,k}
   − Σ_i Σ_t γ^t · h·c_i · I_{i,t}
   + Σ_i γ^T · σ·p_i · I_{i,T+1}
   − M · Σ_t s_t                                   (penalização de C5)

s.t.  Σ_k x_{i,t,k} = 1                              ∀ i,t
      y_{i,t,k} ≤ q̂_{i,t,k} · x_{i,t,k}              ∀ i,t,k   (procura se escalão k ativo)
      Σ_k y_{i,t,k} ≤ I_{i,t}                         ∀ i,t
      I_{i,t+1} = I_{i,t} − Σ_k y_{i,t,k};  I_{i,1} = S_i
      x_{i,t,k} = 0  para k ∉ 𝒦_i                      (C1, C2)
      Σ_k k·x_{i,t+1,k} ≥ Σ_k k·x_{i,t,k}              (C3)
      Σ_k k·x_{i,t,k} − Σ_k k·x_{i,t−1,k} ≤ K·z_{i,t}   (z = indicador de mudança)
      z_{i,t} + z_{i,t'} ≤ 1   ∀ t' ∈ (t, t+c_min−1]    (C4)
      Σ_i I_{i,t} ≤ W_t + s_t,  s_t ≥ 0                (C5, mole)
```

A linearidade só é possível porque `q̂_{i,t,k}` é uma **tabela pré-calculada** (a procura depende de `k`, não do stock). Ordem de grandeza: `252 · 12 · 6 ≈ 1,8×10⁴` binárias **se N=252 se confirmar**: tratável. Solvers: `pywraplp` (SCIP/CBC) ou CP-SAT com preços em cêntimos inteiros **[A DECIDIR]**.

**Robustez [S]:** além do caso determinístico (média), resolver com procura no quantil baixo (`q̂_{0,2}`) e com amostragem de cenários (formulação por cenários com `y` por cenário e variáveis `x` partilhadas) para quantificar o valor da incerteza.

### 4.4 Recomendação

**Principal: MILP (A)**, validado contra **DP exata (D)** (mesma solução ótima no caso sem C5 — teste de correção), **comparado com GA/memético (B)** e **SA/Tabu (C)** sobre o **mesmo simulador estocástico**, e com **TOPSIS/AHP (E)** como camada final para escolher entre políticas da fronteira de Pareto. Justificação para o paper: (i) o problema, estruturalmente, é um MILP de dimensão moderada com restrições lineares do enunciado; (ii) dá garantia de otimalidade e *gap*; (iii) as metaheurísticas respondem à pergunta "o que se perde/ganha quando o preditor é caixa-preta e a procura é estocástica?"; (iv) AHP/TOPSIS cobre o lado de decisão multicritério e DSS da UC.

**Extensão opcional [S]:** extrair **regras interpretáveis** da política ótima com uma **árvore de decisão** (entradas: idade, sell-through, categoria, `Seasonal`; saída: escalão), para ligar a "árvores de decisão" e produzir um DSS legível.

---

## 5. Interface entre camadas e simulação

### 5.1 Interface (contrato de código)

```python
# src/predictors/demand_model.py
class DemandModel:
    def fit(self, df): ...
    def predict_tensor(self, df, T, levels):
        """-> Q[i,t,k] (média), Q_lo[i,t,k], Q_hi[i,t,k] (quantis 10/90)"""
    def elasticity(self, df):        # ε_i ou ε por segmento
    def stranding_risk(self, df, S): # r_i = P(S_i > Σ_t D_it(k=0))
```

- `Q[i,t,k] = λ̂_i · g(t) · (1−d_k)^{−ε̂_i}` no caso paramétrico (GLM); com GBM, `λ̂_i` vem do modelo e `ε̂_i` do módulo de elasticidade.
- O MILP consome `Q`; GA/SA/Tabu consomem `Q` **ou** o simulador.

### 5.2 Fuga entre camadas e viés do otimizador [S, importante]

Se otimizamos e avaliamos sobre o **mesmo** `Q̂`, o resultado é otimista (*optimizer's curse*). Protocolo:

- **Otimizar** com o modelo (parâmetros estimados).
- **Avaliar** num **ambiente "verdadeiro"** separado: parâmetros (`ε`, `d̄`, `δ`, ruído) sorteados de distribuições diferentes (posterior/bootstrap), com **sementes comuns** entre políticas (*common random numbers*) e `R = 1000` réplicas Monte Carlo.
- Reportar média, desvio, IC bootstrap e, para a robustez, `P5`/CVaR da receita.

### 5.3 Pressupostos a justificar no paper

| ID | Pressuposto | Valor base | Sensibilidade |
|---|---|---|---|
| A1 | `V_i` = vendas totais sob o regime observado ao longo de uma janela de coleção `T` | — | — |
| A2 | Procura semanal base a preço de tabela: `λ_i = V_i / (T·L^{Promo_i}·ḡ)` | — | via `L`, `ḡ` |
| A3 | Stock inicial `S_i = ⌈κ·V̂_i^{tabela}⌉` | `κ=1,5` | `κ ∈ {1,2; 1,5; 2,0}` |
| A4 | Profundidade média de desconto por trás da flag `d̄` | 20–30 % | `{10,20,30,40}%` |
| A5 | Decaimento da procura `g(t)=exp(−δ(t−1))` | `δ` a calibrar para peças `Seasonal` vs não | `δ ∈ {0; 0,05; 0,10}` |
| A6 | Custo `c_i = ρ·p_i` | `ρ=0,5` | `{0,4; 0,5; 0,6}` |
| A7 | `h` (posse), `γ`, `σ` | `h=0,5 %/sem`, `γ=0,99`, `σ=0,2` | ± |
| A8 | Capacidade `W_t` decrescente (espaço para a coleção seguinte) | `W_t = ω_t·ΣS_i`, `ω_T=0,2` | `ω_T ∈ {0,1; 0,2; 0,3}` |
| A9 | Ruído de procura `NegBin` com dispersão `ϑ` estimada nos resíduos | — | `ϑ` |
| A10 | Horizonte e escalões | `T=12`, `𝒟={0,…,50}%` | `T∈{8,12,16}` |

Todos os valores numéricos acima são **escolhas nossas [P/S]**, sem base empírica no dataset; ancorar na literatura quando a Tarefa 3 ficar feita e **dizê-lo** no paper.

### 5.4 Riscos e limitações

1. **Sem série temporal real:** conclusões dependem de A1–A10; mitigação = sensibilidade sistemática (E4).
2. **Elasticidade não identificada** (promoção binária, não aleatória, posição endógena): resultados em termos de política *condicional* a `ε`.
3. **N pequeno:** variância alta; modelos simples; ICs.
4. **Circularidade** simulador/preditor (secção 5.2).
5. **Generalização:** Zara ≠ todo o *fast fashion*; usar H&M (Par A) e ASOS/Myntra (Par C) como contraste quando houver ligação entre pares.
6. **Preço promocional vs tabela** ambíguo (auditoria, questão 2).

---

## 6. Plano de experiências e critérios de sucesso

| ID | Experiência | Saída |
|---|---|---|
| E0 | Auditoria e EDA | quadro de qualidade, distribuições, correlações, decisão sobre texto |
| E1 | Comparação de preditores (baseline, GLM, RF, GBM) | tabela MAE/RMSE/RMSLE/WAPE ± IC (CV repetida por grupos) |
| E2 | Elasticidade `β_promo` por segmento e `ε(d̄)` | gráfico de forest plot + curva `ε` vs `d̄` |
| E3 | Políticas B0–B3 vs MILP vs GA vs SA/Tabu | tabela de `J`, receita, margem, ST, stock final, capital libertado, nº mudanças, tempo, gap |
| E4 | Sensibilidade a `d̄, ε, κ, ρ, δ, T, ω_T` | tornado/heatmaps; mapa de regimes onde cada política vence |
| E5 | Robustez estocástica e erro do preditor | `P5`/CVaR, degradação da política vs erro injetado |
| E6 | TOPSIS/AHP sobre a fronteira de Pareto | ranking de políticas, análise de sensibilidade dos pesos |
| E7 | Validação de correção (MILP vs DP exata) | igualdade de ótimo em instâncias sem C5 |

**Critérios de sucesso [S] (a combinar):**

- E1: pelo menos um modelo melhora o baseline em MAE/RMSLE com IC que exclui zero; se não melhorar, reportar e usar o modelo mais simples.
- E3: MILP supera B2 em `J` com IC bootstrap > 0 na maioria dos cenários de E4 (ex.: ≥ 75 %); nenhuma violação de C1–C4 em nenhuma política.
- E7: gap = 0 nas instâncias de validação.
- E5: queda de `J` para o quantil baixo da procura limitada (limite a fixar com os docentes).

---

## 7. Métricas

- **Receita** `R`, **margem** `M = R − custo das unidades vendidas`, **`J`** (objetivo).
- **Sell-through** `ST = unidades vendidas / S_i` (por produto e agregado), **stock final**, **semanas até `ST ≥ 80 %`**.
- **Libertação de capital:** `Σ_t γ^t·(custo das unidades vendidas)` e capital imobilizado médio `Σ_t c_i·I_{i,t}/T`.
- **Profundidade média de desconto**, **nº de reduções por produto**.
- **Risco:** `P5` e CVaR de `J` sobre as `R` réplicas.
- **Computacional:** tempo, gap do MILP, nº de avaliações do simulador (GA/SA).

---

## 8. Mapeamento para o repositório

```
data/dataset2/                         # CSV Zara (ignorado pelo Git)
notebooks/exploratory_data_analysis/
    00_audit.ipynb                     # E0
    01_eda_promocao_sazonalidade.ipynb
notebooks/ml_models/
    10_baselines.ipynb  11_glm.ipynb  12_gbm.ipynb  13_elasticity.ipynb    # E1, E2
notebooks/decision_optimization/
    20_simulator_check.ipynb  21_milp.ipynb  22_ga_sa_tabu.ipynb
    23_topsis_ahp.ipynb  24_experiments_sensitivity.ipynb                   # E3–E7
src/data_loader/zara_loader.py         # leitura, limpeza, chave de grupo, φ_i
src/predictors/
    baselines.py  demand_model.py  elasticity.py  evaluation.py
src/optimizers/
    problem.py          # Instance, Policy, restrições C1–C5 (verificador)
    simulator.py        # NegBin, g(t), stock, CRN         [S: ou src/simulation/]
    baselines.py        # B0–B3
    dp_exact.py  milp.py  ga.py  sa_tabu.py  topsis.py
paper/state_of_the_art/                # Tarefa 3 (fichas + .bib)
paper/draft/                           # Tarefa 2 (main.tex, secções)
reports/                               # este documento, resultados, figuras
```

Stack: `pandas, scikit-learn, xgboost, lightgbm, ortools, matplotlib/seaborn/plotly`, mais `statsmodels` (GLM Poisson/NegBin) e `scipy` **[S]**.

---

## 9. Plano de demo (11 jan, 30 min) e pitch (14 jan, 5 min)

**Demo [S], 30 min** (de equipa; o Par B contribui com o módulo Zara e a interface comum):

| Min | Conteúdo |
|---|---|
| 0–3 | Problema, dilema e pergunta integradora |
| 3–8 | Dados Zara + limitações + pressupostos (honestidade como mensagem) |
| 8–14 | Camada preditiva: resultados E1/E2 |
| 14–24 | **Demo ao vivo:** escolher categoria/produto; ajustar `ε`/`d̄`, `κ`, `d_max`, `m_min`, `T`; gerar política MILP vs B2 vs GA; gráficos de stock, preço e receita |
| 24–28 | Sensibilidade, robustez e TOPSIS |
| 28–30 | Limitações, trabalho futuro, perguntas |

Formato: notebook Jupyter limpo (widgets/plotly) ou, se a equipa quiser, Streamlit **[A DECIDIR]**. Preparar **resultados pré-calculados** como plano B e fixar sementes.

**Pitch 5 min [S]:** 0:00 problema e dilema · 0:45 abordagem em duas camadas (ML → decisão) · 1:45 o que descobrimos (1 gráfico: política MILP vs heurística) · 3:30 robustez e limitações · 4:30 mensagem final e próximos passos. Um único *slide* de arquitetura e um de resultados.

---

## 10. Calendário sugerido até à 1.ª versão (18 dez) [S]

| Semana | Foco |
|---|---|
| 18–24 nov | Auditoria do dataset (E0); decisões da secção 11 |
| 25 nov–1 dez | Tarefa 3 (estado da arte) + EDA |
| 2–8 dez | Esqueleto do paper (Tarefa 2), secções de estado da arte; baselines e GLM |
| 9–15 dez | Formulação e simulador; MILP mínimo (1 categoria) |
| 16–18 dez | Revisão contra a grelha, referências, entrega v1 |

Janeiro: E3–E7, paper final (10 jan), demo (11 jan), pitch (14 jan).

---

## 11. Decisões em aberto

**Com o João Vitor (Par B)**
1. Divisão de trabalho: quem fica com a camada preditiva e quem com a prescritiva (sugestão: um lidera cada camada e rotulam-se mutuamente o código).
2. Escalões `𝒟`, `T`, e valores base de A3–A8 (secção 5.3).
3. Solver (`pywraplp`/SCIP vs CP-SAT) e interface da demo (Jupyter vs Streamlit).
4. Critério de sucesso (secção 6).

**Com os docentes**
5. Aceitam um ambiente **simulado** calibrado no dataset (dado que o Zara não tem tempo, stock ou desconto %)? O `CONTEXT.md` pede dados com datas e inventário, que este dataset não tem.
6. Quais pressupostos de custo/margem e pesos AHP são aceitáveis para o enunciado?
7. Há um *template* de paper obrigatório e um limite de páginas?
8. Podemos complementar o Zara com um dataset auxiliar (H&M do Par A ou ASOS/Myntra do Par C) só para calibrar curvas de desconto?

**Com a equipa (Pares A/B/C)**
9. **Formulação e notação comuns** (secção 2) para o paper e para a demo conjunta; interface única `DemandModel`.
10. Como se integram os três datasets na demo de 30 min.

---

## 12. Resumo da fase (Tarefa 1)

- A arquitetura é **ML → tabela de procura `Q̂[i,t,k]` → otimização**, com **simulador** que supre a falta de tempo/stock/desconto % do dataset.
- Prescritiva: **MILP principal**, validado por **DP exata**, comparado com **GA/memético e SA/Tabu**, e com **TOPSIS/AHP** para escolher a política final.
- O resultado depende de pressupostos explícitos (A1–A10) e de análise de sensibilidade; **os campos do dataset ainda têm de ser confirmados** (E0).
