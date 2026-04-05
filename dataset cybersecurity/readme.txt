# Projeto Prático: Construção das Camadas Bronze e Prata para Dados de Machine Learning

Este repositório contém a implementação de um pipeline de engenharia de dados que organiza e trata dados brutos de incidentes cibernéticos, dividindo-os em camadas **Bronze** (dados brutos auditados) e **Prata** (dados limpos e prontos para modelagem preditiva).

O projeto atende rigorosamente às boas práticas de governança de dados, documentação de linhagem (*data lineage*), prevenção de vazamento de dados (*anti-leakage*) e análise exploratória.

---

## 📁 Estrutura do Projeto

O projeto foi dividido em 3 notebooks independentes, cada um tratando de uma vertente específica do ecossistema de incidentes cibernéticos:

### 1. `financial_impact.ipynb`
* **Objetivo**: Avaliar o impacto financeiro direto e custos de extorsão sofridos pelas empresas.
* **Mapeamento de Padrões**: Relação entre o valor exigido por criminosos e o valor efetivamente pago pelas empresas.
* **Target de ML**: Previsão de probabilidade de pagamento de resgate.

### 2. `market_impact.ipynb`
* **Objetivo**: Analisar a reação do mercado financeiro e a volatilidade das ações de empresas listadas em bolsa após a divulgação de um ataque.
* **Mapeamento de Padrões**: Tempo de recuperação do preço das ações baseado no tamanho de mercado (*Market Cap*).
* **Target de ML**: Previsão de impacto prolongado no preço do papel (recuperação lenta).

### 3. `incedets_master.ipynb`
* **Objetivo**: Centralizar e tratar o catálogo mestre de incidentes, englobando severidade e vetores de ataque primários.
* **Mapeamento de Padrões**: Horas de indisponibilidade (*downtime*) provocadas por diferentes categorias de ataque.
* **Target de ML**: Previsão de ocorrência de indisponibilidade severa de sistemas.

---

## ⚙️ Instruções de Execução

Siga os passos abaixo para preparar o ambiente e reproduzir as execuções dos notebooks.

### Pré-requisitos
Certifique-se de ter o Python 3.8+ instalado em sua máquina.

### Passo 1: Clonar o Repositório
```bash
git clone <url-do-seu-repositorio>
cd <nome-da-pasta-do-seu-repositorio>



DATA LINEAGE

Camada Bronze:

    Leitura do arquivo .csv original.

    Adição de metadados de auditoria: data/hora da carga, arquivo de origem e dia da semana.

    Persistência em formato .parquet mantendo a estrutura bruta.

Camada Prata:

    Tratamento de valores nulos (nulos financeiros convertidos em zero ou média, categóricos 
    preenchidos com "desconhecido").

    Padronização e tipagem de datas.

    Eliminação de duplicidades.

    Criação da variável Target (Label) para Machine Learning.

Filtro Anti-Leakage:

    Geração de um arquivo .parquet secundário na camada Prata onde todas as colunas que representam 
    informações de eventos futuros ao ataque foram expurgadas, garantindo que o dataset de ML não cause 
    falsas acurácias em modelos preditivos.