# Gestão e Armazenamento Local de Dados

Este diretório destina-se ao armazenamento local dos dados e metadados utilizados no projeto de **Otimização de Descontos no Retalho Têxtil**.

> **Nota Crítica de Gestão de Versões (Git):**
> Os ficheiros de dados brutos (`.csv`, `.parquet`, `.zip`, etc.) estão **estritamente ignorados pelo Git** (através do `.gitignore` na raiz do projeto) para evitar exceder limites de tamanho de repositório e sobrecarregar o tráfego da equipa.
> **Nunca force a inclusão de ficheiros de dados de grandes dimensões no controlo de versões.**

---

## Estrutura de Pastas

```text
data/
├── dataset1/         # Ficheiros do Dataset 1 (ex.: H&M Personalized Fashion Recommendations)
├── dataset2/         # Ficheiros do Dataset 2 (ex.: Zara Sales Dataset)
└── dataset3/         # Ficheiros do Dataset 3 (ex.: Retail Markdown / Sales Dataset)
```

## Instruções de Download e Configuração

Consulte o documento completo [DATA.md](../../DATA.md) para detalhes exaustivos sobre os critérios de seleção, origens de dados, dicionários de atributos e links oficiais de download.

1. **Dataset 1:** Descarregue os ficheiros e coloque-os descompactados na pasta `data/dataset1/`.
2. **Dataset 2:** Descarregue os ficheiros e coloque-os na pasta `data/dataset2/`.
3. **Dataset 3:** Descarregue os ficheiros e coloque-os na pasta `data/dataset3/`.

Caso utilize a API da Kaggle, configure as suas credenciais no ficheiro `.env` a partir do template `.env.example`.

