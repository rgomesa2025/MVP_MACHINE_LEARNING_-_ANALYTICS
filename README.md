# MVP — Machine Learning & Analytics
## Previsão da Produção Offshore de Petróleo por Campo (ANP)

---

## 1. Visão Geral

Este MVP constrói um pipeline *end-to-end* para **prever a produção mensal de óleo por campo produtor offshore para os próximos 5 anos (60 meses)**, a partir de duas décadas de dados públicos da ANP.

A modelagem trata o problema na granularidade **Campo-Mês**. Essa escolha reduz o ruído operacional existente no nível de poço individual, como paradas pontuais, poços inativos e variações mensais muito específicas. Como atributos preditivos auxiliares, o modelo utiliza o histórico **defasado** de Óleo, Gás Natural, Gás Associado e Gás Não Associado, por meio de *lags* e médias móveis calculados dentro de cada campo.

A abordagem busca capturar a dinâmica temporal não-linear dos reservatórios offshore, incluindo campos maduros em declínio, períodos de estabilidade e rampas de produção de ativos de alta produtividade, sem utilizar variáveis contemporâneas que possam gerar vazamento de informação entre óleo e gás.

---

## 2. Dataset

- **Fonte:** [Produção de Petróleo e Gás Natural — ANP](https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-estatisticos) · [Consulta de Produção por Poço](https://cdp.anp.gov.br/ords/r/cdp_apex/consulta-dados-publicos-cdp/consulta-produ%C3%A7%C3%A3o-por-po%C3%A7o)
- **Período coberto:** 2005–2025
- **Escopo analítico:** produção offshore
- **Repositório de dados raw:** [rgomesa2025/MVP_MACHINE_LEARNING_E_ANALYTICS/data](https://github.com/rgomesa2025/MVP_MACHINE_LEARNING_E_ANALYTICS/tree/main/data)

> A base histórica apresenta um desafio de padronização (*data wrangling*): alguns anos vêm consolidados em um único `.csv`, outros fragmentados em arquivos mensais, com colunas que variam de formato entre os anos. O pipeline mapeia, unifica e lê os arquivos diretamente da URL pública do GitHub, dispensando uploads manuais e garantindo reprodutibilidade.

---

## 3. Como Executar

O projeto foi desenhado para rodar **nativamente no Google Colab**, sem instalação obrigatória de dependências externas, pois os principais pacotes utilizados já vêm pré-instalados.

1. Abra o notebook `Producao_Nacional_Petroleo.ipynb` no Google Colab.
2. Execute as células em ordem: `Ambiente de execução → Executar tudo`.
3. Os dados são baixados automaticamente do repositório público; não há necessidade de upload manual.

**Ambiente local opcional:**

```bash
pip install -q pandas numpy matplotlib seaborn scikit-learn requests joblib
```

---

## 4. Bibliotecas Utilizadas

- **Ingestão e requisição:** `requests`
- **Manipulação e padronização:** `pandas`, `numpy`
- **Visualização:** `matplotlib`, `seaborn`
- **Modelagem e Machine Learning:** `scikit-learn`
  - `Pipeline`
  - `ColumnTransformer`
  - `RobustScaler`
  - `OneHotEncoder`
  - `DummyRegressor`
  - `Ridge`
  - `RandomForestRegressor`
  - `TimeSeriesSplit`
  - `RandomizedSearchCV`
- **Persistência de artefatos:** `joblib`

A semente global é fixada em `SEED = 42` para melhorar a reprodutibilidade dos resultados.

---

## 5. Estrutura do Notebook

| Seção | Conteúdo |
| :--- | :--- |
| 1. Definição do Problema | Descrição, objetivo, tipo de problema, premissas, hipóteses e critérios de sucesso |
| 2. Ambiente e Reprodutibilidade | Setup, sementes, bibliotecas e funções auxiliares |
| 3. Seleção e Carga dos Dados | Ingestão consolidada da ANP, volumetria, tipagem, ausentes, duplicatas e dicionário de dados |
| 4. Análise Exploratória dos Dados | Distribuição do target, gases coproduzidos, comportamento temporal e outliers |
| 5. Preparação dos Dados | Separação features/target, alinhamento de granularidade e divisão temporal treino/teste |
| 6. Pré-processamento e Pipeline | Tratamento de variáveis numéricas e categóricas dentro de pipeline |
| 7. Baseline e Modelos Candidatos | Baseline estatístico de mediana, Ridge e Random Forest |
| 8. Treinamento e Avaliação Inicial | Treinamento dos modelos candidatos e comparação inicial das métricas |
| 9. Validação e Otimização | Otimização da Random Forest com `RandomizedSearchCV` e `TimeSeriesSplit` |
| 10. Avaliação Final | Comparação final no conjunto de teste 2024–2025 e diagnóstico de resíduos |
| 11. Comparação Final dos Modelos | Escolha do modelo final pela menor WAPE |
| 12. Boas Práticas e Rastreabilidade | Metadados, decisões técnicas e limitações |
| 13. Conclusão | Síntese dos resultados, limitações e próximos passos |
| 14. Salvamento de Artefatos e Projeção | Persistência do pipeline, gráficos e projeção 2026–2030 |

---

## 6. Metodologia

- **Granularidade Campo-Mês:** os dados são consolidados por campo produtor e mês de referência, reduzindo ruídos operacionais do nível de poço.
- **Atributos autorregressivos:** são criados *lags* `[1, 3, 6, 12]` e médias móveis `[3, 6, 12]`, calculados dentro de cada campo e respeitando a ordem cronológica.
- **Variáveis auxiliares de gás:** Gás Natural, Gás Associado e Gás Não Associado são usados somente em forma defasada, evitando vazamento de informação contemporânea.
- **Divisão temporal:** o treino utiliza o histórico até 2023, enquanto 2024–2025 é reservado como teste fora da amostra.
- **Validação temporal:** a otimização da Random Forest utiliza `TimeSeriesSplit`, sem embaralhamento, para preservar a lógica temporal do problema.
- **Pipeline de pré-processamento:** o `RobustScaler` auxilia especialmente o modelo linear `Ridge`, enquanto o `OneHotEncoder(handle_unknown="ignore")` trata variáveis categóricas como bacia e operador.
- **Projeção recursiva:** a previsão 2026–2030 é gerada mês a mês, realimentando os atributos temporais. Para evitar explosão matemática, o notebook aplica controle de coerência operacional por campo, com limites baseados no histórico recente e no percentil 95 de produção.

---

## 7. Métricas de Avaliação

| Métrica | Papel |
| :--- | :--- |
| **WAPE** | Métrica principal; mede o erro absoluto ponderado pelo volume real produzido |
| **MAE** | Métrica de apoio; mede o erro médio absoluto em barris/dia |
| **RMSE** | Métrica de apoio; penaliza erros maiores |
| **R²** | Indicador complementar de ajuste global |

**Critério de sucesso:** o modelo final deve superar claramente o baseline estatístico da mediana no conjunto de teste 2024–2025, reduzindo principalmente o WAPE e mantendo coerência com MAE, RMSE e R².

A projeção futura deve permanecer em escala física compatível com a produção offshore analisada, sem volumes negativos e sem crescimento matemático artificial causado pelo processo recursivo.

---

## 8. Resultados

O modelo final não é assumido previamente. Ele é selecionado automaticamente no notebook pela **menor WAPE no conjunto de teste 2024–2025**, comparando:

- `Ridge`
- `RandomForestRegressor` inicial
- `RandomForestRegressor` otimizada por `RandomizedSearchCV`

Essa decisão evita escolher um modelo apenas por expectativa teórica. A Random Forest é uma candidata forte por capturar relações não-lineares, mas o `Ridge` também é mantido na comparação por ser uma referência linear regularizada competitiva quando os atributos temporais estão bem construídos.

Na execução corrigida, a lógica central é: o modelo campeão será aquele que apresentar o menor WAPE no teste, mantendo coerência com as demais métricas.

A projeção 2026–2030 deve ser interpretada como **cenário técnico gerado pelo MVP**, e não como previsão determinística da produção offshore futura. Na execução validada, a curva projetada permaneceu em escala coerente, variando aproximadamente de **5,30 milhões bbl/dia em 2026** para **8,05 milhões bbl/dia em 2030**. Esse resultado representa um cenário crescente aprendido a partir dos padrões históricos offshore disponíveis, exigindo cautela por causa da natureza recursiva da previsão.

**Principais aprendizados:** a governança dos dados e a granularidade Campo-Mês foram decisivas para o sucesso do pipeline. O uso de variáveis defasadas reduziu o risco de vazamento de dados e permitiu transformar uma base tabular histórica em um problema supervisionado com memória temporal.

---

## 9. Limitações e Próximos Passos

**Limitações:** o modelo é empírico e tabular. Ele carrega memória temporal por meio de *lags* e médias móveis, mas não incorpora variáveis físicas e operacionais como pressão de reservatório, intervenções em poços, entrada de novas plataformas, paradas programadas, restrições logísticas ou decisões de investimento.

Na projeção recursiva, o erro tende a se acumular ao longo dos 60 meses, aumentando a incerteza nos anos mais distantes. Por isso, os resultados devem ser usados como apoio analítico e não como substituto de modelos físicos de reservatório.

**Próximos passos:**

- Comparar a abordagem empírica com modelos físicos de declínio, como curvas de Arps.
- Avaliar modelos de reforço de gradiente, como `HistGradientBoostingRegressor` ou `XGBoost`.
- Substituir a banda baseada em RMSE por métodos de previsão quantílica.
- Integrar variáveis operacionais e regulatórias, como paradas programadas, entrada de novas unidades de produção e restrições logísticas.
- Avaliar a projeção por campo individualmente, além da visão agregada offshore.

---

## 10. Artefatos Gerados

Ao final da execução, os artefatos são persistidos na pasta `artefatos_mvp/`:

- Pipeline final integrado com o modelo selecionado (`joblib`)
- `RobustScaler` isolado
- `OneHotEncoder` isolado
- Tabela de resultados comparativos
- Gráfico de diagnóstico de resíduos
- Projeção numérica agregada de produção offshore 2026–2030
