# Gestão e Seleção de Datasets: Otimização de Descontos no Retalho Têxtil

## 1. Enquadramento e Critérios de Seleção

O presente documento formaliza os critérios metodológicos de avaliação e seleção dos conjuntos de dados (*datasets*) a adotar pela **Equipa 3** no âmbito do **Challenge 2** do MEIA (ISEP), articulando as unidades curriculares de **AAUTIA** (Aprendizagem Automática) e **PLNTDIA** (Planeamento e Tomada de Decisão Inteligente).

### 1.1 Alinhamento com o Feedback da Equipa Docente

Na sequência da orientação com o corpo docente, foi determinado que o escopo de dados deve centrar-se **estrita e exclusivamente no ecossistema de retalho de moda e vestuário (*fashion retail & fast fashion*)**, excluindo categorias genéricas de supermercados ou grandes superfícies (como Walmart ou retalho geral indiferenciado).

Esta diretriz permite:
- Modelar o comportamento de coleções e peças sujeitas a ciclos de vida curtos e forte sazonalidade;
- Refletir com autenticidade a dinâmica de atributos de produto (cor, tamanho, tipo de corte, tecido, estação);
- Estudar a pressão real de escoamento de inventário para libertação de capital de giro em loja física e canais digitais.

### 1.2 Objetivo Analítico e Requisitos do Negócio

A solução visa definir a política ótima de descontos e remarcações (*markdown optimization*) para peças de vestuário em função do tempo de permanência em loja/catálogo, prevenindo:
- **Custo de Inventário Parado e Retenção de Capital:** Imobilização prolongada de recursos financeiros e espaço de armazenamento em produtos com saída estagnada;
- **Erosão da Margem e Desvalorização Excessiva:** Descontos extemporâneos ou excessivos que eliminem a margem de contribuição líquida da peça ou gerem prejuízo operacional.

### 1.3 Critérios de Elegibilidade dos Datasets

1. **Domínio Exclusivo de Moda:** Marcas de vestuário ou plataformas especializadas em moda (*apparel, footwear, accessories*).
2. **Preço de Referência e Desconto:** Existência de preço base/recomendado (*original price* / *MRP*), preço de venda efetivo (*selling price*) e/ou percentagem de desconto aplicada.
3. **Temporalidade e Ciclo de Vida:** Possibilidade de inferir ou ler diretamente o tempo de exposição (*days on shelf* / *age in store*), a coleção sazonal e o ritmo de escoamento.
4. **Viabilidade Científica (Google Scholar):** Disponibilidade prévia de literatura académica e estudos indexados onde o dataset (ou a metodologia sobre a marca) seja analisado, viabilizando a recolha de pelo menos 3 artigos por par de estudantes.

---

## 2. Catálogo Especializado: 5 Datasets de Moda e Vestuário

Apresenta-se a análise técnica de 5 datasets de referência dedicados ao retalho de moda, integrando grandes marcas multinacionais e plataformas digitais de vestuário.

---

### Dataset Candidato 1: H&M Personalized Fashion Recommendations
- **Origem / Repositório:** Kaggle (H&M Group)
- **Acesso:** https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations/data
- **Dimensão e Cobertura:** Mais de 31 milhões de transações, 105.000 artigos de moda e 1,3 milhões de clientes (2018–2020).
- **Campos Principais:**
  - `transactions_train.csv`: `t_dat` (data da transação), `customer_id`, `article_id`, `price` (preço transacionado normalizado), `sales_channel_id`.
  - `articles.csv`: `article_id`, `product_code`, `prod_name`, `product_type_name`, `product_group_name`, `graphical_appearance_name`, `colour_group_name`, `perceived_colour_value_name`, `department_name`, `index_name`, `section_name`, `garment_group_name`, `detail_desc`.
- **Aplicabilidade ao Problema:**
  - Permite acompanhar a curva temporal de vendas de cada SKU desde a introdução em catálogo até ao escoamento final.
  - A variação do campo `price` ao longo das semanas reflete a aplicação de remarcações graduais e permite medir a elasticidade da procura por departamento de vestuário.
- **Pontos Fortes:** Dados reais de transações de um dos maiores grupos mundiais de retalho de moda; metadados descritivos detalhados.
- **Limitações:** Requer agregação temporal dos registos para derivar o inventário inicial estimado e a cadência semanal de vendas.
- **Termos de Pesquisa para o Google Scholar:**
  - `"H&M dataset" AND ("markdown" OR "dynamic pricing" OR "demand forecasting")`
  - `"H&M personalized fashion" AND "price elasticity"`
  - `"fast fashion retail" "H&M" "sales velocity" transaction data`

---

### Dataset Candidato 2: Zara Sales & Brand Performance Dataset
- **Origem / Repositório:** Kaggle ("ZARA Sales: Predicting Sales Volume & Brand Performance" / "Zara Sales for EDA")
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `zara sales eda product position promotion`)
- **Dimensão e Cobertura:** Catálogo de artigos de moda da Zara com métricas de desempenho de vendas, atributos de produto e políticas promocionais.
- **Campos Principais:**
  - `Product ID`, `Product Name`, `Description`, `Product Category` / `Section` (Mulher, Homem, Criança, etc.).
  - `Price`, `Currency`.
  - `Sales Volume` (volume de vendas registado).
  - `Promotion` (variável indicadora de aplicação de desconto/promoção).
  - `Seasonal` (indicador de coleção sazonal / rotatividade temporal).
  - `Product Position` (posicionamento da peça em loja física e montra digital).
- **Aplicabilidade ao Problema:**
  - Exemplifica o modelo clássico de *fast fashion* de rotação rápida de montra da Inditex.
  - Permite correlacionar diretamente o efeito de ativar promoções com o aumento no volume de escoamento e a posição da peça.
- **Pontos Fortes:** Foco direto no modelo da Zara; estrutura tabular limpa e imediata para classificação e regressão rápida em AAUTIA.
- **Limitações:** Volume agregado de transações em vez de séries temporais minuto-a-minuto.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Zara" AND ("fast fashion" OR "markdown optimization") AND "sales volume"`
  - `"dynamic pricing" "apparel" "Zara" promotion sales`
  - `"fast fashion inventory management" "markdown" sales elasticity`

---

### Dataset Candidato 3: ASOS E-Commerce & Dynamic Pricing Dataset
- **Origem / Repositório:** Kaggle / ASOS Web Scraping Archive
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `asos ecommerce products price discount`)
- **Dimensão e Cobertura:** Mais de 30.000 artigos de moda catalogados com preços históricos, preços recomendados de tabela e percentagens de desconto aplicadas.
- **Campos Principais:**
  - `product_id`, `brand_name`, `product_type`, `category_name`.
  - `current_price` (preço efetivo de venda).
  - `previous_price` / `rrp` (preço recomendado original sem desconto).
  - `is_marked_down` (indicador booleano de artigo em saldo/promoção).
  - `discount_percentage` (profundidade percentual do desconto concedido).
  - `colour`, `sizes_available` (grade de tamanhos em stock remanescente).
- **Aplicabilidade ao Problema:**
  - Modela diretamente o desfasamento entre o preço base e a remarcação necessária para escoar o artigo.
  - A variável `sizes_available` permite analisar a questão das quebras de sortido (*broken size curves*), fator crítico que acelera o encalhe de peças de roupa e obriga a liquidação agressiva.
- **Pontos Fortes:** Fornece explicitamente o preço original e a percentagem de desconto; fundamentação académica extensa por via dos artigos publicados pela própria equipa de investigação da ASOS.
- **Limitações:** Necessidade de harmonizar categorias de vestuário e filtrar artigos de acessórios não têxteis se pretendido.
- **Termos de Pesquisa para o Google Scholar:**
  - `"ASOS" AND ("dynamic pricing" OR "markdown optimization") AND "fashion retail"`
  - `"causal machine learning" "markdown" "pricing" ASOS`
  - `"fashion retail" "broken sizes" "clearance markdown" pricing`

---

### Dataset Candidato 4: Myntra Fashion Products & Dynamic Discounts Dataset
- **Origem / Repositório:** Kaggle
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `myntra fashion dataset mrp discount price`)
- **Dimensão e Cobertura:** Mais de 40.000 a 100.000 produtos de vestuário abrangendo marcas globais de renome (Levi's, Nike, Puma, Mango, H&M, etc.).
- **Campos Principais:**
  - `BrandName`, `ProductTitle`, `Category`, `SubCategory`.
  - `MRP` (*Maximum Retail Price* — preço original de etiqueta sem desconto).
  - `Price` / `DiscountedPrice` (preço de venda com desconto praticado).
  - `DiscountPercent` (percentagem explícita de desconto concedido).
  - `Season` (estação da coleção: Summer, Fall, Winter, Spring).
  - `Gender`, `Rating`, `RatingCount`.
- **Aplicabilidade ao Problema:**
  - O campo `Season` permite identificar formalmente o ciclo da coleção e a urgência de escoamento antes da transição sazonal.
  - A diferença entre `MRP` e `DiscountedPrice` espelha com exatidão a estratégia de remuneração e corte de preço por marca e por categoria de produto.
- **Pontos Fortes:** Variável explícita de estação/coleção (`Season`); riqueza de marcas conceituadas; cálculo direto de margem e desconto.
- **Limitações:** Focado em catálogo online; não apresenta séries de transações individuais por segundo.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Myntra" AND "fashion" AND ("discount" OR "markdown" OR "price prediction")`
  - `"apparel" "MRP" "selling price" discount optimization`
  - `"fashion seasonal clearance" "price elasticity" "e-commerce"`

---

### Dataset Candidato 5: Shein Fast-Fashion Pricing & Clearance Dataset
- **Origem / Repositório:** Kaggle
- **Acesso:** https://www.kaggle.com/datasets (pesquisa: `shein fashion pricing discount inventory`)
- **Dimensão e Cobertura:** Milhares de registos de vestuário da plataforma líder em *ultra-fast fashion*.
- **Campos Principais:**
  - `sku`, `title`, `category_name`, `subcategory`.
  - `retail_price` (preço de referência inicial) e `sale_price` (preço promocional ativo).
  - `discount_percentage` (percentagem de redução).
  - `stock_status` / `inventory_remaining` (indicador de disponibilidade em armazém).
  - `rating_count`, `reviews` (tração de mercado).
- **Aplicabilidade ao Problema:**
  - Fornece o extremo da rapidez de rotação: produtos concebidos para ciclos de permanência de poucas semanas, onde a ausência de desconto rápido gera acumulação imediata de stock morto.
  - Permite avaliar estratégias agressivas de *flash clearance* e escoamento rápido de lotes.
- **Pontos Fortes:** Representa a fronteira tecnológica atual do retalho de vestuário hiper-rápido; contraste interessante com o retalho tradicional.
- **Limitações:** Maior volatilidade e catálogo com menor ciclo de vida por peça.
- **Termos de Pesquisa para o Google Scholar:**
  - `"Shein" AND ("fast fashion" OR "pricing strategy") AND ("discount" OR "inventory")`
  - `"ultra-fast fashion" supply chain markdown optimization`
  - `"clearance pricing" "apparel" short lifecycle demand forecasting`

---

## 3. Matriz Comparativa dos Datasets Especializados em Moda

| Critério de Avaliação | Dataset 1: H&M | Dataset 2: Zara | Dataset 3: ASOS | Dataset 4: Myntra | Dataset 5: Shein |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Segmento de Retalho** | Fast Fashion Global | Fast Fashion de Loja/Online | Moda Digital Multimarca | Marketplace de Moda | Ultra-Fast Fashion |
| **Granularidade** | Transação (SKU e cliente) | Produto / Posição de Loja | Produto / Tamanho | Produto / Coleção | SKU / Lote de Produção |
| **Preço Base vs. Desconto** | Preço transacionado | Flag de promoção ativa | `rrp` vs. `current_price` | `MRP` vs. `DiscountedPrice`| `retail_price` vs. `sale_price` |
| **Métrica de Desconto** | Derivável por histórico | Binária (Sim/Não) | `discount_percentage` (%) | `DiscountPercent` (%) | `discount_percentage` (%) |
| **Dimensão Temporal / Ciclo**| Série temporal (`t_dat`) | Sazonalidade (`Seasonal`) | Quebra de tamanhos | Estação (`Season`) | Rotação hiper-rápida |
| **Adequação AAUTIA (ML)** | Máxima (procura/elasticidade)| Elevada (classif./regressão) | Elevada (regressão causal) | Elevada (elasticidade/preço) | Elevada (previsão de vendas) |
| **Adequação PLNTDIA (Decisão)**| Elevada (trajetórias preço) | Elevada (política catálogo) | Muito Elevada (regras saldo)| Muito Elevada (remarcação) | Muito Elevada (escoamento) |
| **Cobertura Google Scholar** | Muito Elevada | Elevada | Elevada | Elevada | Média / Em expansão |

---

## 4. Recomendações Estratégicas para os 3 Datasets Finais

Com a exclusão de conjuntos generalistas (como Walmart ou Online Retail II) e a convergência para um ecossistema 100% focado no setor têxtil, sugerem-se três configurações para a seleção final da equipa:

### Opção Recomendada (Equilíbrio Multicanal e Sazonal)
1. **H&M:** Base transacional detalhada com mais de 31 milhões de registos, garantindo uma modelação robusta de séries temporais e atributos de moda para AAUTIA.
2. **Zara:** Caso de estudo icónico de *fast fashion* europeu com dados de posicionamento e rotatividade sazonal.
3. **ASOS** ou **Myntra:** Fornece a comparação formal entre preço de tabela original (`MRP`/`RRP`) e o preço com desconto, com especificação explícita de coleção sazonal (`Season`) e quebra de tamanhos.

---

## 5. Estrutura Operacional dos 3 Pares de Trabalho

Com base na escolha dos 3 datasets da lista especializada, os pares de trabalho da equipa organizar-se-ão de acordo com o seguinte modelo formal de investigação:

### Par A
- **Membros da Equipa:** Santiago da Mata Bossa (`1211173`) e [Elemento 2 a designar]
- **Dataset Atribuído:** H&M Personalized Fashion Recommendations
- **Objetivo Técnico:** Pré-processamento dos registos transacionais (`transactions_train.csv`) e metadados de produto (`articles.csv`), engenharia de atributos de vestuário e modelação da elasticidade-preço da procura ao longo do ciclo de vida das peças em loja.
- **Artigos Científicos Indexados (Google Scholar - Mínimo 3 a pesquisar pelo par):**
  1. *Artigo 1:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  2. *Artigo 2:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  3. *Artigo 3:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]

### Par B
- **Membros da Equipa:** João Vitor Carneiro Moreira (`1260436`) e Rômulo Semião Arantes (`1260446`)
- **Dataset Atribuído:** Zara Sales & Brand Performance Dataset
- **Objetivo Técnico:** Modelação preditiva do impacto de promoções no volume de vendas, análise da rotação sazonal de catálogo (*fast fashion*) e otimização da política de descontos para liquidação atempada de stock.
- **Artigos Científicos Indexados (Google Scholar - Mínimo 3 a pesquisar pelo par):**
  1. *Artigo 1:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  2. *Artigo 2:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  3. *Artigo 3:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]

### Par C
- **Membros da Equipa:** [Elemento 5] e [Elemento 6]
- **Dataset Atribuído:** ASOS Dynamic Pricing ou Myntra Fashion Products
- **Objetivo Técnico:** Modelação de curvas de desconto face ao preço base (`MRP`/`RRP`) e escoamento por transição de estação.
- **Artigos Científicos Indexados (Google Scholar - Mínimo 3 a pesquisar pelo par):**
  1. *Artigo 1:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  2. *Artigo 2:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
  3. *Artigo 3:* [Título, Autores, Ano, Publicação, Contributo metodológico e relevância — a preencher pelo par]
