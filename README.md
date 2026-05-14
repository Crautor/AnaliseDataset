# steam-price-prediction

Este repositório contém o projeto de Inteligência Artificial e Ciência de Dados desenvolvido pelo Grupo 05. O objetivo central é analisar o ecossistema de jogos da plataforma Steam e construir um modelo preditivo capaz de estimar o preço de prateleira de um jogo com base em suas características técnicas e de mercado.

## Equipa (Grupo 05)
* João Vitor Candido
* João Vitor Dysarz
* João Vitor Haas
* Nathan Daniel Breier
* Vitor Hugo Dallanol

---

## Definição do Problema
Pergunta de Negócio: Qual o preço justo de um jogo baseado nas suas características (no lançamento)?

No mercado de desenvolvimento de software, precificar um produto digital é um desafio. O nosso modelo visa ajudar estúdios independentes e publicadoras a encontrarem o ponto ótimo de preço, evitando barreiras de entrada financeiras (preço muito alto) ou desvalorização do produto (preço muito baixo).

Tipo de Problema de Machine Learning:
* Regressão Linear (Supervisionado)

---

## Base de Dados (Dataset)
* Nome: Steam Dataset for Recommendation & Prediction
* Fonte: [Kaggle](https://www.kaggle.com/datasets/patelris/steam-top-1495-games-dataset)
* Variável Alvo (Target): price_usd (Preço Base em Dólares)

---

## Estrutura do Repositório
O projeto está dividido em Sprints quinzenais, seguindo a metodologia ágil e o ciclo de vida CRISP-DM. Os dados e notebooks de cada etapa ficam armazenados juntos em suas respetivas pastas.

```text
steam-price-prediction
|-- data
|   |-- Sprint_01
|   |   |-- steam_top_games_2026.csv
|   |   |-- Projeto_Grupo_05_Sprint_01.ipynb
|   |-- sprint_02
|   |-- sprint_03
|   |-- sprint_04
|-- README.md
```

---

## Instruções e Validação da Sprint 1

Esta secção detalha os requisitos da Sprint 1 (30/04 a 14/05) e como o nosso projeto atende a cada um deles, garantindo a qualidade técnica e a reprodutibilidade da entrega.

### Objetivos Atingidos
* Escolha de Dataset e Problema de Negócio: Selecionámos o dataset do ecossistema Steam (Steam Dataset for Recommendation & Prediction) para resolver um problema real de precificação.
* Análise Exploratória de Dados (EDA) Completa: O projeto contém a limpeza dos dados, tratamento de valores nulos (Missing Values), e a validação estatística e visual de 4 hipóteses de negócio detalhadas.

---

## Como Executar o Projeto (Reprodutibilidade)

Este Jupyter Notebook foi desenvolvido e testado para ser executado nativamente no Google Colab.

### Execução via Google Colab
Para garantir a reprodutibilidade técnica exigida, os dados são importados automaticamente do nosso repositório oficial no GitHub. Não é necessário realizar o upload manual de nenhum ficheiro CSV.

Dataset Link (Raw): https://raw.githubusercontent.com/JoaoDysarz/steam-price-prediction/main/data/Sprint_01/steam_top_games_2026.csv

Passos:
1. Abra o ficheiro .ipynb no ambiente Google Colab.
2. Certifique-se de que a variável url_github no Passo 1 (Carregamento de Dados) aponta para o link acima.
3. No menu superior do Colab, clique em Ambiente de execução > Executar tudo (ou utilize o atalho Ctrl + F9).

### Execução Local (VS Code / Jupyter)
Caso opte por executar o projeto fora do ambiente Colab, será necessário instalar as dependências de Python utilizadas no projeto.

Requisitos:
* Python 3.x
* Bibliotecas: pandas, matplotlib, seaborn, numpy

Comando para instalação:
```bash
pip install pandas matplotlib seaborn numpy
```