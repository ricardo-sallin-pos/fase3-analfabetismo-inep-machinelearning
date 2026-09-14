# Prevendo Analfabetismo nos Municípios Brasileiros

Projeto desenvolvido no **FIAP AI Scientist — Fase 3**, com o objetivo de investigar quais características socioeconômicas, educacionais e geográficas estão relacionadas ao desempenho de alfabetização dos municípios brasileiros e utilizar **Machine Learning** para identificar padrões, estimar resultados e apoiar a priorização de políticas públicas.

> **Transformando dados públicos em hipóteses sobre políticas públicas de alfabetização.**

---

## Objetivo

O projeto busca responder a uma questão central:

**É possível estimar o desempenho de alfabetização de um município a partir de suas características socioeconômicas, educacionais, geográficas e de financiamento público?**

A partir dessa pergunta, foram investigadas hipóteses relacionadas a:

* **Renda média:** municípios mais ricos apresentam melhor desempenho?
* **Alfabetização dos adultos:** o nível educacional dos adultos está relacionado à alfabetização das crianças?
* **Financiamento público:** maiores investimentos em educação estão associados a melhores resultados?
* **Acesso à educação:** municípios com poucas escolas proporcionalmente à população ou territorialmente dispersos apresentam maior dificuldade?
* **Localização:** municípios de regiões metropolitanas apresentam comportamento diferente dos municípios do interior?

Essas hipóteses motivaram o cruzamento de variáveis educacionais, sociais, econômicas e geográficas.

---

## Dados

O projeto reúne dados de diferentes fontes públicas, incluindo informações sobre:

* alfabetização infantil;
* alfabetização de adultos e idosos;
* quantidade de alunos e escolas;
* população e área territorial;
* renda média;
* financiamento via Fundeb;
* complementações VAAF, VAAT e VAAR;
* despesas com educação;
* distância até a capital;
* localização geográfica dos municípios;
* metas municipais de alfabetização.

A base de renda utiliza dados da **PNAD Contínua**, enquanto as coordenadas geográficas dos municípios foram obtidas a partir de uma base disponibilizada pelo Gov.br.

### Estimativa de renda

Como a base utilizada para renda contemplava apenas 201 municípios, esses municípios foram utilizados como **Cidades-Pai** para estimar a renda dos demais municípios brasileiros.

Cada município foi associado à Cidade-Pai geograficamente mais próxima utilizando **Haversine + Nearest Neighbors**, e a renda estimada recebeu um redutor baseado na distância.

---

## Arquitetura de dados

Os dados foram organizados segundo uma arquitetura **Medallion**:

```text
                 FONTES DE DADOS
                       │
                       ▼
                  ┌─────────┐
                  │ BRONZE  │
                  │  Dados  │
                  │  brutos  │
                  └────┬────┘
                       │
                       ▼
                  ┌─────────┐
                  │ SILVER  │
                  │ Dados   │
                  │ tratados│
                  │ e cruzados
                  └────┬────┘
                       │
                       ▼
                  ┌─────────┐
                  │  GOLD   │
                  │ Dados   │
                  │ para ML │
                  └────┬────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       K-means    Random Forest  Regressão
                                 Logística
```

**Bronze** concentra os dados originais; **Silver** realiza os cruzamentos, tratamento de nulos e consolidação em uma fonte única de verdade; e **Gold** disponibiliza conjuntos específicos para cada modelo.

---

## Análise Exploratória

Antes dos modelos, foram realizadas análises de:

* valores ausentes;
* correlação de Spearman;
* multicolinearidade através de **VIF**;
* matriz de dispersão;
* Mutual Information;
* relação entre financiamento Fundeb e desempenho de alfabetização.

---

# Modelos

## 1. K-means — Clusterização de municípios

O primeiro modelo teve caráter **não supervisionado**.

O objetivo foi identificar grupos de municípios com características socioeconômicas e educacionais semelhantes.

As variáveis com distribuições muito assimétricas foram transformadas com `log1p` antes da padronização.

O número de clusters foi investigado através da **curva de cotovelo (WCSS)** e do **coeficiente de silhueta**.

---

## 2. Random Forest — Importância das variáveis

O segundo modelo utilizou **Random Forest Regressor** para investigar quais variáveis apresentam maior importância na explicação do desempenho de alfabetização.

O modelo foi treinado com dados de 2023 e utilizado para ranquear as variáveis segundo sua importância.

---

## 3. Regressão Logística — Probabilidade de atingir a meta

O terceiro modelo transformou o problema em uma classificação binária:

```text
bateu a meta → 1
não bateu → 0
```

A partir das variáveis socioeconômicas e educacionais, o modelo estima a **probabilidade de um município atingir sua meta de alfabetização**.

Além da classificação, o modelo produz um ranking dos municípios de acordo com a probabilidade estimada de atingir a meta.

Foram avaliadas métricas como Accuracy, Precision, Recall, F1-Score, ROC-AUC e Matriz de Confusão.

Também foram analisados os **coeficientes e Odds Ratios** da regressão para investigar o impacto das variáveis na probabilidade estimada.


---

# Implicações para políticas públicas

Os resultados sugerem algumas possibilidades de investigação e aplicação:

* acompanhar não apenas os repasses, mas também as **despesas efetivamente empenhadas e executadas**;
* avaliar municípios que apresentam **resultados expressivos mesmo com menor financiamento**;
* identificar municípios com características semelhantes, mas desempenhos muito diferentes;
* direcionar políticas públicas de acordo com o perfil socioeconômico e educacional de cada grupo;
* utilizar os modelos para apoiar a identificação de regiões que podem demandar maior atenção.

---

# Limitações

O projeto utiliza dados agregados por município e dispõe de apenas **dois anos de dados de alfabetização (2023 e 2024)** para a análise temporal. Com mais histórico, aumenta a capacidade de estabelecer relações causais e separar efeitos estruturais de variações anuais.

Além disso, parte dos dados de renda precisou ser **estimada para municípios que não estavam presentes na PNAD**, utilizando proximidade geográfica e uma regra de redução baseada na distância.

Portanto, os modelos devem ser entendidos principalmente como instrumentos de **exploração, identificação de padrões e geração de hipóteses**, e não como prova de causalidade.

---

# Tecnologias utilizadas

* **Python**
* **Pandas**
* **NumPy**
* **Scikit-learn**
* **Statsmodels**
* **Matplotlib**
* **Seaborn**
* **K-Means**
* **Random Forest**
* **Regressão Logística**
* **Nearest Neighbors**
* **BigQuery**
* **SQL**
* **Google Colab**
* **Arquitetura Medallion**

---

## Estrutura conceitual do projeto

```text
Dados públicos
      │
      ▼
Integração e tratamento
      │
      ▼
EDA + hipóteses
      │
      ├───────────────┐
      ▼               ▼
  Clusterização   Modelagem
    K-Means       supervisionada
                      │
                ┌─────┴─────┐
                ▼           ▼
           Random Forest  Logística
                │           │
                ▼           ▼
          Importância    Probabilidade
          das variáveis  de atingir meta
                │           │
                └─────┬─────┘
                      ▼
             Hipóteses para
            políticas públicas
```

---

## Contexto acadêmico

Projeto desenvolvido como parte da formação **FIAP AI Scientist — Fase 3**, explorando técnicas de Engenharia de Dados, Análise Exploratória, Machine Learning e interpretação de modelos aplicadas a um problema real de política pública educacional.
