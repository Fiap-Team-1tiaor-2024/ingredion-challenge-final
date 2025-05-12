# Projeto de IA para Previsão de Produtividade Agrícola - Challenge Ingredion (Sprint 3 Finalizada)

**Equipe:** [Seu Nome / Nome do Grupo]
**Data:** 12 de Maio de 2025
**(Link do Repositório: https://github.com/Fiap-Team-1tiaor-2024/ingredion-challenge)**

## Visão Geral do Projeto

Este projeto, desenvolvido para o Challenge FIAP em parceria com a Ingredion, teve como objetivo explorar a aplicação de Inteligência Artificial e técnicas de análise de dados para prever a produtividade agrícola da cultura do cacau no município de Ilhéus, BA. A Sprint 3 focou na incorporação de dados históricos detalhados de produtividade (IBGE), índices de vegetação (NDVI e EVI) e, crucialmente, dados meteorológicos, para construir e avaliar múltiplos modelos de regressão.

Este README consolida a metodologia, os resultados e as conclusões da Sprint 3, refletindo a análise mais recente com a inclusão de todas as fontes de dados.

## Estrutura do Repositório

- `/`: Contém este README.md.
- `src/`:
  - `Challenge_Sprint3_Analise_Rendimento_Completo.ipynb` (Ou nome similar, contendo todo o código da Sprint 3).
- `data/`:
  - `ibge-dados-cacau.csv` (Dados de produtividade IBGE)
  - `NDVI.csv` (Dados NDVI)
  - `EVI.csv` (Dados EVI)
  - `inmet-ilhéus-chuvas-ventos.csv` (Dados meteorológicos)
- `images/`: (Para salvar os gráficos e tabelas que serão inseridos neste README)

## 1. Metodologia de Coleta e Preparação dos Dados (Sprint 3)

### 1.1. Fontes de Dados Utilizadas

1.  **Produtividade Agrícola (IBGE):**
    - Arquivo: `ibge-dados-cacau.csv`.
    - Fonte: Instituto Brasileiro de Geografia e Estatística (IBGE), Sistema SIDRA - Produção Agrícola Municipal (PAM).
    - Descrição: Dados anuais de rendimento médio (kg/ha), área colhida (ha) e quantidade produzida (toneladas) para cacau (em amêndoa) no município de Ilhéus, BA. Período original 1974-2022.
2.  **Índices de Vegetação (NDVI e EVI):**
    - Arquivos: `NDVI.csv`, `EVI.csv`.
    - Fonte: [Usuário: Especificar a plataforma de origem, ex: SatVeg, Google Earth Engine, etc.]
    - Descrição: Séries temporais de NDVI e EVI para um ponto de referência em Ilhéus, BA, cobrindo aproximadamente 2000-2025.
3.  **Dados Meteorológicos:**
    - Arquivo: `inmet-ilhéus-chuvas-ventos.csv`.
    - Fonte: [Usuário: Especificar a fonte, ex: INMET - Estação A410 (Ilhéus)].
    - Descrição: Dados horários contendo variáveis como precipitação, temperatura (bulbo, máxima, mínima), umidade relativa, radiação global, etc. Período original [Usuário: Especificar período original do arquivo de clima, ex: 2003-2023].

### 1.2. Pré-processamento e Engenharia de Features

1.  **Dados do IBGE:**
    - Seleção de colunas relevantes e renomeação (`Ano`, `Produtividade_Real_kg_ha`, `Area_Colhida_ha_Real`, `Qtde_Produzida_Ton_Real`).
    - Filtragem para Ilhéus e produto "Cacau (em amêndoa)".
    - Período filtrado para alinhamento: 2003-2022 (considerando o início dos dados meteorológicos).
2.  **Dados de NDVI e EVI:**
    - Conversão da coluna `Data` para datetime, valores de VI para numérico.
    - Extração de `Ano` e `Mes`.
    - Filtragem para o período 2003-2022.
    - **Agregação Anual:** Valores de NDVI e EVI (após filtrar por `VI > 0.1`) foram agregados anualmente para calcular: `_medio_anual`, `_max_anual`, `_min_anual`, `_std_anual`, `_count_anual`.
3.  **Dados Meteorológicos:**
    - Criação de uma coluna `Timestamp` combinando data e hora.
    - Seleção de colunas: `precipitacao_total`, `temperatura_bulbo_hora`, `umidade_rel_hora`, `radiacao_global`.
    - Tratamento inicial de NaNs horários (ex: precipitação e radiação preenchidas com 0).
    - **Agregação Diária:** Cálculo de `chuva_total_diaria_mm`, `temp_max_diaria_C`, `temp_min_diaria_C`, `temp_media_diaria_C`, `umidade_media_diaria_perc`, `radiacao_total_diaria_kJ_m2`. NaNs diários foram preenchidos com `ffill` e `bfill`.
    - **Agregação Anual:** Cálculo de `precip_total_anual_mm`, `temp_max_media_anual_C`, `temp_min_media_anual_C`, `temp_media_anual_C`, `umidade_media_anual_perc`, `radiacao_total_anual_kJ_m2`, e `dias_com_chuva_gt_1mm_anual`.
    - Período filtrado: 2003-2022.
4.  **Integração e Criação do DataFrame Final:**
    - O DataFrame do IBGE foi unido (`merge`) com os dados agregados anuais de NDVI, EVI e, subsequentemente, com as features climáticas anuais, usando `Ano` como chave.
    - Foram adicionadas features defasadas (lagged): `Produtividade_Real_kg_ha_lag1` e `Area_Colhida_ha_Real_lag1`.
    - Linhas com NaNs resultantes dos lags ou da defasagem inicial dos dados meteorológicos (anos 2003 sem lag, por exemplo) foram removidas.
    - O DataFrame final para modelagem (`df_sprint3_model_input`) cobriu o período de **2004 a 2022 (19 anos)**.

## 2. Justificativa das Variáveis Selecionadas para Modelagem

- **Variável Alvo (y):**
  - `Produtividade_Real_kg_ha`: Rendimento médio da produção de cacau (kg/ha).
- **Features (X) Utilizadas (20 features):**
  - `Area_Colhida_ha_Real`
  - `ndvi_medio_anual`, `ndvi_max_anual`, `ndvi_min_anual`, `ndvi_std_anual`, `ndvi_count_anual`
  - `evi_medio_anual`, `evi_max_anual`, `evi_min_anual`, `evi_std_anual`, `evi_count_anual`
  - `precip_total_anual_mm`, `temp_max_media_anual_C`, `temp_min_media_anual_C`, `temp_media_anual_C`, `umidade_media_anual_perc`, `radiacao_total_anual_kJ_m2`, `dias_com_chuva_gt_1mm_anual`
  - `Produtividade_Real_kg_ha_lag1`, `Area_Colhida_ha_Real_lag1`
- **Feature Excluída para Evitar Data Leakage:** `Qtde_Produzida_Ton_Real`.

## 3. Justificativa dos Modelos e Lógica Preditiva

Foram testados diversos modelos de regressão para capturar diferentes tipos de relações nos dados:

- `Linear Regression` e `Ridge Regression`: Modelos lineares, sendo Ridge uma versão regularizada.
- `Decision Tree Regressor`: Modelo baseado em árvore, capaz de capturar relações não-lineares.
- `Support Vector Regression (SVR)`: Modelo baseado em máquinas de vetores de suporte.
- `Random Forest Regressor`, `Gradient Boosting Regressor`, `XGBoost Regressor`: Modelos de ensemble baseados em árvores, geralmente robustos e com bom desempenho.

A lógica é que cada modelo aprenda a relação entre as features anuais (VIs, clima, área, lags) e o rendimento do cacau. A avaliação foi feita em um conjunto de teste separado temporalmente.

## 4. Resultados da Previsão de Rendimento e Discussão Crítica

Os modelos foram treinados com dados de 2004 a 2012 (9 anos) e testados no período de 2013 a 2022 (10 anos), utilizando um total de 19 pontos de dados anuais.

### 4.1. Comparativo de Métricas dos Modelos (Conjunto de Teste)

| Modelo            | RMSE      | MAE       | R²        |
| ----------------- | --------- | --------- | --------- |
| Gradient Boosting | 27.321217 | 22.433582 | 0.288690  |
| Random Forest     | 28.922070 | 21.738000 | 0.202891  |
| Decision Tree     | 39.678710 | 29.800000 | -0.500286 |
| XGBoost           | 45.133490 | 39.028369 | -0.941140 |
| SVR               | 45.487693 | 39.351425 | -0.971727 |
| Ridge Regression  | 58.431088 | 41.320412 | -2.253471 |
| Linear Regression | 61.017892 | 42.970924 | -2.547916 |

_(Resultados da sua execução da Célula S3.10)_

### 4.2. Interpretação dos Resultados

A inclusão dos dados meteorológicos resultou em uma **melhoria significativa** no desempenho de alguns modelos, notadamente `Gradient Boosting` (R² ≈ 0.29) e `Random Forest` (R² ≈ 0.20), que agora apresentam R² positivo. Isso indica que as features climáticas anuais adicionaram valor preditivo. No entanto, a maioria dos outros modelos ainda performou mal no conjunto de teste, com R² negativo.

- **Gráfico Comparativo Previsto vs Real:**  
  ![realxprevisoes](/images/realxprevisoes.png)
  - O gráfico mostra que, embora Gradient Boosting e Random Forest tenham um R² positivo, a capacidade de prever as flutuações exatas ano a ano ainda é limitada, mas melhor do que sem os dados climáticos.

### 4.3. Importância das Features / Coeficientes

- **Linear Regression**  
  ![linearregression](/images/linearregression.png)
- **Ridge Regression**  
  ![ridgeregression](/images/ridgeregression.png)
- **Decision Treen**  
  ![decisiontree](/images/decisiontree.png)
- **Linear Regression**  
  ![randomforrest](/images/randomforrest.png)
- **Random Forrest**  
  ![gradientboosting](/images/gradientboosting.png)
- **XGBoost**  
  ![xgboost](/images/xgboost.png)

- **Observações da Importância das Features:**
  - `Area_Colhida_ha_Real`: Permanece uma feature de alta importância em muitos modelos baseados em árvore.
  - **Features Climáticas:** Variáveis como `temp_max_media_anual_C`, `temp_media_anual_C` e `precip_total_anual_mm` (especialmente no XGBoost) demonstraram relevância. `dias_com_chuva_gt_1mm_anual` também apareceu nos modelos lineares.
  - **Features Lagged:** `Produtividade_Real_kg_ha_lag1` e `Area_Colhida_ha_Real_lag1` mostraram importância, principalmente no Random Forest.
  - **Índices de Vegetação (NDVI/EVI):** Apresentaram importância variada, geralmente menor em comparação com área, clima e lags, mas ainda contribuindo para alguns modelos.

### 4.4. Discussão Crítica e Limitações

1.  **Impacto Positivo do Clima:** A adição de features climáticas anuais melhorou o R² para os modelos de Gradient Boosting e Random Forest, tornando-os preditivos (R² > 0). Isso confirma a hipótese de que o clima é um fator importante.
2.  **Tamanho do Dataset vs. Número de Features:** O dataset final para modelagem contém apenas 19 observações anuais (2004-2022). Utilizar 20 features com um dataset tão pequeno é um desafio.
3.  **Divisão Treino/Teste:** A divisão de 9 anos para treino e 10 anos para teste (`TEST_YEARS_SPLIT = 10`) é agressiva. Com tão poucos dados de treino, os modelos podem ter dificuldade em aprender padrões robustos, e os resultados no teste podem ser voláteis.
4.  **Granularidade das Features:** A agregação anual de VI e clima, embora útil, ainda pode "achatar" dinâmicas intra-anuais cruciais para o desenvolvimento do cacau.
5.  **Qualidade e Representatividade dos Dados:** Dados pontuais de VI e dados de uma única estação meteorológica podem não representar perfeitamente as condições de toda a área produtora de cacau em Ilhéus.

### 4.5. Sugestões de Melhorias para Trabalhos Futuros

1.  **Aumentar o Histórico de Dados:** A principal limitação é o número de anos. Tentar obter séries históricas mais longas de produtividade, VI e, crucialmente, clima para Ilhéus.
2.  **Features Climáticas Mais Granulares:** Explorar features climáticas mensais, trimestrais ou baseadas em estágios fenológicos específicos do cacau.
3.  **Engenharia de Features Avançada:** Calcular Growing Degree Days (GDD), índices de estresse hídrico, etc.
4.  **Modelos Alternativos:** Com mais dados, modelos de séries temporais mais sofisticados (SARIMAX com regressores exógenos, Prophet) poderiam ser explorados.
5.  **Análise de Erros:** Investigar os anos em que os melhores modelos (GB, RF) erraram mais significativamente para tentar identificar fatores não capturados.

## 5. Como Executar (Usando Google Colab)

1.  **Abra o Notebook:** No Google Colab.
2.  **Monte o Google Drive:** Execute a Célula S3.0.
3.  **Instale Dependências:** `!pip install xgboost` se necessário.
4.  **Configure os Caminhos:** **AJUSTE** `DRIVE_BASE_PATH_S3` (e outros caminhos de dados) na Célula S3.0 para corresponder à sua estrutura no Drive.
5.  **Execute as Células:** Execute as células da Sprint 3 (S3.0 em diante) sequencialmente.

## 6. Referências das Bases Públicas Utilizadas

- **Instituto Brasileiro de Geografia e Estatística (IBGE):**
  - Sistema IBGE de Recuperação Automática (SIDRA). Tabela de Produção Agrícola Municipal (PAM). Cultura: Cacau (em amêndoa). Município: Ilhéus - BA.
  - Acesso: [ sidra.ibge](https://sidra.ibge.gov.br)
- **Índices de Vegetação (NDVI e EVI):**
  - Fonte: Dados derivados da plataforma SatVeg". Arquivos: `NDVI.csv`, `EVI.csv`.
  - Acesso: [satveg](https://www.satveg.cnptia.embrapa.br/)
- **Dados Meteorológicos:**
  - Fonte: Instituto Nacional de Meteorologia (INMET) - Estação Automática A410 (Ilhéus, BA) via BDMEP"]. Arquivo: `inmet-ilhéus-chuvas-ventos.csv`.
  - Acesso: [Basedosdados.org / INMET](https://basedosdados.org/dataset/782c5607-9f69-4e12-b0d5-aa0f1a7a94e2?table=2c7fdc3d-f2ed-4c78-84b8-d9c792a06703)

## 8. Autores

- Gabriela da Cunha Rocha - <RM561041@fiap.com.br>
- Gustavo Segantini Rossignolli - <RM560111@fiap.com.br>
- Vitor Lopes Romão - <RM559858@fiap.com.br>

---
