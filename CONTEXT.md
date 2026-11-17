# Contexto do Projeto: Seleção Inteligente de Descontos no Retalho Têxtil

## 1. Identificação da Equipa e Projeto

- **Curso:** Mestrado em Engenharia de Inteligência Artificial (MEIA)
- **Instituição:** Instituto Superior de Engenharia do Porto (ISEP)
- **Ano Letivo:** 2026/2027
- **Desafio:** Challenge 2
- **Equipa:** Equipa 3
- **Tema:** Retalho (*Fashion & Apparel Retail*)

### Elementos da Equipa

| Estudante | Número de Aluno | E-mail Institucional | Grau / Instituição |
| :--- | :--- | :--- | :--- |
| Alexandre Dias Teixeira | 1230946 | 1230946@isep.ipp.pt | MEIA / ISEP |
| Álvaro Miguel Pinto da Silva Rodrigues Cardoso | 1940052 | 1940052@isep.ipp.pt | MEIA / ISEP |
| Fernando Manuel Oliveira Granjo | 1260430 | 1260430@isep.ipp.pt | MEIA / ISEP |
| João Vitor Carneiro Moreira | 1260436 | 1260436@isep.ipp.pt | MEIA / ISEP |
| Rômulo Semião Arantes | 1260446 | 1260446@isep.ipp.pt | MEIA / ISEP |
| Santiago da Mata Bossa | 1211173 | 1211173@isep.ipp.pt | MEIA / ISEP |

---

## 2. Enquadramento Curricular

O presente projeto é desenvolvido no âmbito da articulação pedagógica e científica de duas unidades curriculares do MEIA:

1. **AAUTIA (Aprendizagem Automática):** Focada na exploração de dados, engenharia de atributos, treino, afinação e validação de modelos estatísticos e de *Machine Learning* para aprendizagem a partir de padrões históricos.
2. **PLNTDIA (Planeamento e Tomada de Decisão Inteligente):** Centrada na modelação de espaços de estados, heurísticas, algoritmos de pesquisa e otimização combinatória (ex.: Programação Linear, Algoritmos Genéticos, Pesquisa Tabu), teoria dos jogos, sistemas de apoio à decisão (DSS) e análise multi-critério (AHP, TOPSIS).

### Desafio Integrador

> *"Como utilizar a aprendizagem automática para o planeamento e o apoio à decisão inteligente?"*

A solução desenvolvida combina uma componente preditiva (AAUTIA) para estimativa de comportamento da procura com uma componente prescritiva e decisora (PLNTDIA) para otimizar trajetórias de preços e remarcações. Adicionalmente, no âmbito de PLNTDIA, o trabalho compreende a elaboração e submissão de um artigo científico (*scientific paper*), o desenvolvimento da solução computacional, a manutenção de um blog de reporte semanal e a realização do *Demo Day*.

---

## 3. Problema de Negócio e Solução Proposta

### 3.1 Contexto e Dilema de Negócio no Retalho Têxtil

No setor de retalho têxtil e moda (*fast fashion* e vestuário sazonal), as coleções apresentam um ciclo de vida estritamente delimitado e elevada taxa de rotação. A permanência prolongada de peças em loja introduz dois riscos económicos antagónicos:

1. **Custo de Inventário Parado e Retenção de Capital:** A retenção excessiva de produtos nas prateleiras ou armazéns imobiliza capital de giro, consome espaço físico útil e agrava os custos de oportunidade perante novas coleções.
2. **Erosão da Margem e Desvalorização Prematura:** A aplicação de descontos imediatos, agressivos ou desregulados reduz drasticamente a margem de lucro por peça, podendo anular a rentabilidade ou originar prejuízo operacional direto.

### 3.2 Solução: Seleção Inteligente de Descontos e Remarcações (*Markdown Optimization*)

O objetivo do projeto é conceber e implementar um sistema inteligente de apoio à decisão para a recomendação e calendarização de descontos em peças de vestuário, tendo em conta:

- **Tempo de Permanência em Loja (*Age in Store* / *Days on Shelf*):** Avaliação contínua da fase do ciclo de vida da peça face à sazonalidade e rotação da coleção.
- **Velocidade de Escoamento (*Sell-Through Rate*):** Rácio entre o inventário inicial recebido e as unidades vendidas ao longo do tempo.
- **Elasticidade-Preço da Procura:** Sensibilidade estimada dos clientes a variações de preço para diferentes categorias, atributos de produto e faixas de valor.
- **Margem de Contribuição e Custo Base:** Preservação estrita de um patamar mínimo de rentabilidade que evite a venda com margem líquida negativa.

---

## 4. Arquitetura Conceptual da Solução

A solução técnica articula duas camadas complementares:

```text
[ Dados de Vendas, Stocks e Artigos ]
                 |
                 v
   +---------------------------+
   |   Camada Preditiva        |
   |   (AAUTIA - ML)           |
   | • Previsão de Procura     |
   | • Elasticidade do Preço   |
   | • Risco de Encalhe        |
   +---------------------------+
                 |
                 v
   +---------------------------+
   |   Camada Prescritiva      |
   |   (PLNTDIA - Decisão)     |
   | • Otimização de Markdown  |
   | • Pesquisa Operacional /  |
   |   Algoritmos Genéticos    |
   | • Multi-Critério (AHP)    |
   +---------------------------+
                 |
                 v
[ Política de Descontos e Calendarização ]
```

### 4.1 Camada Preditiva (AAUTIA)
- Previsão da procura em função de atributos da peça (categoria, cor, tamanho, preço base), do histórico de vendas e de variáveis temporais (semana do ano, estação, dias em exposição).
- Estimação da probabilidade de escoamento atempado e do impacto percentual de cada escalão de desconto sobre o volume de transações.

### 4.2 Camada Prescritiva e Decisora (PLNTDIA)
- Formulação do problema de remarcações como um problema de otimização restrita ou tomada de decisão multi-critério:
  - **Função Objetivo:** Maximizar a receita total acumulada e a libertação de capital até ao encerramento da coleção.
  - **Restrições:** Limites máximos de desconto aplicáveis, cadência mínima entre reduções consecutivas, salvaguarda do custo de aquisição/produção e limites de espaço de armazém.
  - **Métodos a Considerar:** Otimização combinatória (Branch and Bound, Programação Linear Inteira), meta-heurísticas (Algoritmos Genéticos, Pesquisa Tabu, Otimização por Enxame de Partículas), árvores de decisão e métodos multi-critério (TOPSIS/AHP).

---

## 5. Metodologia de Investigação e Organização do Trabalho

Para assegurar rigor científico e cumprimento integral dos requisitos pedagógicos, a equipa adotou uma metodologia estruturada em quatro fases operacionais:

### Fase 1: Seleção de Datasets Representativos
- Pesquisa de pelo menos 3 *datasets* públicos e fidedignos em repositórios de referência (Kaggle, Zenodo, IEEE DataPort, EU Open Data Portal).
- Requisito de dados: Devem conter dados transacionais de retalho têxtil, com registo de preços praticados, percentagens de desconto/promoção, volumes de vendas, datas/períodos de exposição e níveis de inventário.

### Fase 2: Distribuição da Equipa por Pares
A equipa de 6 elementos divide-se em 3 pares dedicados, atribuindo um *dataset* de moda a cada par:
- **Par A (2 elementos):** Análise exploratória e modelação do Dataset 1 (H&M Personalized Fashion Recommendations), integrando Santiago da Mata Bossa (`1211173`) e parceiro a designar.
- **Par B (2 elementos):** Análise exploratória e modelação do Dataset 2 (Zara Sales & Brand Performance).
- **Par C (2 elementos):** Análise exploratória e modelação do Dataset 3 (ASOS Dynamic Pricing ou Myntra Fashion Products).

### Fase 3: Levantamento Bibliográfico e Estado da Arte (Google Scholar)
- Cada par conduz uma pesquisa focada no Google Scholar com recurso a palavras-chave direcionadas (ex.: *"markdown optimization"*, *"dynamic pricing fashion retail"*, *"inventory clearance"*, *"demand forecasting price elasticity"* e a designação do respetivo dataset).
- Cada par seleciona e analisa criticamente pelo menos 3 artigos científicos indexados que utilizem o dataset correspondente ou que abordem diretamente a modelação de decisões de desconto no mesmo domínio.
- Total acumulado: No mínimo 9 artigos científicos analisados e fichados pela equipa.

### Fase 4: Redação do Estado da Arte e Seleção de Algoritmos
- Consolidação dos contributos num Estado da Arte aprofundado, integrando a revisão de literatura geral e específica conforme a matriz de avaliação de PLNTDIA.
- Decisão fundamentada sobre a família de algoritmos preditivos (AAUTIA) e algoritmos de otimização/decisão (PLNTDIA) a implementar na solução final.

---

## 6. Calendário e Prazos Principais

| Marco / Entregável | Unidade Curricular | Data Limite |
| :--- | :--- | :--- |
| Entrega da 1.ª Versão do Artigo Científico (Estado da Arte) | PLNTDIA | 18 de dezembro |
| Entrega da Versão Final do Artigo Científico | PLNTDIA | 10 de janeiro |
| Sessão de Demonstração Técnica (Demo Online, 30 min) | PLNTDIA / AAUTIA | 11 de janeiro |
| Pitch Presencial do Demo Day (5 min) | PLNTDIA / AAUTIA | 14 de janeiro |
| Submissão Final do Blog da Equipa | PLNTDIA | Dia seguinte ao Demo Day |
| Exame de Época Normal | PLNTDIA | 18 de janeiro |
| Exame de Recurso | PLNTDIA | 1 de fevereiro |

---

## 7. Estrutura Prevista do Repositório

```text
meia-equipa3-challenge2-26_27/
│
├── CONTEXT.md                    # Documento de enquadramento global do projeto
├── Material_aulas/               # Enunciados, guiões de laboratório e apresentações
│   ├── AAUTIA/                   # Guiões práticos de Aprendizagem Automática
│   └── PLNTDIA/                  # Slides, critérios de avaliação e programa
│
├── data/                         # Diretório para armazenamento dos 3 datasets
│   ├── dataset1/                 # Dataset 1 e metadados associados
│   ├── dataset2/                 # Dataset 2 e metadados associados
│   └── dataset3/                 # Dataset 3 e metadados associados
│
├── notebooks/                    # Notebooks de exploração e prototipagem
│   ├── exploratory_data_analysis/# Análise exploratória e limpeza de dados
│   ├── ml_models/                # Modelos preditivos de AAUTIA
│   └── decision_optimization/    # Algoritmos de decisão e otimização de PLNTDIA
│
├── src/                          # Código-fonte modular da solução final
│   ├── data_loader/              # Módulos de ingestão e pré-processamento
│   ├── predictors/               # Pipelines de inferência e modelação preditiva
│   └── optimizers/               # Algoritmos de planeamento e políticas de preço
│
├── paper/                        # Artigo científico em LaTeX / PDF
│   ├── state_of_the_art/         # Levantamento bibliográfico e fichamentos
│   └── draft/                    # Versões preliminares e versão final do paper
│
└── reports/                      # Relatórios complementares e materiais de suporte
```

