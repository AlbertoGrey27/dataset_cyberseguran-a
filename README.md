# Projeto Prático: Pipeline Completo de Cibersegurança — Bronze / Prata / Ouro / ML

Uma implementação de pipeline de engenharia de dados e machine learning para incidentes cibernéticos, seguindo a arquitetura Medallion (Bronze → Prata → Ouro). Este repositório contém notebooks Pandas e refatorações PySpark para cada camada, além do pipeline completo de ML.

---

## Índice

- [Descrição](#descrição)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Notebooks principais](#notebooks-principais)
- [Diretório de dados](#diretório-de-dados)
- [Fluxo de execução (passos)](#fluxo-de-execução-passos)
- [Detalhes das camadas](#detalhes-das-camadas)
- [Pipeline ML (Ouro)](#pipeline-ml-ouro)
- [Data Lineage (Linhagem)](#data-lineage-linhagem)
- [Anti-Leakage Checklist](#anti-leakage-checklist)
- [Saídas esperadas](#saídas-esperadas)
- [Como contribuir / Próximos passos](#como-contribuir--próximos-passos)

---

## Descrição

Este projeto demonstra um pipeline completo de dados e ML para análise de incidentes de cibersegurança. O objetivo é converter CSVs brutos em datasets prontos para modelagem, aplicando camadas de qualidade (Bronze → Prata → Ouro), pipeline de features e modelos de classificação.

## Estrutura do projeto

- Notebooks (cada um com versão Pandas e seções PySpark):
  - `incedets_master.ipynb`
  - `financial_impact.ipynb`
  - `market_impact.ipynb`
  - `pyspark_gold_layer.ipynb`
  - `pipeline_completo.ipynb`
- Dados: diretório `Dados/` com subpastas `originais/`, `bronze/`, `prata/`, `ouro/`.
- Arquivos auxiliares gerados: parquet, preprocessor.pkl, tabela_transformacoes.csv

## Notebooks principais

| Notebook | Conteúdo | Observação |
|---|---|---|
| incedets_master.ipynb | Catálogo mestre de incidentes (severidade, downtime, labels) | Gera Silver (Pandas) e *_spark.parquet (PySpark) |
| financial_impact.ipynb | Impacto financeiro, resgates e custos | Gera Silver + Spark |
| market_impact.ipynb | Reação do mercado, volatilidade | Gera Silver + Spark |
| pyspark_gold_layer.ipynb | Window functions, join Gold, desempenho PySpark | Lê Silvers Spark e gera Ouro |
| pipeline_completo.ipynb | EDA, Feature Engineering (Ouro), Modelagem ML | Gera dataset_ml_ouro.parquet e preprocessor.pkl |

## Diretório de dados

Estrutura esperada em `Dados/`:

```
Dados/
  originais/
    incidents_master.csv
    financial_impact.csv
    market_impact.csv
  bronze/        # parquet com metadados de ingestão
  prata/         # dados limpos (parquet)
  ouro/          # datasets prontos para ML (parquet + artefatos)
```

Arquivos de saída esperados em `Dados/ouro/`:

- `gold_completo.parquet`
- `gold_agregacoes_setor.parquet`
- `dataset_ml_ouro.parquet`
- `preprocessor.pkl`
- `tabela_transformacoes.csv`

## Fluxo de execução (passos)

### Pré-requisitos

- Python 3.8+
- Instalar dependências:

```bash
pip install -r requirements.txt
```

### Ordem de execução (obrigatória)

1. Executar os notebooks individuais (Camada Prata):
   - `incedets_master.ipynb` — executar todas as células (incluindo seção PySpark)
   - `financial_impact.ipynb` — executar todas as células
   - `market_impact.ipynb` — executar todas as células

   > Observação: as seções PySpark estão após as seções Pandas. Use "Run All" ou execute até o final para garantir que os arquivos `*_spark.parquet` sejam gerados.

2. Executar `pyspark_gold_layer.ipynb` (Camada Ouro PySpark): lê os Silvers Spark e gera o Gold, agregações e rankings.
3. Executar `pipeline_completo.ipynb` (ML): EDA, Feature Engineering para Ouro, treinamento e avaliação dos modelos.

## Detalhes das camadas

### Bronze
- Leitura dos CSVs originais
- Adição de metadados de auditoria: `_ingestion_timestamp`, `_source_file`, `_week_day`
- Persistência em Parquet

### Prata
- Leitura da Bronze
- Tratamento de nulos (financeiros → 0, categóricos → "desconhecido")
- Padronização e tipagem de datas
- Remoção de duplicidades
- Criação da variável Target (Label)
- Filtros anti-leakage (remoção de colunas pós-evento)

### Ouro (PySpark)
- Join dos 3 Silvers via `incident_id` (join explícito)
- Agregações por setor via `groupBy().agg()`
- Rankings e cálculos analíticos via Window functions
- Persistência final em Parquet

## Pipeline ML (Ouro)

Principais transformações aplicadas na camada Ouro para ML:

- Missing values: mediana para variáveis assimétricas, média para normais, constante para categóricas
- Outliers: winsorização (cap 1%–99%) em colunas selecionadas
- Encoding: OneHotEncoder + LabelEncoder quando adequado
- Scaling: StandardScaler
- Pipeline: `ColumnTransformer` + `Pipeline` do scikit-learn (fit/transform separado para evitar data leakage)

Modelagem:
- Algoritmos: Árvores de decisão (duas configurações)
- Splits: Treino/Teste estratificado 70/30
- Métricas: acurácia, precisão, recall, F1-score

## Data Lineage (Linhagem)

- Bronze ← CSV original (com metadados de ingestão)
- Prata ← Bronze (tratamento, tipagem, limpeza)
- Ouro ← Prata (joins, agregações, ranking, feature engineering)

## Anti-Leakage Checklist

- [x] Remover colunas `downtime_hours` do dataset ML
- [x] Remover `total_loss_usd` e `insurance_payout`
- [x] Remover `days_to_price_recovery`
- [x] Remover `notes` / `review_flag`
- [x] Aplicar pipeline fit/transform sem vazamento entre treino/teste

## Saídas esperadas e artefatos

- Parquets: `Dados/ouro/gold_completo.parquet`, `Dados/ouo/gold_agregacoes_setor.parquet`, `Dados/ouro/dataset_ml_ouro.parquet`
- Artefatos: `Dados/ouro/preprocessor.pkl`, `Dados/ouro/tabela_transformacoes.csv`

## Dicas de execução rápida

- Para gerar os Parquets Spark primeiro, certifique-se de executar as seções PySpark em cada notebook individual antes do `pyspark_gold_layer.ipynb`.

Exemplo de execução (linha de comando Python/Notebook):

```bash
# instalar deps
pip install -r requirements.txt
# abrir Jupyter / VS Code e executar os notebooks na ordem descrita
jupyter notebook
```

## Como contribuir / Próximos passos

- Verificar qualidade dos dados na camada Bronze
- Automatizar execução via scripts ou orquestrador (Airflow, Prefect)
- Empacotar preprocessador como um módulo reutilizável

---

Se quiser, eu posso:
- Comitar o `README.md` no repo
- Gerar badges e um resumo executivo (em inglês)
- Extrair o `tabela_transformacoes.csv` automaticamente a partir do notebook

Obrigado!