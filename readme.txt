# Projeto Pratico: Pipeline Completo de Ciberseguranca - Bronze, Prata, Ouro e ML

Este repositorio contem a implementacao de um pipeline completo de engenharia de dados
e machine learning para dados de incidentes ciberneticos, seguindo a arquitetura Medallion
(Bronze -> Prata -> Ouro).

## Estrutura do Projeto

### Notebooks Individuais (Pipeline de Dados)
| Notebook | Conteudo |
|---|---|
| `incedets_master.ipynb` | Catalogo mestre de incidentes (severidade, vetores, downtime) |
| `financial_impact.ipynb` | Impacto financeiro, resgates e custos |
| `market_impact.ipynb` | Reacao do mercado e volatilidade de acoes |

Cada notebook contem:
- Pipeline pandas original (Bronze + Prata)
- Refatoracao PySpark (Bronze + Prata + Window + Ouro + Timing)

### Notebook Principal
| Notebook | Conteudo |
|---|---|
| `pipeline_completo.ipynb` | Pipeline completo: EDA, Gold, ML e referencia PySpark |

### Diretorio de Dados
```
Dados/
  originais/         # CSV brutos (raw data)
    incidents_master.csv
    financial_impact.csv
    market_impact.csv
  bronze/            # Camada Bronze (dados auditados, .parquet)
  prata/             # Camada Prata (dados limpos, .parquet)
  ouro/              # Camada Ouro (ML-ready, .parquet)
    gold_completo.parquet           # Join dos 3 datasets
    gold_agregacoes_setor.parquet   # Agregacoes por setor
    dataset_ml_ouro.parquet         # Dataset ML-ready (pos feature engineering)
    preprocessor.pkl                # Pipeline fitted (fit/transform)
    tabela_transformacoes.csv       # Documentacao das transformacoes
```

## Pipeline Completo (pipeline_completo.ipynb)

### 1. EDA Orientada a Hipoteses
- 3 hipoteses de negocios sobre incidentes de ciberseguranca
- 7+ graficos: boxplots, scatterplots, countplots, heatmap de correlacao
- Identificacao de outliers via IQR
- Interpretacoes textuais com conclusoes orientadas a decisao

### 2. Camada Ouro (Feature Engineering)
- **Missings**: 2 estrategias (mediana para skewed, media para normal, constante para categoricas)
- **Outliers**: Winsorizacao (cap 1%-99%) em 2+ colunas
- **Encoding**: OneHotEncoder + LabelEncoder
- **Scaling**: StandardScaler
- **Pipeline**: sklearn ColumnTransformer com fit/transform (sem data leakage)
- Dataset salvo em Parquet + tabela de transformacoes documentada

### 3. Modelagem (Arvores de Decisao)
- 2 modelos com configs distintas (max_depth=3/gini e max_depth=6/entropy)
- Divisao 70/30 estratificada (representatividade de classes)
- 4 metricas: acuracia, precisao, recall, F1-score
- Matriz de confusao do melhor modelo
- Visualizacao da arvore resultante
- Comparacao explicita Prata (dados crus) vs Ouro (pre-processado)

### 4. Refatoracao PySpark
Presente nos 3 notebooks individuais:
- Leitura Parquet com spark.read
- df.join() com tipo explicito (left)
- groupBy().agg() do PySpark
- Window.partitionBy().orderBy() para calculos analiticos
- Timing comparativo Pandas vs PySpark

## Instrucoes de Execucao

### Pre-requisitos
- Python 3.8+
- Dependencias em `requirements.txt`

### Passos
1. Instalar dependencias:
   ```
   pip install -r requirements.txt
   ```
2. Executar notebooks individuais para gerar Bronze/Prata
3. Executar `pipeline_completo.ipynb` para EDA, Ouro, ML e PySpark
4. Verificar saidas em `Dados/ouro/`

## Data Lineage (Linhagem dos Dados)

### Bronze
- Leitura do CSV original
- Adicao de metadados de auditoria: _ingestion_timestamp, _source_file, _week_day
- Persistencia em .parquet

### Prata
- Leitura da Bronze
- Tratamento de nulos (financeiros -> 0, categoricos -> "desconhecido")
- Padronizacao e tipagem de datas
- Eliminacao de duplicidades
- Criacao da variavel Target (Label) para ML
- Filtro Anti-Leakage (colunas pos-evento removidas)

### Ouro
- Join dos 3 datasets (master, financial, market) via incident_id
- Tratamento de outliers (winsorizacao)
- Encoding (OneHot + Label)
- Scaling (StandardScaler)
- Pipeline fit/transform (garantia anti-leakage entre treino/teste)
- Dataset ML-ready salvo em .parquet

## Anti-Leakage Checklist
- [x] Colunas de downtime_hours removidas do dataset ML
- [x] Colunas de total_loss_usd/insurance_payout removidas
- [x] Colunas de days_to_price_recovery removidas
- [x] Colunas de notes/review_flag removidas
- [x] Pipeline fit/transform aplicado separadamente (nada do teste vaza)

## Tabela de Transformacoes (Ouro)
Consulte `Dados/ouro/tabela_transformacoes.csv` para a documentacao detalhada
de cada transformacao aplicada na camada Ouro.
