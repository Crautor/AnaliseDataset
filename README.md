# Steam Price Prediction — Grupo 05

Projeto de Machine Learning desenvolvido pelo **Grupo 05** para prever o preço de lançamento de jogos na plataforma Steam com base em características técnicas e de mercado.

## Equipe

| Nome |
|:-----|
| João Vitor Candido |
| João Vitor Dysarz |
| João Vitor Haas |
| Nathan Daniel Breier |
| Vitor Hugo Dallanol |

---

## Definição do Problema

**Pergunta de Negócio:** Qual o preço justo de um jogo com base nas suas características no lançamento?

**Contexto:** Precificar um produto digital é um dos maiores desafios para estúdios independentes. Um preço muito alto cria barreira de entrada; muito baixo desvaloriza o produto. Nosso modelo oferece um *termômetro de mercado* baseado em dados reais da Steam.

**Tipo de Problema:** Regressão Supervisionada (target: `price_usd`)

---

## Dataset

- **Nome:** Steam Top Games 2026
- **Fonte:** [Kaggle — Steam Top 1495 Games Dataset](https://www.kaggle.com/datasets/patelris/steam-top-1495-games-dataset)
- **Volume:** 1.495 jogos × 29 colunas
- **Escopo:** Apenas jogos pagos (1.113 após remoção dos Free-to-Play). Após a limpeza e remoção de outliers da Sprint 2, a base de modelagem fica com **1.099 jogos**, divididos em 659 treino / 220 validação / 220 teste.

---

## Estrutura do Repositório

```
AnaliseDataset/
├── README.md
├── requirements.txt
└── data/
    ├── Sprint_01/
    │   ├── steam_top_games_2026.csv        ← Dataset bruto original
    │   └── Projeto_Grupo_05_Sprint_01.ipynb
    ├── Sprint_02/
    │   ├── Projeto_Grupo_05_Sprint_02.ipynb
    │   └── data/Sprint_02/
    │       ├── steam_train_60.csv           ← Split de treino (60%)
    │       ├── steam_val_20.csv             ← Split de validação (20%)
    │       └── steam_test_20.csv            ← Split de teste blindado (20%)
    ├── Sprint_03/
    │   └── Projeto_Grupo_05_Sprint_03.ipynb
    └── Sprint_04/
        ├── Projeto_Grupo_05_Sprint_04.ipynb
        └── modelo_steam_log_v1.pkl          ← Gerado ao executar o notebook
```

---

## Resumo por Sprint (Ciclo CRISP-DM)

### Sprint 1 — Entendimento do Negócio e EDA
- Escolha do dataset e definição do problema de negócio
- Análise exploratória univariada e bivariada
- Diagnóstico de valores nulos (`metacritic_score`: 57% ausente)
- Validação de **4 hipóteses** sobre o que influencia o preço:
  - H1: Gênero (RPG/Simulation > Indie/Action) ✅
  - H2: Nota Metacritic (correlação fraca, relação não-linear) ⚠️
  - H3: Inflação — ano de lançamento influencia preço ✅
  - H4: Publisher-Backed cobra mais que Self-Published ✅

### Sprint 2 — Pré-Processamento e Feature Engineering
- Remoção de outliers extremos e jogos gratuitos
- Engenharia de features: `genre_count`, `approval_rating`, `publishing_model`, `release_year`
- Estratégia de imputação (mediana para contínuas, moda para discreta/categórica)
- Split dos dados com seed fixo (reprodutibilidade)
- Pipeline de pré-processamento encapsulado (imputação e escalonamento ajustados apenas no treino, sem vazamento de pré-processamento)

### Sprint 3 — Modelagem e Otimização
- Avaliação de 3 algoritmos via Validação Cruzada 5-Fold:
  - Regressão Linear (baseline)
  - **Random Forest** ← selecionado pela menor variância entre folds
  - XGBoost (demonstração com Early Stopping)
- Transformação logarítmica do target (`log1p`) para mitigar heterocedasticidade
- GridSearchCV: 36 combinações × 5 folds = 180 fits

### Sprint 4 — Interpretação, Análise Final e Entrega
- Análise aprofundada dos erros (4 visualizações + recuperação dos nomes dos jogos das piores predições)
- Feature Importance do modelo campeão
- Revisão crítica das hipóteses da Sprint 1
- Tabela consolidada de desempenho (todos os modelos)
- Limitações honestas e próximos passos documentados

---

## Arquitetura de Validação

> Esclarecimento importante (incorpora o feedback da Sprint 2):

O **modelo campeão** é treinado e otimizado sobre **80% dos dados** (Treino 60% + Validação 20%, unificados em uma base de desenvolvimento) usando **Validação Cruzada K-Fold (5 folds)** dentro do `GridSearchCV`. Os **20% de teste permanecem totalmente isolados** e só são usados na avaliação final.

O split de três vias 60/20/20, com conjunto de validação explícito, foi usado **apenas** na demonstração de *Early Stopping* do XGBoost — **não** é a estratégia de avaliação do campeão.

---

## Modelo Final

| Parâmetro | Valor |
|:---|:---|
| Algoritmo | Random Forest Regressor |
| Transformação do Target | `np.log1p` / `np.expm1` |
| Otimização | GridSearchCV (5-Fold) |
| Hiperparâmetros | `max_depth=10`, `n_estimators=200`, `min_samples_leaf=1`, `min_samples_split=2` |
| Features | `metacritic_score`, `genre_count`, `approval_rating`, `release_year`, `publishing_model` |
| Arquivo exportado | `modelo_steam_log_v1.pkl` |

**Por que Random Forest e não XGBoost?**  
O Random Forest apresentou menor desvio padrão entre os 5 folds do Cross-Validation (maior estabilidade), o que é preferível a um RMSE médio ligeiramente menor com maior variância. Modelos estáveis generalizam melhor em produção.

---

## Resultados Finais (avaliação honesta)

| Métrica | Modelo Campeão | Baseline (`DummyRegressor`, média) |
|:---|:---:|:---:|
| MAE (teste) | **USD 9,77** | USD 10,22 |
| RMSE (teste) | USD 14,21 | **USD 13,34** |
| R² (teste) | **−0,10** | ~0,00 |
| RMSE (CV) | USD 13,89 | USD 13,34 |

**Leitura dos resultados:**

- **Sem overfitting:** o RMSE de validação cruzada (13,89) e o de teste (14,21) são praticamente iguais — delta de apenas USD 0,32. O modelo generaliza; o problema é de **viés/sub-ajuste**, não de variância.
- **Vence o baseline em MAE (≈4,4%), mas perde em RMSE.** O modelo acerta melhor a faixa de preço comum, porém o R² de teste é levemente negativo: ele **não supera a média em variância explicada**. O RMSE pune quadraticamente os erros grandes nos jogos AAA, que o modelo não consegue precificar.
- **A descoberta principal não é o score, e sim o entendimento do problema:** o preço de um jogo depende fortemente de fatores que não estão no dataset (orçamento de produção, força da IP, marketing). O modelo funciona como **termômetro de mercado** na faixa USD 5–30, não como preditor exato.

**Importância das features:** `approval_rating` (~60%) ≫ `metacritic_score` (~18%) > `genre_count` (~12%) > `release_year` (~5%) ≈ `publishing_model` (~5%).

**Padrão de erro (regressão à média):** o modelo superestima jogos baratos e subestima os caros. As 10 piores predições são todas títulos AAA — por exemplo *Forspoken* (US$ 69,99, previsto US$ 4,87), *The Crew Motorfest* (69,99 → 10,63), *Indiana Jones and the Great Circle* (69,99 → 18,27) e *OCTOPATH TRAVELER II* (59,99 → 9,27).

---

## Limitações Conhecidas

1. **Não supera o baseline em variância explicada (R² negativo).** Em RMSE, prever a média erra menos. O modelo é útil na faixa comum, mas não "vence" estatisticamente.
2. **Vazamento temporal da feature dominante (`approval_rating`).** Ela é derivada das avaliações de usuários, que **não existem no dia do lançamento**. O pipeline elimina o vazamento de *pré-processamento*, mas essa feature configura um *data leakage* temporal: o modelo serve para análise retrospectiva de catálogo, e precisaria ser re-treinado **sem** `approval_rating` para uso preditivo real em lançamentos.
3. **Não distingue Indie de AAA.** Falta uma variável de orçamento de produção — o modelo subestima sistematicamente jogos acima de USD 40.
4. **`genre_count` perde granularidade.** Usar a quantidade de gêneros como proxy trata RPG e Action da mesma forma.
5. **`metacritic_score` ausente em ~57% dos jogos.** A imputação pela mediana reduz o sinal dessa feature.

---

## Próximos Passos

1. **Re-treinar sem `approval_rating`** — medir o desempenho usando apenas features disponíveis no lançamento (o modelo "honesto" para precificação preditiva).
2. **Modelagem em dois estágios** — classificar Indie vs. AAA antes do regressor de preço.
3. **Feature engineering** — enriquecer com tamanho em GB (proxy de orçamento), suporte a multiplayer e número da franquia.
4. **Stacking** — combinar Random Forest com Regressão Linear para reduzir o erro residual mantendo interpretabilidade.

---

## Como Executar

### Google Colab (recomendado)
1. Abra qualquer `.ipynb` no Colab
2. Clique em **Ambiente de execução → Executar tudo** (`Ctrl+F9`)
3. Os dados são carregados automaticamente do GitHub — sem uploads necessários

### Local (VS Code / Jupyter)
```bash
pip install pandas matplotlib seaborn numpy scikit-learn xgboost joblib
jupyter notebook
```

---

## Reprodutibilidade

- `random_state=42` fixo em todos os modelos
- `KFold(n_splits=5, shuffle=True, random_state=42)` para validação cruzada
- Split dos dados gerado uma única vez na Sprint 2 e versionado no repositório
- Pipeline encapsula o pré-processamento — elimina o vazamento de *pré-processamento* (o vazamento temporal do `approval_rating` está documentado em Limitações Conhecidas)