# Mercado Livre — Data Science & Machine Learning

> Análise de vendas, engenharia de atributos e classificação de produtos por potencial de alta demanda a partir de dados reais de uma operação do Mercado Livre.

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python\&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas\&logoColor=white)](https://pandas.pydata.org/)
[![Scikit--learn](https://img.shields.io/badge/scikit--learn-Machine%20Learning-F7931E?logo=scikit-learn\&logoColor=white)](https://scikit-learn.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter\&logoColor=white)](https://jupyter.org/)
[![Status](https://img.shields.io/badge/Status-Em%20desenvolvimento-yellow)](#próximos-passos)

---

## Sobre o projeto

Este projeto aplica conceitos de Data Science e Machine Learning a um problema real de negócio: analisar o histórico de vendas de uma loja que comercializa produtos e acessórios para motocicletas no Mercado Livre e investigar quais características estão associadas a itens de maior demanda.

O projeto parte de relatórios de vendas exportados da plataforma e percorre um fluxo completo de análise:

```text
Dados brutos
    ↓
Limpeza e tratamento
    ↓
Definição do problema
    ↓
Agregação por produto
    ↓
Engenharia de features
    ↓
Treinamento
    ↓
Avaliação
    ↓
Investigação de Data Leakage
    ↓
Ranking final
```

O objetivo não é apenas treinar um modelo, mas entender os dados, questionar resultados suspeitos e construir uma avaliação mais realista.

---

## Problema de negócio

A operação possui diversos anúncios de produtos, mas nem todos apresentam o mesmo volume de vendas.

A principal pergunta explorada neste projeto é:

> É possível identificar, a partir de características disponíveis do produto, quais itens apresentam maior potencial de demanda?

Para transformar essa pergunta em um problema supervisionado de classificação, foi criado o alvo `alta_demanda`.

A classificação foi definida da seguinte forma:

* `alta_demanda = 1`: produto classificado como de alta demanda;
* `alta_demanda = 0`: produto classificado como de baixa demanda.

O corte utilizado foi a mediana de unidades vendidas por item, escolhida por ser menos sensível a produtos com volumes muito acima dos demais.

---

## Dados utilizados

Foram combinados dois relatórios de vendas exportados do Mercado Livre:

* Relatório de 12/09;
* Relatório de 15/09.

Após tratamento e remoção de duplicidades:

| Métrica                                | Resultado |
| -------------------------------------- | --------: |
| Vendas únicas                          |       182 |
| Produtos/SKUs distintos                |        76 |
| Itens classificados como baixa demanda |        48 |
| Itens classificados como alta demanda  |        28 |
| Mediana de unidades vendidas           | 1 unidade |

### Privacidade dos dados

Os arquivos CSV originais não são versionados no Git, pois contêm dados reais da operação.

O `.gitignore` impede que os arquivos de dados sejam enviados ao repositório:

```gitignore
data/raw/*.csv
data/processed/*.csv
```

Dessa forma, o projeto mantém o código, a metodologia e os resultados da análise sem expor informações comerciais da operação.

---

## Estrutura do projeto

```text
mercado-livre-data-science/
│
├── data/
│   ├── raw/                 # Dados brutos exportados do Mercado Livre
│   └── processed/           # Dados tratados e processados
│
├── images/                  # Visualizações geradas durante a análise
│
├── models/                  # Espaço reservado para modelos treinados
│
├── notebooks/
│   └── analise_vendas_mercado_livre.ipynb
│
├── src/                     # Espaço reservado para código reutilizável
│
├── .gitignore
└── README.md
```

---

## Pipeline de Data Science

### 1. Tratamento dos dados

Os relatórios são carregados e preparados para análise.

Entre as etapas realizadas estão:

* padronização das datas;
* tratamento de valores nulos;
* conversão das datas para o formato adequado;
* identificação de vendas efetivadas;
* combinação dos relatórios;
* remoção de duplicidades;
* organização dos dados por produto.

As datas originalmente apresentadas em português são convertidas para um formato adequado para análise utilizando Pandas.

---

### 2. Definição da variável-alvo

No nível de venda, foi criada a variável:

```python
venda_efetivada
```

Essa variável representa se uma determinada venda foi efetivamente concluída.

Posteriormente, como o objetivo passou a ser analisar os produtos, os dados foram agregados por:

```text
SKU + Título do anúncio
```

A partir dessa agregação foi criada a variável:

```python
alta_demanda
```

A classificação utiliza o volume de unidades vendidas como referência.

---

### 3. Agregação por produto

Para cada produto foram calculadas métricas como:

| Feature                   | Descrição                               |
| ------------------------- | --------------------------------------- |
| `total_vendas`            | Quantidade de vendas registradas        |
| `total_unidades_vendidas` | Total de unidades vendidas              |
| `vendas_efetivadas`       | Quantidade de vendas concluídas         |
| `preco_medio`             | Preço médio do anúncio                  |
| `receita_total`           | Receita acumulada                       |
| `primeira_venda`          | Data da primeira venda                  |
| `ultima_venda`            | Data da última venda                    |
| `tempo_ativo_dias`        | Intervalo entre primeira e última venda |
| `ticket_medio`            | Receita média por unidade               |

Também foi recuperado o tipo de anúncio predominante de cada SKU.

---

## Engenharia de Features

Foram criadas variáveis derivadas para representar melhor o comportamento comercial dos produtos.

### Tempo ativo

```text
tempo_ativo_dias =
última venda - primeira venda
```

Essa variável representa o intervalo observado entre a primeira e a última venda de determinado produto.

### Ticket médio

```text
ticket_medio =
receita_total / total_unidades_vendidas
```

Representa o valor médio por unidade vendida.

### Tipo de anúncio

A variável categórica `Tipo de anúncio` foi transformada utilizando One-Hot Encoding, permitindo que os modelos do Scikit-learn utilizem essa informação.

---

## Modelos de Machine Learning

Foram comparados dois algoritmos de classificação:

### Decision Tree

Uma Árvore de Decisão foi utilizada como modelo mais simples e interpretável.

Configuração utilizada:

```python
DecisionTreeClassifier(
    max_depth=5,
    random_state=42
)
```

### Random Forest

Também foi utilizada uma Random Forest composta por 200 árvores:

```python
RandomForestClassifier(
    n_estimators=200,
    max_depth=5,
    random_state=42
)
```

A Random Forest foi utilizada para investigar se um conjunto de árvores poderia apresentar comportamento mais robusto do que uma única árvore de decisão.

---

## Avaliação dos modelos

A base foi dividida em:

* 80% para treinamento;
* 20% para teste.

Configuração utilizada:

```python
test_size=0.2
random_state=42
stratify=y
```

Foram analisadas diferentes métricas:

* Accuracy;
* Precision;
* Recall;
* F1-score;
* Matriz de confusão;
* Importância das features.

A utilização de múltiplas métricas é importante porque a distribuição entre as classes não é perfeitamente equilibrada.

---

# Data Leakage

Um dos principais aprendizados deste projeto surgiu quando os modelos apresentaram um desempenho aparentemente perfeito.

Com todas as features disponíveis, os dois modelos chegaram a:

```text
100% de acurácia no conjunto de teste
```

Em vez de considerar esse resultado automaticamente como sucesso, ele foi investigado.

### Por que o resultado era suspeito?

A variável `alta_demanda` foi criada diretamente a partir do volume de unidades vendidas.

Ao mesmo tempo, algumas das features utilizadas pelo modelo também carregavam informações diretamente relacionadas ao volume de vendas.

Um exemplo é:

```text
total_vendas
```

Essa variável possui uma relação muito forte com a própria definição do alvo.

Outro exemplo é:

```text
tempo_ativo_dias
```

que pode carregar informação indireta sobre a quantidade de oportunidades que um produto teve para realizar vendas.

Isso caracteriza um risco de Data Leakage.

Em outras palavras, o modelo pode estar recebendo informações que representam, direta ou indiretamente, a própria resposta que deveria aprender a prever.

---

## Teste progressivo de Data Leakage

Foram realizados três experimentos.

| Cenário                                       | Decision Tree | Random Forest |
| --------------------------------------------- | ------------: | ------------: |
| Todas as features                             |          100% |          100% |
| Removendo `total_vendas`                      |          100% |           94% |
| Removendo `total_vendas` + `tempo_ativo_dias` |           81% |           88% |

### Interpretação

A última configuração é a mais relevante para uma avaliação mais realista, pois remove as duas variáveis mais diretamente relacionadas ao histórico de volume de vendas.

Nesse cenário, permaneceram principalmente features como:

```text
preco_medio
receita_total
ticket_medio
tipo_anuncio
```

A Random Forest apresentou:

```text
Accuracy: 88%
```

Enquanto a Decision Tree apresentou:

```text
Accuracy: 81%
```

Esses resultados devem ser interpretados com cautela.

A base possui apenas 76 produtos e o conjunto de teste contém 16 itens. Portanto, uma acurácia de 88% não deve ser interpretada como uma estimativa definitiva do desempenho do modelo em novos períodos ou em uma base maior.

---

## Ranking final

Após a investigação de Data Leakage, foi utilizada a Random Forest sem as features:

```text
total_vendas
tempo_ativo_dias
```

O modelo gera a probabilidade prevista para a classe `alta_demanda`:

```python
probabilidade_alta_demanda_v3
```

Essa probabilidade permite ordenar os produtos de acordo com a previsão do modelo.

O ranking pode ser utilizado como ferramenta analítica para investigar produtos com maior probabilidade prevista de pertencer à classe de alta demanda.

### Possíveis aplicações

Esse tipo de ranking pode apoiar análises relacionadas a:

* priorização de produtos para divulgação;
* planejamento de estoque;
* identificação de produtos com maior potencial;
* investigação de características associadas à demanda;
* apoio à tomada de decisão comercial.

O ranking não representa uma garantia de vendas futuras.

---

## Visualizações

O projeto também possui visualizações para facilitar a interpretação dos dados e dos modelos.

### Distribuição de unidades vendidas

![Distribuição de unidades vendidas](images/distribuicao_unidades_vendidas%20-%20dados_12_e_15_setembro.png)

A visualização apresenta a distribuição do volume vendido por produto e o corte utilizado para definição da classe de demanda.

### Matriz de confusão

![Matriz de confusão](images/matriz_de_confusao%20-%20dados_12_e_15_setembro.png)

A matriz de confusão permite analisar os acertos e erros de classificação dos modelos.

Também existem versões das visualizações considerando somente os dados do relatório de 12/09.

---

## Tecnologias utilizadas

| Tecnologia       | Utilização                     |
| ---------------- | ------------------------------ |
| Python           | Linguagem principal            |
| Pandas           | Manipulação e análise de dados |
| NumPy            | Operações numéricas            |
| Matplotlib       | Visualização de dados          |
| Seaborn          | Visualização estatística       |
| Scikit-learn     | Machine Learning               |
| Jupyter Notebook | Desenvolvimento e documentação |

---

## Como executar

### 1. Clone o repositório

```bash
git clone <URL_DO_REPOSITORIO>
cd mercado-livre-data-science
```

### 2. Crie um ambiente virtual

No Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

No Linux/macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Instale as dependências

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

### 4. Adicione os dados

Coloque os arquivos CSV exportados do Mercado Livre dentro de:

```text
data/raw/
```

Os arquivos reais não são disponibilizados no repositório por conterem dados comerciais da operação.

### 5. Execute o notebook

```bash
jupyter notebook
```

Depois abra:

```text
notebooks/analise_vendas_mercado_livre.ipynb
```

Execute as células na ordem apresentada no notebook.

---

## O que este projeto demonstra

Este projeto demonstra diversas etapas de um workflow de Data Science aplicado a um problema real:

* Importação e tratamento de dados reais;
* Padronização de dados;
* Análise exploratória;
* Definição de variável-alvo;
* Agregação por SKU;
* Engenharia de features;
* Encoding de variáveis categóricas;
* Separação entre treino e teste;
* Treinamento de modelos de classificação;
* Avaliação utilizando múltiplas métricas;
* Construção de matrizes de confusão;
* Análise de importância das variáveis;
* Investigação de Data Leakage;
* Comparação entre diferentes cenários;
* Geração de ranking de produtos.

---

## Próximos passos

### Dados

* [ ] Incorporar mais períodos de vendas;
* [ ] Aumentar o número de produtos observados;
* [ ] Adicionar dados de estoque;
* [ ] Incorporar custos e margem de lucro;
* [ ] Adicionar informações de categoria;
* [ ] Adicionar métricas relacionadas aos anúncios;
* [ ] Incorporar informações de visualizações e conversão.

### Machine Learning

* [ ] Testar Logistic Regression;
* [ ] Testar Gradient Boosting;
* [ ] Testar XGBoost;
* [ ] Implementar validação cruzada;
* [ ] Realizar ajuste de hiperparâmetros;
* [ ] Avaliar métricas adicionais;
* [ ] Utilizar uma divisão temporal para validação;
* [ ] Avaliar o modelo em períodos futuros não utilizados no treinamento.

### Produto

* [ ] Criar pipeline automatizado de processamento;
* [ ] Salvar modelos treinados;
* [ ] Criar uma API de previsão;
* [ ] Criar um dashboard;
* [ ] Automatizar a atualização do ranking;
* [ ] Transformar a análise em uma ferramenta de apoio à decisão.

---

## Organização do notebook

O notebook principal segue uma sequência estruturada:

```text
01. Importação das bibliotecas
02. Carregamento dos dados
03. Exploração inicial
04. Tratamento dos dados
05. Definição da variável-alvo
06. Agregação por item
07. Definição de alta demanda
08. Engenharia de features
09. Preparação para modelagem
10. Separação entre treino e teste
11. Treinamento dos modelos
12. Avaliação
13. Ranking inicial
14. Conclusão
15. Investigação de Data Leakage
16. Ranking final sem Data Leakage
```

---

## Conclusão

O projeto mostra um processo de análise que vai além de simplesmente treinar um modelo e observar sua acurácia.

O primeiro experimento produziu 100% de acurácia, mas a investigação mostrou que parte desse desempenho estava relacionada a variáveis muito próximas da definição do próprio alvo.

Ao remover essas informações e repetir o experimento, foram obtidos resultados mais realistas:

```text
Random Forest: 88%
Decision Tree: 81%
```

Esse processo demonstra uma etapa fundamental de Machine Learning: entender de onde a performance está vindo e verificar se o modelo realmente está aprendendo padrões úteis, em vez de receber indiretamente a resposta.

A investigação de Data Leakage foi, portanto, parte essencial do projeto e não apenas uma etapa adicional de modelagem.

---

## Autor

**ZuziR6**

Projeto desenvolvido como estudo prático de Data Science, Machine Learning, análise de dados e modelagem preditiva, utilizando um problema real de negócio.

---

Se este projeto foi útil ou interessante, considere deixar uma estrela no repositório.
