# Auditoria do Dataset 2 (Zara) — Par B

**Equipa 3 · Challenge 2 · MEIA/ISEP 2026/27 · Par B (Rômulo Semião Arantes, João Vitor Carneiro Moreira)**
**Versão:** v0.1 (23/11/2026) · **Complementa:** [arquitetura_par_b.md](arquitetura_par_b.md) (secção 0, experiência E0)

> Etiquetas como na arquitetura: **[F]** facto verificado nos dados · **[P]** pressuposto · **[S]** sugestão · **[A DECIDIR]**.

---

## 0. Resumo

1. **[F]** O ficheiro tem **252 linhas × 16 colunas**, separador `;`, uma única marca (Zara), uma moeda (USD), e foi recolhido (*scraping*) no site Zara US a **19/02/2024, numa janela de 20 minutos**. Não há dimensão temporal de vendas.
2. **[F]** O dataset é a junção de **duas partes de natureza diferente**:
   - **Parte real (scraping):** `url, sku, name, description, price, currency, scraped_at, terms, section`;
   - **Parte "de negócio":** `Product ID, Product Position, Promotion, Product Category, Seasonal, Sales Volume`.
3. **[F]** A parte de negócio **não tem qualquer sinal estatístico**: `Sales Volume` é compatível com uma **uniforme inteira entre ~500 e ~3000** e é **independente** de `Promotion`, `Seasonal`, `Product Position`, `section`, `terms` e `price` (todos os testes de permutação com p > 0,69). Um modelo linear com todas as variáveis é **pior** do que prever a média em validação cruzada.
4. **[F]** O mesmo SKU/URL aparece com `Sales Volume`, `Promotion` e `Product Position` diferentes. Estas colunas foram, com grande probabilidade, **geradas aleatoriamente** e coladas a um catálogo real.
5. **Consequência:** a cadeia da arquitetura v0.1 *"aprender `λ̂` e `β_promo` com os dados → `ε̂` → MILP"* **não funciona**: o efeito de promoção estimado é ≈ 0, logo `ε̂ ≈ 0`, e a política ótima trivial seria *nunca descontar*.
6. **Proposta [S]**, alinhada com a indicação do professor (treino de modelo pequeno, foco no planeamento/decisão): usar o Zara como **gerador de instâncias reais** (preços, tipo de peça, secção) para o problema de otimização; tirar a resposta da procura ao desconto da **literatura e/ou do dataset do Par C** (que tem `%` de desconto); e pôr a Aprendizagem Automática **ao serviço do planeamento** (aprender regras de markdown a partir das soluções ótimas). Detalhes na secção 5.

---

## 1. Fonte e reprodutibilidade

| Item | Estado |
|---|---|
| Ficheiro auditado | `data/dataset2/zara_sales_github_copy.csv` (ignorado pelo Git) |
| Origem | Cópia em `github.com/ThalithaThomas/ZARA_SALES_ANALYSIS` (`zara 2.csv`), que diz vir do Kaggle |
| SHA-256 | `2c364c86b37bad13cc4baf37037f999b5a90f1468a3fae2e8e53366e44b081ec` |
| Página Kaggle provável | `kaggle.com/datasets/xontoloyo/data-penjualan-zara` (existe, mas não foi possível ler o conteúdo; `.../zara-sales-for-eda` dá 404) |
| **Ação pendente** | Descarregar o original do Kaggle (com conta) e confirmar `shape` e *hash*. Se o original for diferente desta cópia, repetir a auditoria. Até lá, **citar no paper o Kaggle e não o GitHub**. |

Ferramentas: Python 3 (biblioteca padrão); testes de permutação com 2000 permutações, semente 0.

---

## 2. Esquema real vs. `DATA.md`

O esquema descrito no `DATA.md` e na arquitetura (secção 0) **não corresponde** ao ficheiro.

| Coluna (verbatim) | Tipo | Únicos | Vazios | Observações |
|---|---|---|---|---|
| `Product ID` | inteiro | 252 | 0 | 110075–199631; não é chave do produto real (ver 3.3) |
| `Product Position` | categórica | 3 | 0 | `Aisle` 97 · `End-cap` 86 · `Front of Store` 69 |
| `Promotion` | binária | 2 | 0 | `No` 132 · `Yes` 120 |
| `Product Category` | constante | 1 | 0 | `Clothing` em todas as linhas → **sem informação** |
| `Seasonal` | binária | 2 | 0 | `Yes` 128 · `No` 124 |
| `Sales Volume` | inteiro | 234 | 0 | 529–2989; média 1824; desvio 698; sem unidade nem período |
| `brand` | constante | 1 | 0 | `Zara` |
| `url` | texto | 228 | 0 | 24 URLs repetidos |
| `sku` | texto | 228 | 0 | 24 SKUs repetidos (os mesmos do URL) |
| `name` | texto | 195 | 1 | 1 linha sem nome |
| `description` | texto | 222 | 2 | mediana de 130 caracteres |
| `price` | decimal | 28 | 0 | 7,99–439,00 USD; mediana 79,90; só 28 preços distintos |
| `currency` | constante | 1 | 0 | `USD` |
| `scraped_at` | data-hora | 229 | 0 | 2024-02-19 08:50 → 09:10 (momento da recolha, **não** data de venda) |
| `terms` | categórica | 5 | 0 | jackets 140 · sweaters 41 · t-shirts 32 · shoes 31 · jeans 8 |
| `section` | categórica | 2 | 0 | MAN 218 · WOMAN 34 |

**Diferenças face ao `DATA.md`:** não existe `Product Name` (é `name`); `Product Category` é constante, e a segmentação útil está em `terms` (tipo de peça) e `section` (género); existem `url`, `sku`, `brand`, `scraped_at` e `terms`, que não estão documentados. Não existem custo, stock, datas de venda nem percentagem de desconto (confirma o que a arquitetura assumia).

---

## 3. Qualidade dos dados

### 3.1 Duplicados e incoerências [F]

- **24 pares** de linhas partilham `url`/`sku`, e **em todos os 24 o `name` é diferente**. Exemplo: o mesmo SKU aparece como "PLAID OVERSHIRT" e "HERRINGBONE TEXTURED JACKET".
- **~20 linhas** têm um nome que não corresponde ao URL. Exemplo: `name = LEATHER BIKER JACKET`, mas o URL é `wool-blend-knit-sweater`. Há um desalinhamento entre colunas da parte de *scraping*.
- **`terms` incoerente com o nome** em pelo menos 17 linhas. Exemplo: 14 linhas com `terms = t-shirts` cujo nome é JACKET/OVERSHIRT/BLAZER.
- Linhas com nome vazio: 1; descrição vazia: 2.

### 3.2 Preço [F]

- Os preços são de tabela do site (terminações `.90`, `.00`, `.99`) e há apenas 28 valores distintos.
- O preço médio é ligeiramente mais alto com `Promotion = Yes` (92 vs 81 USD), o que é o contrário do que aconteceria se `price` fosse o preço já com desconto. **Resposta à questão 2 da auditoria (arquitetura, secção 0):** `price` é o preço de catálogo à data da recolha; não há indicação de que inclua desconto.

### 3.3 `Product ID` vs. produto real [F]

O mesmo SKU tem `Product ID`s diferentes e, dentro de cada par duplicado, `Sales Volume` difere em 24/24, `Promotion` em 11/24 e `Product Position` em 18/24. Uma peça real não pode ter, ao mesmo tempo, duas posições em loja e dois volumes de vendas: estas colunas não descrevem o produto do URL.

---

## 4. Sinal estatístico da parte "de negócio"

### 4.1 Distribuição de `Sales Volume` [F]

- Mínimo 529, máximo 2989, assimetria −0,11; o histograma (classes de 250) é aproximadamente plano: 23, 18, 24, 18, 30, 24, 39, 25, 22, 29.
- **Teste KS face a uma Uniforme(500, 3000): D = 0,077 < 0,086** (valor crítico a 5 %) → **não se rejeita a uniformidade**. Vendas reais de moda seguem tipicamente distribuições muito assimétricas (poucos *best-sellers*, cauda longa), não uniformes.

### 4.2 Relação com as variáveis explicativas [F]

| `Sales Volume` explicado por | R² (variância entre grupos) | p (permutação) |
|---|---|---|
| `Promotion` | 0,0000 | 0,94 |
| `Seasonal` | 0,0000 | 0,96 |
| `Product Position` | 0,0029 | 0,73 |
| `section` | 0,0005 | 0,73 |
| `terms` | 0,0089 | 0,69 |
| `Promotion × Position × Seasonal` (12 células) | 0,0248 | 0,86 |
| `price` | corr = −0,07 (log-log: −0,07) | — |

- **Média de vendas com e sem promoção:** 1827 vs 1821 → lift = **1,003**. Dentro de cada tipo de peça o lift varia entre 0,88 e 1,23, com grupos pequenos e sem padrão.
- **As *flags* também são independentes entre si e do produto:** χ²(`Promotion`, `Seasonal`) = 0,00; χ²(`Promotion`, `terms`) = 0,81 (gl = 4); χ²(`Promotion`, `Position`) = 0,10 (gl = 2). A única associação marginal é `Seasonal × terms` (χ² = 11,3, gl = 4, p ≈ 0,02), que não resiste à correção para comparações múltiplas.

### 4.3 Teste preditivo [F]

Validação cruzada com 5 *folds*, alvo `Sales Volume`:

| Modelo | MAE |
|---|---|
| Média do treino (baseline) | **598** |
| Regressão linear (Ridge) com `Promotion, Seasonal, Position, section, terms, log(price)` | 629 |

O modelo com todas as variáveis é **pior** do que a média. Não se espera que RF/GBM mudem esta conclusão, visto que não há sinal marginal nem de interação (tabela 4.2). Isto deve ser confirmado em E1 com o protocolo completo, mas não devem esperar-se surpresas.

### 4.4 Interpretação

As evidências 4.1–4.3 e 3.3, em conjunto, indicam que `Product ID`, `Product Position`, `Promotion`, `Seasonal` e `Sales Volume` foram **geradas sinteticamente, de forma independente e uniforme**, e acrescentadas a um *scraping* real do catálogo Zara US. **[A confirmar]** com a descrição da página Kaggle e, se possível, com o autor do dataset. No paper isto é dito como *"as variáveis de desempenho não apresentam estrutura estatística detetável"*, com os testes acima, e não como acusação.

---

## 5. Impacto na arquitetura e proposta

### 5.1 O que deixa de funcionar na v0.1

| Componente (arquitetura) | Problema |
|---|---|
| 3.1 `Y1 = Sales Volume` | Não é previsível (4.3). Um preditor treinado aprende a média. |
| 3.4 Elasticidade via `β_promo` | `β_promo ≈ 0` ⇒ `ε = β_promo / (−ln(1−d̄)) ≈ 0` para qualquer `d̄`. Com `ε = 0`, o desconto não aumenta a procura e o ótimo do MILP é **nunca descontar**: resultado trivial. |
| 3.5 `GroupKFold` por *stem* do nome | Os grupos não são fiáveis: os nomes estão desalinhados com o URL/SKU (3.1). |
| 3.1 `Y3` risco de encalhe | O proxy assenta em `Y1`, que é ruído. |
| E1, E2 | Resultado esperado: "nenhum modelo bate a média; efeito de promoção nulo". É um resultado legítimo, mas não alimenta a otimização. |

### 5.2 O que continua a valer [F]

- A **formulação PLNTDIA** (secção 2), o **MILP**, a **DP exata**, o **GA/SA/Tabu**, o **TOPSIS/AHP** e o **simulador** não dependem do sinal do `Sales Volume`; dependem de `p_i`, das categorias e dos parâmetros de procura.
- **Dados reais aproveitáveis:** 228 produtos únicos (por URL), com **preço real**, **tipo de peça** (`terms`, depois de corrigido), **secção** e **texto descritivo**. Chega para gerar instâncias realistas do problema: distribuição de preços, mistura de categorias, margens por escalão `𝒦_i` (C2).

### 5.3 Proposta reorientada [S] — "ML pequeno, decisão no centro"

Vai ao encontro da indicação do professor: o treino de modelos é uma parte pequena, e o peso do trabalho está no planeamento/decisão, com a Aprendizagem Automática ao serviço dele (pergunta integradora: *"como usar a ML para o planeamento e o apoio à decisão?"*).

```
 Zara (catálogo real limpo)         Literatura / Par C (ASOS/Myntra, % desconto)
   p_i, terms, section, texto              ε por categoria, decaimento δ, lift por escalão
            │                                         │
            └──────────────┬──────────────────────────┘
                           ▼
            GERADOR DE INSTÂNCIAS + SIMULADOR (A1–A10, cenários)
                           ▼
         CAMADA PRESCRITIVA (PLNTDIA) — núcleo do trabalho
      MILP · DP exata · GA/memético · SA/Tabu · TOPSIS/AHP
                           ▼
            milhares de (estado → escalão ótimo)
                           ▼
         CAMADA DE APRENDIZAGEM (AAUTIA) — treino pequeno
   árvore de decisão / GBM pequeno que aprende a POLÍTICA ótima
   → regras interpretáveis de markdown (DSS) + avaliação no simulador
```

1. **Zara → gerador de instâncias.** Limpar (secção 6) e usar `price`, `terms`, `section` para criar instâncias de 20–228 produtos. A procura base `λ_i` deixa de vir de um modelo treinado e passa a ser parâmetro de cenário. **Opcional:** usar `Sales Volume` *apenas* como fonte de heterogeneidade de escala entre produtos (`λ_i ∝ V_i`), declarando-o explicitamente como variável sem sinal.
2. **Resposta ao desconto (`ε`, lift por escalão, `δ`)** vinda de:
   - **(a) literatura de markdown em moda** (Tarefa 3: intervalos de elasticidade por categoria), e/ou
   - **(b) dataset do Par C** (ASOS/Myntra tem `discount_percentage` explícito → curva de lift por profundidade de desconto). Liga os pares e responde à questão 8 da arquitetura. **[A DECIDIR com a equipa]**
   Tudo tratado como **cenários + análise de sensibilidade** (E4), como já previsto.
3. **AAUTIA "pequeno" com papel claro:**
   - **(i) Resultado negativo documentado** (secções 4.1–4.3): é EDA/validação honesta e ocupa pouco espaço.
   - **(ii) Aprender a política** (*policy distillation*): resolver o MILP/DP em muitas instâncias e cenários, gerar pares `(idade t, sell-through, stock relativo, terms, preço, escalão atual) → escalão ótimo`, e treinar uma **árvore de decisão** (e, no máximo, um GBM pequeno para comparação). Avaliar **no simulador** a perda de `J` face ao ótimo e o ganho face ao baseline B2. O treino é pequeno, os dados são gerados por nós sem fuga para os dados de teste (instâncias de teste separadas), e o resultado é um **DSS interpretável**. Já estava previsto como extensão na secção 4.4 da arquitetura; passa a ser a componente principal de ML.
   - **(iii) [opcional]** modelo substituto (*surrogate*) do simulador para acelerar o GA.
4. **O que se mantém sem alterações:** formulação (secção 2), MILP/DP/GA/SA/TOPSIS (secção 4), protocolo anti-*optimizer's curse* (5.2), métricas (7).

**Porque não trocar de dataset?** O Zara já está atribuído ao Par B no `DATA.md`; com a reorientação acima é utilizável. Mas a pergunta deve ser feita aos docentes (secção 7), porque o `CONTEXT.md` pede dados com tempo, desconto e inventário, e este dataset não tem nenhum dos três.

---

## 6. Plano de limpeza (`src/data_loader/zara_loader.py`)

1. Ler com `sep=';'`, `encoding='utf-8'` e verificar os carateres acentuados (há pelo menos um nome com `BOUCL�`).
2. Remover as colunas constantes: `Product Category`, `brand`, `currency`.
3. Remover a linha sem `name`.
4. **Chave de produto = `url`** (ou `sku`); deduplicar: 252 → 228. Nos 24 pares, manter o registo cujo `name` coincide com o *slug* do URL.
5. Corrigir `terms` a partir do *slug* do URL/nome quando forem incoerentes (ex.: JACKET/OVERSHIRT/BLAZER → `jackets`); manter `terms` original numa coluna à parte.
6. Marcar as colunas sintéticas (`Promotion`, `Seasonal`, `Product Position`, `Sales Volume`) num dicionário de metadados, para nunca entrarem como *features* causais sem aviso.
7. Derivar: `log_price`, preço relativo à mediana de `terms`, conjunto admissível `𝒦_i` dado `ρ` e `m_min`.

---

## 7. Alterações a fazer e decisões

**Documentos a atualizar (depois de discutir com o João Vitor):**
- `DATA.md`, Dataset 2: esquema real (secção 2 deste relatório), N = 252/228, data da recolha, limitações.
- `arquitetura_par_b.md`: secção 0 (questões 1–5 respondidas aqui), secção 3 (ML passa a *policy learning*), secção 5.3 (A2/A4 deixam de ser calibrados no Zara), E1/E2.

**Perguntas para os docentes [A DECIDIR]:**
1. Aceitam o Zara como **catálogo real para gerar instâncias**, com a procura calibrada pela literatura/Par C, dado que as variáveis de vendas não têm sinal?
2. A componente de AAUTIA pode ser **aprender a política ótima** (árvore de decisão sobre soluções do MILP) em vez de prever vendas? Encaixa no "treino pequeno" que indicaram?
3. Preferem que o Par B troque de dataset? Se sim, qual alternativa da lista do `DATA.md`?

**Com a equipa:** podemos usar as curvas de desconto do Par C (ASOS/Myntra) para calibrar `ε` por categoria?
