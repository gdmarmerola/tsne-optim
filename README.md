# tsne-optim

## Otimizando visualizações de dados através da configuração automática do t-SNE

Neste repositório, apresento meu trabalho de conclusão de curso na Unicamp: "Ajuste automático de hiperparâmetros do algoritmo t-SNE para visualização de dados de elevada dimensão", sob a orientação do Prof. Dr. Fernando José Von Zuben.  

[Link](https://www.dropbox.com/sh/rmsyfs2k72vnqni/AABlCgLb4dfHg6FIvVnM3jqNa?dl=0) com os dados, visualizações e histórico dos experimentos.

## Introdução e Metodologia

O ajuste dos hiperparâmetros do t-SNE não é trivial. Apesar existirem argumentos suportando a tese de que [o algoritmo é insensível aos seus hiperparâmetros](http://www.jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf), outras explorações apontam que o buraco é mais embaixo, mostrando que os [hiperparâmetros alteram significativamente o resultado do algoritmo.](https://distill.pub/2016/misread-tsne/).

Visando suportar o cientista de dados nesta tarefa, desenvolvemos, neste trabalho, uma metodologia de configuração automática para o t-SNE. A lógica é bastante simples e bem representada na figura abaixo:

![](https://github.com/gdmarmerola/tsne-optim/blob/master/Pipeline.png)

A metodologia mostrada consiste de três partes:

**1 - Pipeline.** Construímos uma classe que armazena os dados em análise e implementa um método que recebe um conjunto de hiperparâmetros e executa o t-SNE (após PCA) com esta configuração. Além disso, implementamos funções para extrair clusters a partir dos dados em baixa e alta dimensionalidade e outras funções para computar as métricas a seguir.

**2 - Métricas.** No final do *Pipeline* utilizamos métricas para medir aspectos que queremos otimizar na execução do t-SNE. Foram deifinidas três métricas:

* *Rank-order correlation*: medimos a preservação de estrutura global comparando o ordenamento das distâncias entre os clusters em alta e em baixa dimensionalidade. Suponha grupos (A), (B) e (C), e que o grupo mais próximo de (A) no espaço original é (B), seguido de (C). Se em baixa dimensionalidade tivermos esse mesmo ordenamento, consideramos que o algoritmo preserva estutura global (rank-order correlation igual a 1).
* *Ajusted Mutual Information*: medimos consistência de agrupamento através de cálculo da informação mútua ajustada (AMI) entre os agrupamentos obtidos com HDBSCAN em baixa dimensionalidade e em alta dimensionalidade. Caso os agrupamentos concordem, podemos dizer que o algoritmo preserva a estrutura de agrupamento nos dados (AMI igual a 1).
* *Divergênccia KL*: O t-SNE é conhecido por terminar em mínimos locais. Portanto, o valor final da divergência KL pode ser utilizado como uma métrica a ser otimizada, evitando essa situação. Contudo, é preciso cuidado: a perplexidade deve estar fixa, e é desejável também fixar os hiperparâmetros do PCA (apesar de resultados interessantes terem sido obtidos de outra maneira).

**3 - Otimização.** A combinação do Pipeline com uma métrica de avaliação define uma função-objetivo, que podemos otimizar. São realizados experimentos com três algoritmos de otimização de hiperparâmetros populares: busca aleatória e TPE, com o módulo [hyperopt](http://hyperopt.github.io/hyperopt/) e Processos Gaussianos (módulo [BayesianOptimization](https://github.com/fmfn/BayesianOptimization)).

O processo é conduzido da seguinte forma:

1. Definem-se espaços de busca através de distribuições de probabilidade para os métodos de busca aleatória e TPE. Para Processos Gaussianos, basta definir o espaço de busca através de fronteiras (limites superiores e inferiores) para cada um dos hiperparâmetros.
2. É instanciada uma tarefa de otimização escolhendo um conjunto de dados.
3. Para cada rodada de otimização, a sequência de PCA e t-SNE é executada sob a configuração de hiperparâmetros selecionada pela metodologia de otimização em análise. Utilizando os mapeamentos e os resultados de agrupamento, computamos as métricas de avaliação definidas. Para a métrica rank-order correlation (estrutura global), HDBSCAN é executado somente nos dados em alta dimensionalidade, enquanto que para AMI (consistência de agrupamento) HDBSCAN é executado tanto em alta dimensionalidade quanto em baixa dimensionalidade. Para a métrica divergência KL, HDBSCAN não é executado. 
4. O algoritmo de otimização de hiperparâmetros recebe o resultado do experimento, calculando uma nova configuração para o *Pipeline*.

## Notebooks

Os Notebooks disponíveis implementam experimentos com as três métricas de avaliação consideradas, três algoritmos de otimização e seis conjuntos de dados. Segue descrição: 
* `notebooks/ami-optim.ipynb`: experimentos para otimizar a métrica de informação mútua ajustada.
* `notebooks/baseline-plots.ipynb`: construindo visualizações com os hiperparâmetros sugeridos por Maaten & Hinton, para comparação qualitativa.
* `notebooks/kl-divergence-optim.ipynb`: experimentos para otimizar a métrica de divergência KL.
* `notebooks/optimizer-test.ipynb`: teste dos algoritmos de otimização.
* `notebooks/rank-order-optim.ipynb`: experimentos para otimizar a métrica de *rank-order correlation*.
* `notebooks/real-data-processing.ipynb`: criando as bases de dados reais.
* `notebooks/synthetic-data-gen.ipynb`: criando as bases de dados sintéticas.

## Resultados

Os resultados foram promissores. As melhores configurações segundo as métricas definidas mostraram que existem intervalos mais amplos para a configuração do t-SNE. Particularmente, perplexidades mais altas que o usual mostraram bons resultados. Uma avaliação mais completa é feita no relatório contido neste repositório. Por ora, mostramos uma visualização do dataset [COIL-20](http://www.cs.columbia.edu/CAVE/software/softlib/coil-20.php) com hiperparâmetros sugeridos por [Maaten & Hinton](http://www.jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf) e com os hiperparâmetros ótimos para o objetivo de representação de estrutura global.

![](https://github.com/gdmarmerola/tsne-optim/blob/master/coil-20-default.png "Hiperarâmetros sugeridos por Maaten & Hinton")
<div style="text-align:center">Hiperarâmetros sugeridos por Maaten & Hinton</div>
![](https://github.com/gdmarmerola/tsne-optim/blob/master/coil-20-best.png "Hiperparâmetros otimizados para preservação de estrutura global")
<div style="text-align:center">Hiperparâmetros otimizados para preservação de estrutura global</div>

Os clusters apresentam uma separação razoavelmente mais clara na visualização otimizada, obtida com uma perplexidade de 115 (fora do intervalo considerado como boa prática). 
