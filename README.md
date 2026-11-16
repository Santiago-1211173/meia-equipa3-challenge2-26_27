# Seleção Inteligente de Descontos no Retalho Têxtil
## *Markdown Optimization & Dynamic Pricing in Fashion Retail*

[![MEIA](https://img.shields.io/badge/MEIA-2026%2F2027-blue.svg)](https://www.isep.ipp.pt/)
[![ISEP](https://img.shields.io/badge/ISEP-Engenharia%20de%20IA-orange.svg)](https://www.isep.ipp.pt/)
[![Challenge 2](https://img.shields.io/badge/Challenge-2-green.svg)](#)
[![Equipa](https://img.shields.io/badge/Equipa-3-brightgreen.svg)](#)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)

---

## 1. Identificação do Projeto e Equipa

O presente repositório aloja os desenvolvimentos técnicos, experimentais e científicos do **Challenge 2** da **Equipa 3**, no âmbito do **Mestrado em Engenharia de Inteligência Artificial (MEIA)** do **Instituto Superior de Engenharia do Porto (ISEP)**, ano letivo de 2026/2027.

O projeto materializa a integração pedagógica e aplicada de duas unidades curriculares fundamentais:
- **AAUTIA (Aprendizagem Automática):** Modelação preditiva de procura, estimação da elasticidade-preço e inferência de probabilidade de escoamento.
- **PLNTDIA (Planeamento e Tomada de Decisão Inteligente):** Otimização prescritiva de remarcações (*markdowns*), formulação em espaços de estados e análise de decisão multi-critério (AHP/TOPSIS).

### Elementos da Equipa (Equipa 3)

| Estudante | Número de Aluno | E-mail Institucional | Grau / Instituição |
| :--- | :--- | :--- | :--- |
| **Alexandre Dias Teixeira** | 1230946 | `1230946@isep.ipp.pt` | MEIA / ISEP |
| **Álvaro Miguel Pinto da Silva Rodrigues Cardoso** | 1940052 | `1940052@isep.ipp.pt` | MEIA / ISEP |
| **Fernando Manuel Oliveira Granjo** | 1260430 | `1260430@isep.ipp.pt` | MEIA / ISEP |
| **João Vitor Carneiro Moreira** | 1260436 | `1260436@isep.ipp.pt` | MEIA / ISEP |
| **Rômulo Semião Arantes** | 1260446 | `1260446@isep.ipp.pt` | MEIA / ISEP |
| **Santiago da Mata Bossa** | 1211173 | `1211173@isep.ipp.pt` | MEIA / ISEP |

---

## 2. Problema de Negócio e Abordagem Proposta

No retalho de vestuário e moda (*fast fashion* e coleções sazonais), a rotação de artigos obedece a janelas temporais curtas e rigorosas. As equipas de gestão de inventário enfrentam um dilema económico clássico:
1. **Custo de Inventário Parado:** Reter peças em loja por tempo excessivo imobiliza capital, satura o espaço útil em loja/armazém e aumenta a depreciação face à chegada de novas coleções.
2. **Erosão Prematura da Margem:** Aplicar descontos excessivos ou demasiado cedo canibaliza as receitas e pode gerar vendas abaixo do custo de produção/aquisição.

### A Solução: Arquitetura em Duas Camadas

```text
[ Transações Históricas, Inventários e Metadados de Artigos ]
                              │
                              ▼
        ┌──────────────────────────────────────────┐
        │        CAMADA PREDITIVA (AAUTIA)         │
        │ • Previsão da procura por artigo         │
        │ • Modelação da elasticidade-preço        │
        │ • Risco de encalhe / sell-through rate   │
        └──────────────────────────────────────────┘
                              │
                              ▼
        ┌──────────────────────────────────────────┐
        │       CAMADA PRESCRITIVA (PLNTDIA)       │
        │ • Otimização de trajetórias de markdown  │
        │ • Pesquisa Operacional / Meta-heurísticas│
        │ • Tomada de Decisão Multi-Critério (AHP) │
        └──────────────────────────────────────────┘
                              │
                              ▼
    [ Política Recomendada de Descontos e Calendarização ]
```

Consulte os detalhes operacionais e científicos completos em [CONTEXT.md](CONTEXT.md).

---

## 3. Estrutura do Repositório

```text
meia-equipa3-challenge2-26_27/
│
├── CONTEXT.md                    # Enquadramento metodológico, curricular e científico
├── DATA.md                       # Catálogo de datasets candidatos, critérios e metadados
├── README.md                     # Apresentação geral do repositório e guia de arranque
├── requirements.txt              # Dependências Python do projeto
├── .env.example                  # Template de variáveis de ambiente
├── .gitignore                    # Regras de exclusão do Git (dados, caches, modelos)
├── .gitattributes               # Normalização de terminadores de linha
│
├── Material_aulas/               # Materiais de suporte pedagógico
│   ├── AAUTIA/                   # Guiões práticos de laboratório
│   └── PLNTDIA/                  # Critérios de avaliação, apontamentos e programa
│
├── data/                         # Diretório local para datasets (não versionado)
│   ├── README.md                 # Instruções de download e organização dos dados
│   ├── dataset1/                 # Dataset 1 (ex.: H&M Recommendations)
│   ├── dataset2/                 # Dataset 2 (ex.: Zara Sales Dataset)
│   └── dataset3/                 # Dataset 3 (ex.: Retail Markdown Dataset)
│
├── notebooks/                    # Jupyter Notebooks de experimentação
│   ├── exploratory_data_analysis/# Análise exploratória de dados (EDA) e limpeza
│   ├── ml_models/                # Treino e avaliação de modelos preditivos (AAUTIA)
│   └── decision_optimization/    # Modelação de decisão e otimizadores (PLNTDIA)
│
├── src/                          # Módulos Python reutilizáveis da solução final
│   ├── data_loader/              # Ingestão, validação e pré-processamento de dados
│   ├── predictors/               # Pipelines e wrappers de modelos preditivos
│   └── optimizers/               # Algoritmos de otimização combinatória e decisão
│
├── models/                       # Checkpoints e artefactos treinados (não versionados)
│
├── paper/                        # Artigo científico da equipa (LaTeX / PDF)
│   ├── state_of_the_art/         # Levantamento bibliográfico e fichas de leitura
│   └── draft/                    # Versões preliminares e versão final do paper
│
└── reports/                      # Relatórios complementares e apresentações
```

---

## 4. Instalação e Configuração do Ambiente

### 4.1 Pré-requisitos
- **Python:** Versão 3.10 ou superior recomendada.
- **Git:** Instalado e configurado na máquina local.

### 4.2 Criação e Ativação do Ambiente Virtual

No terminal da raiz do projeto:

**No Windows (PowerShell):**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**No Linux / macOS (Bash):**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4.3 Instalação de Dependências

Com o ambiente virtual ativo:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 4.4 Configuração de Variáveis de Ambiente

Copie o ficheiro de exemplo `.env.example` para `.env` e ajuste as variáveis conforme necessário:
```bash
cp .env.example .env     # Linux / macOS
copy .env.example .env   # Windows
```

---

## 5. Gestão de Dados (*Datasets*)

Por motivos de boas práticas de controlo de versões, **os ficheiros brutos de dados (CSV, Parquet, ZIP) não são registados no Git**.
- Consulte [DATA.md](DATA.md) para consultar o catálogo detalhado dos 5 datasets candidatos, links de download e termos de pesquisa para o Google Scholar.
- Coloque os ficheiros nas subpastas respetivas dentro de `data/` seguindo as orientações em [data/README.md](data/README.md).

---

## 6. Calendário e Prazos Principais

| Marco / Entregável | Unidade Curricular | Data Limite |
| :--- | :--- | :--- |
| **1.ª Versão do Artigo Científico (Estado da Arte)** | PLNTDIA | 18 de dezembro |
| **Versão Final do Artigo Científico** | PLNTDIA | 10 de janeiro |
| **Sessão de Demonstração Técnica (Demo Online, 30 min)** | PLNTDIA / AAUTIA | 11 de janeiro |
| **Pitch Presencial do Demo Day (5 min)** | PLNTDIA / AAUTIA | 14 de janeiro |
| **Submissão Final do Blog da Equipa** | PLNTDIA | Dia seguinte ao Demo Day |
| **Exame de Época Normal** | PLNTDIA | 18 de janeiro |
| **Exame de Recurso** | PLNTDIA | 1 de fevereiro |

---

## 7. Políticas de Contribuição e Git

Para manter o repositório organizado e com histórico limpo:
- **Ramo Principal (`main`):** Reservado para código estável e versões finais de entregáveis.
- **Ramos de Funcionalidade (`feature/<nome-da-tarefa>`):** Desenvolva novas análises ou módulos em *feature branches* e integre através de Pull/Merge Requests.
- **Commits Semânticos:** Utilize mensagens convencionais (ex.: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`).
- **Limpeza de Notebooks:** Antes de commitar notebooks, limpe as saídas com excesso de dados (`Clear All Outputs`) caso contenham saídas sensíveis ou muito extensas.
