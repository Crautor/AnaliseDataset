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
- **Escopo:** Apenas jogos pagos (1.113 após remoção dos Free-to-Play)

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
- Split estratificado 60/20/20 com seed fixo (reprodutibilidade)
- Pipeline de pré-processamento encapsulado (sem data leakage)

### Sprint 3 — Modelagem e Otimização
- Avaliação de 3 algoritmos via Validação Cruzada 5-Fold:
  - Regressão Linear (baseline)
  - **Random Forest** ← selecionado pela menor variância entre folds
  - XGBoost (demonstração com Early Stopping)
- Transformação logarítmica do target (`log1p`) para mitigar heterocedasticidade
- GridSearchCV: 36 combinações × 5 folds = 180 fits

### Sprint 4 — Interpretação, Análise Final e Entrega *(este notebook)*
- Análise aprofundada dos erros (4 visualizações)
- Feature Importance do modelo campeão
- Revisão crítica das hipóteses da Sprint 1
- Tabela consolidada de desempenho (todos os modelos)
- Limitações honestas e próximos passos documentados

---

## Modelo Final

| Parâmetro | Valor |
|:---|:---|
| Algoritmo | Random Forest Regressor |
| Transformação do Target | `np.log1p` / `np.expm1` |
| Otimização | GridSearchCV (5-Fold) |
| Features | `metacritic_score`, `genre_count`, `approval_rating`, `release_year`, `publishing_model` |
| Arquivo exportado | `modelo_steam_log_v1.pkl` |

**Por que Random Forest e não XGBoost?**  
O Random Forest apresentou menor desvio padrão entre os 5 folds do Cross-Validation (maior estabilidade), o que é preferível a um RMSE médio ligeiramente menor com maior variância. Modelos estáveis generalizam melhor em produção.

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
- Pipeline encapsula o pré-processamento — elimina risco de data leakage
