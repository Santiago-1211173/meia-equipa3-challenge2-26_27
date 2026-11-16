# Gestão e Seleção de Datasets: Otimização de Descontos no Retalho Têxtil

## 1. Enquadramento e Critérios de Seleção

O presente documento formaliza os critérios metodológicos de avaliação e seleção dos conjuntos de dados (*datasets*) a adotar pela **Equipa 3** no âmbito do **Challenge 2** do MEIA (ISEP), articulando as unidades curriculares de **AAUTIA** (Aprendizagem Automática) e **PLNTDIA** (Planeamento e Tomada de Decisão Inteligente).

### 1.1 Objetivo Analítico e Requisitos do Negócio

A solução a desenvolver visa determinar a política ideal de descontos e remarcações (*markdown optimization*) para peças de vestuário, considerando o tempo de permanência em loja, a rotação de coleções sazonais e o escoamento de inventário, evitando simultaneamente:
- O custo de retenção de capital em inventário estagnado (*holding cost* e depreciação);
- A desvalorização excessiva ou extemporânea da peça, preservando uma margem líquida positiva (*margin preservation*).

### 1.2 Critérios de Elegibilidade dos Datasets

Para viabilizar a modelação preditiva e a otimização de decisões, os datasets candidatos devem cumprir os seguintes requisitos:

1. **Domínio e Atributos de Produto:** Conter dados do setor de retalho (preferencialmente têxtil/vestuário), com categorização dos artigos (ex.: categoria, departamento, género, estilo, cor, tamanho).
2. **Dimensão de Preço e Desconto:** Conter valores de preço base (*original price* / *MRP*), preço de venda efetivo (*selling price*) e/ou percentagem de desconto aplicada.
3. **Dimensão Temporal e Inventário:** Permitir o cálculo ou extração do tempo de permanência em loja (*days on shelf* / *age in store*), data da transação, sazonalidade e unidades vendidas face ao stock remanescente.
4. **Viabilidade Científica (Google Scholar):** Existência prévia de literatura académica e artigos indexados que utilizem o dataset ou abordem o mesmo problema no mesmo contexto de dados, permitindo a cada par de trabalho recolher no mínimo 3 artigos para o Estado da Arte de PLNTDIA.

---

## 2. Catálogo de Candidatos: 5 Datasets em Avaliação

Apresenta-se de seguida a análise detalhada de 5 datasets de referência disponíveis publicamente, selecionados pela sua relevância para o problema de remarcações dinâmicas e rotatividade de stocks.

---

### Dataset Candidato 1: H&M Personalized Fashion Recommendations
- **Origem / Repositório:** Kaggle (H&M Group)
- **Acesso:** https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations/data
- **Dimensão e Cobertura:** Mais de 31 milhões de transações de compra, 105.000 artigos de moda e 1,3 milhões de clientes entre 2018 e 2020.
- **Campos Principais:**
  - `transactions_train.csv`: `t_dat` (data da compra), `customer_id`, `article_id`, `price` (preço transacionado normalizado), `sales_channel_id`.
  - `articles.csv`: `article_id`, `product_code`, `prod_name`, `product_type_name`, `product_group_name`, `graphical_appearance_name`, `colour_group_name`, `perceived_colour_value_name`, `department_name`, `index_name`, `section_name`, `garment_group_name`, `detail_desc`.
- **Aplicabilidade ao Problema:**
  - Permite rastrear o ciclo de vida temporal de cada artigo desde a sua introdução no catálogo até à última venda.
  - O campo `price` reflete variações ao longo do tempo para o mesmo `article_id`, permitindo inferir quando ocorreram reduções de preço e a correspondente aceleração ou desaceleração do volume de vendas.
  - Riqueza ímpar de metadados visuais e categoriais de moda.
- **Pontos Fortes:** Dados reais de um dos maiores retalhistas globais de *fast fashion*; volume robusto de literatura no Google Scholar.
- **Limitações:** Requer agregação temporal e derivação explícita do nível inicial de inventário (estimável através do somatório de vendas do ciclo de vida da peça).
- **Termos de Pesquisa para o Google Scholar:**
  - `"H&M dataset" AND ("markdown" OR "dynamic pricing" OR "demand forecasting")`
  - `"H&M personalized fashion" AND "inventory" OR "price elasticity"`
  - `"fast fashion retail" "H&M" "sales velocity" transaction data`

---

### Dataset Candidato 2: Zara Sales & Brand Performance Dataset
- **Origem / Repositório:** Kaggle ("ZARA Sales: Predicting Sales Volume & Brand Performance" / "Zara Sales for EDA")
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `zara sales eda product position promotion`)
- **Dimensão e Cobertura:** Catálogo estruturado de produtos de moda Zara com registo de preços, promoções e volume de vendas.
- **Campos Principais:**
  - `Product ID`, `Product Name`, `Description`, `Product Category` / `Section` (Mulher, Homem, Criança, etc.).
  - `Price`, `Currency`.
  - `Sales Volume` (quantidade agregada transacionada).
  - `Promotion` (variável indicadora de aplicação de desconto/promoção).
  - `Seasonal` (indicador de coleção sazonal / rotatividade temporal).
  - `Product Position` (posicionamento da peça em loja/catálogo).
- **Aplicabilidade ao Problema:**
  - Fornece diretamente os atributos essenciais para classificação e regressão rápida em AAUTIA: relação direta entre preço, promoção ativa e volume de vendas.
  - Modela diretamente a transição entre artigos de preço regular e artigos sob promoção sazonal.
- **Pontos Fortes:** Foco direto no modelo de negócio *fast fashion* da Zara (referência em rápida rotação de stocks); estrutura tabular limpa e imediata de manipular.
- **Limitações:** Dados agregados em termos temporais comparativamente a registos transacionais granulares por segundo/minuto.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Zara" AND ("fast fashion" OR "markdown optimization") AND "sales volume"`
  - `"dynamic pricing" "apparel" "Zara" promotion sales`
  - `"fast fashion inventory management" "markdown" sales elasticity`

---

### Dataset Candidato 3: Retail Markdown Optimization: Discounts & Sales
- **Origem / Repositório:** Kaggle ("Retail Markdown Optimization: Discounts & Sales")
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `retail markdown optimization discounts sales`)
- **Dimensão e Cobertura:** Dados estruturados para problemas de otimização de remarcações, impacto de descontos e cadência de escoamento.
- **Campos Principais:**
  - `Item_ID`, `Category`, `Sub_Category`.
  - `Original_Price`, `Discount_Rate` / `Markdown_Depth`, `Current_Price`.
  - `Units_Sold` / `Sales_Velocity`, `Inventory_Remaining` / `Stock_Level`.
  - `Days_In_Store` / `Time_On_Shelf`, `Collection_Season`.
  - `Gross_Margin`, `Holding_Cost_Rate`.
- **Aplicabilidade ao Problema:**
  - Corresponde com rigor formal à função objetivo de PLNTDIA: possui o tempo de prateleira (`Days_In_Store`), o stock remanescente, o desconto atribuído e a velocidade de venda.
  - Permite formular diretamente modelos de programação matemática (Programação Linear Inteira, Algoritmos Genéticos, MDP) e calcular o risco de rutura vs. encalhe.
- **Pontos Fortes:** Métricas nativas de retenção de capital e margem bruta; desenho específico para o dilema de *markdown*.
- **Limitações:** Conjunto de dados focado em contexto promocional, podendo exigir enriquecimento de atributos descritivos do produto.
- **Termos de Pesquisa para o Google Scholar:**
  - `"retail markdown optimization" "days on shelf" inventory clearance`
  - `"dynamic markdown" "apparel" "holding cost" mathematical optimization`
  - `"clearance pricing" "fast fashion" OR "retail" "genetic algorithm" OR "linear programming"`

---

### Dataset Candidato 4: Walmart Retail Data Analytics (Store Sales & Markdowns)
- **Origem / Repositório:** Kaggle (Walmart Recruiting - Store Sales Forecasting / Retail Data Analytics)
- **Acesso:** https://www.kaggle.com/competitions/walmart-recruiting-store-sales-forecasting/data
- **Dimensão e Cobertura:** 421.570 observações semanais cobrindo 45 lojas e 99 departamentos (incluindo vestuário e têxtil-lar) durante 3 anos consecutivos.
- **Campos Principais:**
  - `Store`, `Dept` (departamento, permitindo isolar secções têxteis).
  - `Date`, `Weekly_Sales`, `IsHoliday`.
  - `MarkDown1`, `MarkDown2`, `MarkDown3`, `MarkDown4`, `MarkDown5` (valores anonimizados de descontos e remarcações promocionais).
  - `Temperature`, `Fuel_Price`, `CPI`, `Unemployment`, `Size`, `Type`.
- **Aplicabilidade ao Problema:**
  - Benchmark clássico na literatura académica de séries temporais de retalho com intervenção promocional explícita.
  - Permite avaliar o efeito cruzado de diferentes tipos de desconto (MarkDown1 a 5) sobre a aceleração de vendas semanais.
- **Pontos Fortes:** Validação académica exaustiva (mais de 200 publicações no Google Scholar); dados temporais contínuos com registo de feriados e sazonalidade externa.
- **Limitações:** Dados agregados à escala semanal por departamento, sem granularidade ao nível de SKU unitário.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Walmart" "MarkDown1" "markdown" sales forecasting`
  - `"retail markdown" "store sales forecasting" machine learning`
  - `"promotional markdowns" "inventory" demand response Walmart`

---

### Dataset Candidato 5: Online Retail II (UCI Machine Learning Repository)
- **Origem / Repositório:** UCI Machine Learning Repository / Kaggle
- **Acesso:** https://archive.ics.uci.edu/dataset/502/online+retail+ii
- **Dimensão e Cobertura:** 1.067.371 transações reais registadas entre 01/12/2009 e 09/12/2011 por um retalhista sediado no Reino Unido.
- **Campos Principais:**
  - `Invoice` (número da fatura, prefixado com 'C' para devoluções/cancelamentos).
  - `StockCode` (código identificador do produto).
  - `Description` (descrição textual do produto).
  - `Quantity` (número de unidades transacionadas).
  - `InvoiceDate` (data e hora exata da transação).
  - `Price` / `UnitPrice` (preço unitário do produto em libras).
  - `Customer ID` (identificador único do cliente).
  - `Country` (país de destino).
- **Aplicabilidade ao Problema:**
  - Permite derivar com precisão cirúrgica a primeira e a última aparição de cada código de produto (`StockCode`), estabelecendo o tempo de vida total em catálogo.
  - Possibilita observar reduções graduais do `UnitPrice` ao longo das semanas de vida do produto e quantificar a resposta na `Quantity` vendida.
- **Pontos Fortes:** Dataset real e verificado, amplamente aceite pela comunidade científica; permite analisar devoluções e cancelamentos reais.
- **Limitações:** Catálogo misto de retalho (presentes, decoração e acessórios têxteis), exigindo filtragem de categorias se o foco for estritamente vestuário.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Online Retail II" "dynamic pricing" OR "markdown"`
  - `"UCI" "Online Retail" "price elasticity" demand forecasting`
  - `"Online Retail" transaction data "inventory turnover" "clearance"`

---

## 3. Matriz Comparativa dos Datasets Candidatos

| Critério de Avaliação | Dataset 1: H&M Transactions | Dataset 2: Zara Sales | Dataset 3: Retail Markdown Opt. | Dataset 4: Walmart Retail | Dataset 5: Online Retail II |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Domínio Central** | Moda / Fast Fashion | Moda / Fast Fashion | Retalho / Descontos | Grandes Superfícies | Retalho Geral / E-Commerce |
| **Granularidade** | Transação (SKU e cliente) | Produto / Posição | SKU / Artigo | Semanal por Departamento | Transação (Linha de fatura) |
| **Dados de Desconto** | Variação de preço no tempo | Flag de promoção | Desconto explícito (%) | 5 tipos de Markdown | Variação de preço unitário |
| **Tempo em Loja** | Derivável por histórico | Variável sazonal | Explícito (`Days_In_Store`) | Eixo temporal contínuo | Derivável (`First/Last Date`) |
| **Adequação AAUTIA (ML)** | Elevada (classif. e regressão) | Elevada (regressão direta) | Muito Elevada (predição contínua) | Elevada (séries temporais) | Elevada (predição de procura) |
| **Adequação PLNTDIA (Decisão)**| Média/Elevada (requer agregação)| Média (otimização de catálogo) | Máxima (otimização analítica) | Elevada (alocação de markdown) | Elevada (políticas de preço) |
| **Literatura no Scholar** | Muito Elevada | Elevada | Média/Elevada | Máxima | Máxima |

---

## 4. Recomendações para a Seleção dos 3 Datasets Finais

Para assegurar complementaridade metodológica, diversidade de abordagem e cumprimento dos requisitos de AAUTIA e PLNTDIA, recomenda-se a seguinte tríade de datasets:

1. **Dataset Principal de Retalho de Moda (H&M ou Zara):**
   - Fornece a identidade visual e categorial autêntica do setor têxtil.
   - Ideal para os módulos de classificação de categoria, identificação de artigos críticos e modelação de procura baseada em atributos estéticos/sazonais.
2. **Dataset de Otimização Direta de Remarcações (Retail Markdown Optimization):**
   - Fornece as variáveis nativas de controlo económico (`Days_In_Store`, `Stock_Level`, `Holding_Cost`, `Gross_Margin`).
   - Permite implementar e testar de forma imediata os algoritmos de planeamento e pesquisa operacional de PLNTDIA (Branch and Bound, Algoritmos Genéticos, Programação Linear).
3. **Dataset Transacional Granular ou Benchmark Académico (Online Retail II ou Walmart):**
   - Garante séries temporais ricas para cálculo de elasticidade-preço contínua e previsão de vendas em AAUTIA.
   - Proporciona cobertura bibliográfica exaustiva no Google Scholar, simplificando a fundamentação do artigo científico.

---

## 5. Estrutura Operacional dos 3 Pares de Trabalho

Assim que os 3 datasets forem formalmente ratificados pela equipa, a alocação de trabalho e o levantamento no Google Scholar serão registados de acordo com o seguinte modelo:

### Par A
- **Membros:** [Elemento 1] e [Elemento 2]
- **Dataset Atribuído:** [Designação do Dataset Selecionado 1]
- **Foco Técnico:** Limpeza, análise exploratória e modelação do comportamento da procura.
- **Artigos do Google Scholar (Mínimo 3):**
  1. *Artigo 1:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  2. *Artigo 2:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  3. *Artigo 3:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.

### Par B
- **Membros:** [Elemento 3] e [Elemento 4]
- **Dataset Atribuído:** [Designação do Dataset Selecionado 2]
- **Foco Técnico:** Limpeza, análise exploratória e modelação do comportamento da procura.
- **Artigos do Google Scholar (Mínimo 3):**
  1. *Artigo 1:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  2. *Artigo 2:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  3. *Artigo 3:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.

### Par C
- **Membros:** [Elemento 5] e [Elemento 6]
- **Dataset Atribuído:** [Designação do Dataset Selecionado 3]
- **Foco Técnico:** Limpeza, análise exploratória e modelação do comportamento da procura.
- **Artigos do Google Scholar (Mínimo 3):**
  1. *Artigo 1:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  2. *Artigo 2:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.
  3. *Artigo 3:* Título, Autores, Ano, Revista/Conferência, Resumo metodológico e relevância.

