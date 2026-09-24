 Mercado Livre — Data Science & Machine Learning

Análise de vendas, engenharia de atributos e classificação de produtos por potencial de alta demanda a partir de dados reais de uma operação do Mercado Livre.







 Sobre o projeto

Este projeto aplica conceitos de Data Science e Machine Learning a um problema real de negócio: analisar o histórico de vendas de uma loja que comercializa produtos e acessórios para motocicletas no Mercado Livre e investigar quais características estão associadas a itens de maior demanda.

O projeto parte de relatórios de vendas exportados da plataforma e percorre um fluxo completo de análise:

dados brutos → limpeza → definição do problema → agregação por produto → engenharia de features → treinamento → avaliação → investigação de vazamento → ranking final

O objetivo não é apenas treinar um modelo, mas entender o dado, questionar resultados suspeitos e construir uma avaliação mais realista.

 Principal aprendizado do projeto: um modelo pode apresentar uma métrica aparentemente excelente e ainda assim estar aprendendo informações que já estão implicitamente presentes no alvo. Por isso, a investigação de data leakage faz parte central deste projeto.

 Problema de negócio

A operação possui diversos anúncios de produtos, mas nem todos apresentam o mesmo volume de vendas.

A pergunta explorada é:

É possível identificar, a partir de características disponíveis do produto, quais itens apresentam maior potencial de demanda?

Para transformar essa pergunta em um problema supervisionado de classificação, foi criado o alvo:

alta_demanda = 1 → item acima do corte definido para alta demanda;

alta_demanda = 0 → item abaixo ou igual ao corte.

O corte utilizado foi a mediana de unidades vendidas por item, escolhida por ser menos sensível a produtos com volumes muito acima dos demais.

 Dados utilizados

Foram combinados dois relatórios de vendas exportados do Mercado Livre:

relatório de 12/09;

relatório de 15/09.

Após tratamento e remoção de duplicidades:

Métrica

Resultado

Vendas únicas

182

Produtos/SKUs distintos

76

Itens classificados como baixa demanda

48

Itens classificados como alta demanda

28

Mediana de unidades vendidas

1 unidade

 Privacidade

Os arquivos CSV originais não são versionados no Git, pois contêm dados reais da operação.

Eles são ignorados pelo .gitignore através de:

data/raw/*.csv
data/processed/*.csv

Assim, o repositório mantém o código e a metodologia sem expor os dados comerciais utilizados na análise.

 Estrutura do projeto

mercado-livre-data-science/
│
├── data/
│   ├── raw/                 # Dados brutos exportados do Mercado Livre
│   └── processed/           # Dados tratados/processados
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

 Pipeline de Data Science

1.  Tratamento dos dados

Os relatórios são carregados e preparados para análise, incluindo:

padronização das datas;

tratamento de valores nulos;

conversão de datas no formato utilizado no relatório;

identificação de vendas efetivadas;

combinação dos relatórios;

remoção de duplicidades.

As datas originalmente apresentadas em português são convertidas para o formato adequado ao pandas.

2.  Definição da variável-alvo

No nível de venda, foi criada a variável:

venda_efetivada

Ela representa se a venda foi efetivamente concluída.

Posteriormente, como o objetivo passou a ser analisar produtos, os dados foram agregados por:

SKU + Título do anúncio

E foi criado o alvo:

alta_demanda

baseado no volume de unidades vendidas.

3.  Agregação por produto

Para cada item foram calculadas métricas como:

Feature

Descrição

total_vendas

Quantidade de vendas registradas

total_unidades_vendidas

Total de unidades vendidas

vendas_efetivadas

Quantidade de vendas concluídas

preco_medio

Preço médio do anúncio

receita_total

Receita acumulada

primeira_venda

Data da primeira venda

ultima_venda

Data da última venda

tempo_ativo_dias

Intervalo entre primeira e última venda

ticket_medio

Receita média por unidade

Também foi recuperado o tipo de anúncio predominante de cada SKU.

 Engenharia de Features

Foram criadas variáveis derivadas para representar melhor o comportamento comercial dos produtos.

 Tempo ativo

tempo_ativo_dias =
última venda - primeira venda

Essa variável ajuda a representar há quanto tempo o item aparece no histórico de vendas.

 Ticket médio

ticket_medio =
receita_total / total_unidades_vendidas

Representa o valor médio efetivamente recebido por unidade.

 Tipo de anúncio

A variável categórica Tipo de anúncio foi transformada utilizando One-Hot Encoding, permitindo que os modelos do scikit-learn trabalhem com a informação.

 Modelos

Foram comparados dois algoritmos de classificação:

Decision Tree

Uma Árvore de Decisão, configurada com:

DecisionTreeClassifier(
    max_depth=5,
    random_state=42
)

A árvore foi utilizada como modelo mais simples e interpretável.

Random Forest

Um ensemble composto por 200 árvores:

RandomForestClassifier(
    n_estimators=200,
    max_depth=5,
    random_state=42
)

A Random Forest foi utilizada para investigar se um conjunto de árvores poderia apresentar comportamento mais robusto.

 Avaliação

A base foi dividida em:

80% treinamento

20% teste

com:

test_size=0.2
random_state=42
stratify=y

Foram analisadas:

Accuracy;

Precision;

Recall;

F1-score;

Matriz de confusão;

Importância das features.

A escolha de múltiplas métricas é importante porque a distribuição entre as classes não é perfeitamente equilibrada.

 Data Leakage: o principal aprendizado

Um dos resultados mais importantes do projeto surgiu justamente quando o modelo apresentou um desempenho bom demais para ser aceito sem investigação.

Com todas as features, os dois modelos chegaram a:

100% de acurácia no conjunto de teste.

Em vez de considerar isso automaticamente como sucesso, o resultado foi investigado.

Por que o resultado era suspeito?

O alvo alta_demanda é criado diretamente a partir de:

total_unidades_vendidas

e, nesse conjunto de dados, total_vendas possui forte relação com essa informação.

Além disso, tempo_ativo_dias também pode carregar informação indireta relacionada à repetição de vendas.

Ou seja: algumas features estavam muito próximas da própria informação utilizada para definir o alvo.

 Teste progressivo de vazamento

Foram realizadas três versões do experimento.

Cenário

Decision Tree

Random Forest

Todas as features

100%

100%

Removendo total_vendas

100%

94%

Removendo total_vendas + tempo_ativo_dias

81%

88%

 Interpretação

A última configuração é a mais relevante para avaliar a capacidade preditiva com menor risco de vazamento, pois remove as duas variáveis mais diretamente relacionadas ao histórico de volume de vendas.

Nesse cenário, permaneceram principalmente:

preco_medio
receita_total
ticket_medio
tipo_anuncio

A Random Forest atingiu 88% de acurácia, enquanto a Árvore de Decisão atingiu 81%.

 Esses resultados devem ser interpretados com cautela: a base possui apenas 76 produtos e o conjunto de teste contém 16 itens. Portanto, 88% não deve ser tratado como uma estimativa definitiva de desempenho em novos períodos ou produtos.

 Ranking final

Após o teste de vazamento, o ranking final utiliza a:

Random Forest

treinada sem:

total_vendas
tempo_ativo_dias

O modelo gera:

probabilidade_alta_demanda_v3

que representa a probabilidade prevista pelo classificador para a classe alta_demanda.

Isso permite ordenar os produtos e investigar quais itens apresentam maior probabilidade prevista de pertencer à classe de alta demanda.

Aplicação potencial

Esse ranking pode apoiar decisões como:

priorização de produtos para divulgação;

análise de estoque;

identificação de produtos com maior potencial;

investigação de características associadas à demanda;

apoio à tomada de decisão comercial.

O ranking é uma ferramenta analítica, não uma garantia de vendas futuras.

 Visualizações

O projeto também gera visualizações para facilitar a interpretação dos resultados.

Distribuição de unidades vendidas



A visualização mostra a distribuição do volume vendido por item e o corte utilizado para definir a classe de demanda.

Matriz de confusão



A matriz de confusão permite observar como os modelos classificaram os itens de baixa e alta demanda.

Também estão disponíveis versões das visualizações considerando somente os dados do relatório de 12/09.

 Tecnologias

Tecnologia

Uso

 Python

Linguagem principal

 Pandas

Manipulação e análise de dados

 NumPy

Operações numéricas

 Matplotlib

Visualização

 Seaborn

Visualização estatística

 Scikit-learn

Machine Learning

 Jupyter Notebook

Desenvolvimento e documentação da análise

 Como executar

1. Clone o repositório

git clone <URL_DO_REPOSITORIO>
cd mercado-livre-data-science

2. Crie um ambiente virtual

Windows:

python -m venv .venv
.venv\Scripts\activate

Linux/macOS:

python3 -m venv .venv
source .venv/bin/activate

3. Instale as dependências

pip install pandas numpy matplotlib seaborn scikit-learn jupyter

4. Adicione os dados

Coloque os arquivos CSV exportados do Mercado Livre em:

data/raw/

Os arquivos reais não são disponibilizados no repositório por conterem dados comerciais da operação.

5. Execute o notebook

jupyter notebook

Depois abra:

notebooks/analise_vendas_mercado_livre.ipynb

e execute as células em ordem.

 O que este projeto demonstra

Mais do que simplesmente aplicar um algoritmo de Machine Learning, este projeto demonstra etapas importantes de um workflow real de Data Science:

Importação e tratamento de dados reais

Padronização de dados

Análise exploratória

Definição de variável-alvo

Agregação por SKU

Engenharia de features

Encoding de variáveis categóricas

Separação entre treino e teste

Treinamento de modelos de classificação

Avaliação com múltiplas métricas

Matrizes de confusão

Análise de importância das variáveis

Investigação de data leakage

Comparação entre cenários

Geração de ranking de produtos

 Próximos passos

O projeto foi estruturado para evoluir além do primeiro experimento.

Dados

Incorporar mais períodos de vendas

Aumentar o número de produtos observados

Adicionar dados de estoque

Incorporar custos e margem de lucro

Incorporar visualizações e métricas dos anúncios

Adicionar informações de categoria e produto

Machine Learning

Testar Logistic Regression

Testar Gradient Boosting / XGBoost

Fazer validação cruzada

Ajustar hiperparâmetros

Avaliar métricas além de accuracy

Separar melhor o período histórico do período futuro para validação temporal

Produto

Criar pipeline de processamento automatizado

Salvar modelos treinados

Criar uma API de previsão

Criar dashboard de acompanhamento

Automatizar a atualização do ranking

Transformar a análise em uma ferramenta de apoio à decisão

 Organização do notebook

O notebook principal segue uma sequência didática:

01. Importação das bibliotecas
02. Carregamento dos dados
03. Exploração inicial
04. Tratamento dos dados
05. Definição da variável-alvo
06. Agregação por item
07. Definição de alta demanda
08. Engenharia de features
09. Preparação para modelagem
10. Separação treino/teste
11. Treinamento dos modelos
12. Avaliação
13. Ranking inicial
14. Conclusão
15. Investigação de data leakage
16. Ranking final sem vazamento

 Conclusão

O projeto mostra um processo de análise que vai além de "treinar um modelo e olhar a acurácia".

O primeiro experimento produziu 100% de acurácia, mas a investigação revelou que parte desse desempenho estava relacionada a variáveis muito próximas da definição do próprio alvo.

Ao remover essas informações e repetir o experimento, a performance caiu para níveis mais realistas:

Random Forest: 88%
Decision Tree: 81%

Esse processo evidencia uma etapa fundamental em projetos de Machine Learning: entender de onde a performance está vindo e verificar se o modelo está realmente aprendendo padrões úteis, em vez de apenas receber indiretamente a resposta.

 Autor

ZuziR6

Projeto desenvolvido como estudo prático de Data Science, Machine Learning, análise de dados e modelagem preditiva, utilizando um problema real de negócio.

⭐ Se este projeto foi útil ou interessante, considere deixar uma estrela no repositório.
