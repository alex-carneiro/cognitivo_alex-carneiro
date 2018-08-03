# README

Análise do arquivo _winequality.csv_.

*Objetivo*: construir um modelo capaz de predizer a qualidade de uma amostra de vinho, em uma escala de 0 ~ 10 (valores inteiros), baseado nas seguintes propriedades físico-químicas:

* pH
* dióxido de enxofre livre
* sulfatos
* densidade
* acidez fixa
* Açúcar residual
* acidez volátil
* ácido cítrico
* tipo
* dióxido de enxofre total
* cloretos
* álcool

## Respostas

### a) Como foi a definição da sua estratégia de modelagem?

I baseei minha estratégia de modelagem nas metodologias CRISP-DM e KNN. Primeiro, I busquei por dados fora de conformidade e valores ausentes no dataset. Não havia valores ausentes e eu removi os dados fora de conformidade.

Eu percebi um padrão nos dados fora de conformidade no campo *álcool*, em que eles têm pontos para cada três dígitos e a posição mais provável para o ponto é entre o segundo e terceiro dígitos. Mas como havia apenas 40 amostras fora de conformidade e este tipo de suposição não é sempre confiável, eu decidi apenas remover essas amostras.

O segundo passo foi entender a forma e distribuição dos dados de cada campo.

O terceiro passo foi obter a matriz de correlação para os campos e a correlação entre cada campo e a saída desejada.

Se a matriz de correlação apresentasse valores altos, uma transformação PCA poderia melhorar os resultados. Mas decidi não usá-la porque a matriz de correlação apresentou a maoria dos valores abaixo de _0.3_.


### b) Como foi definida a função de custo utilizada?

Eu escolhi três algoritmos para os testes, dois deles baseados na técnica de mínimos quadrados (Linear Regression and SVM) e o outro é um algoritmo de clustering adaptado para realizar regressões (KNN).

Cada algoritmo foi testado com os dados originais e normalizados. As métricas de validação incluem o Erro Absoluto Médio (MAE), Acurácia e Erro menor que 1.


### c) Qual foi o critério utilizado na seleção do modelo final?

Eu escolhi o algoritmo SVR com os dados normalizados porque este apresentou o menor MAE, maior Acurácia e maior robustez aos erros menores.


### d) Qual foi o critério utilizado para validação do modelo? Por que escolheu utilizar este método?

Eu busquei primeiro por algoritmos que fornecessem baixo MAE e alta Acurácia. Mas, após pensar sobre o problema, decidi considerar também o erro menor que 1 e a robustez a erros menores.

Escolhi estas métricas para validar o modelo porque este problema contém uma componente subjetiva em que o modelo deve ser capaz de prever a impressão de um consumidor baseado em propriedades físico-químicas. Então, um erro de 1 unidade me parece tolerável.

Eu decidi não fazer uma abordagem de classificação porque os valores de saída são sequecialmente relacionados e as abordagens de classificação fornecem resultados melhores para problemas em que os valores para a saída não apresentam relacção, como uma sequência.


### e) Quais evidências você possui de que seu modelo é suficientemente bom?

Meu modelo final é capaz de predizer a qualidade do vinho com um erro esperado de 0.452, isto significa 1 erro de menos de 1 unidade por 2 amostras de vinho em média.

Outra evidência é que podemos esperar um erro menor ou igual a 1 em 94.24% das amostras, ista parece uma boa confiabilidade para uma predição dentro do erro que me parece tolerável.

A última evidência que coletei é que dado uma amostra com erro menor ou igual a 1, temos 64.4% de probabilidade que a predição esteja correta.


O restante deste documento explica como obtive os resultados.


## Data exploration

O arquivo _First Contact.ipynb_ contém a primera análise exploratória feita no dataset.

### Insights

O dataset tem 6497 amostras de vinho.

Os valores de *quality* estão no intervalo _3 ~ 9_.

O campo *alcohol* é lido como string porque alguns valores tem mais de um ponto, por exemplo: '128.933.333.333.333'. Estes valores foram removidos.

Todos os campos apresentam valores em intervalos diferentes. Então uma normalização pode melhorar os resultados de predições. A normalização escolhida faz com que _mean(campo)=0_ and _variance(campo)=1_, mas não interfere no histograma nem na correlação entre os campos.

O campo *type* é binário {Red, White}, de modo que este foi mapeado para {0,1}, respectivamente.

A maioria dos campos apresentam baixa correlação entre si, apenas *total sulfur dioxide* e *free sulfur dioxide* apresentaram correlação acima de _0.7_.

Todos os campos têm baixa correlação com a saída, o maior valor é observado no campo *alcohol* com correlação de _0.44_.


## Modeling

O arquivo _Regression Approach.ipynb_ contém os experimentos feitos com os diferentes algoritmos.

Como a saída desejada é uma escala, uma abordagem baseada em regressão é conveniente para minimizar a diferença entre os valores reais e os preditos.

Os três algoritmos de regressão foram:

* Linear Regression;
* SVR (regressão baseada em SVM);
* KNN Regression.

Os três algoritmos foram avaliados com os dados originais e normalizados.

A taxa de treinamento/teste foi 90%/10%.

As métricas de validação dos métodos foram:

* _MAE_: Erro Absoluto Médio.
* _Accuracy_: Taxa de valores de sáida preditos corretamente.
* _Error=1_: Taxa de valores de saída nos quais o erro observado é 1, por exemplo: a qualidade real é 6 mas o modelo predisse 7 ou 5.
* _Error<=1_: Taxa de valores de saída nos quais o erro observado é menor ou igual a 1, isto é _Accuracy + Error=1_.

### Resultados da Linear Regression

|Métrica |Dados originais|Dados normalizados|
|--------|---------------|------------------|
|MAE     |0.558          |0.558             |
|Accuracy|48.92%         |48.92%            |
|Error=1 |43.96%         |43.96%            |
|Error<=1|92.98%         |92.98%            |


### Resultados do SVR

|Métrica |Dados originais|Dados normalizados|
|--------|---------------|------------------|
|MAE     |0.497          |0.469             |
|Accuracy|56.04%         |59.75%            |
|Error=1 |38.70%         |33.75%            |
|Error<=1|94.74%         |93.50%            |

### Resultados do KNN Regression

|Métrica |Dados originais|Dados normalizados|
|--------|---------------|------------------|
|MAE     |0.597          |0.534             |
|Accuracy|47.83%         |52.94%            |
|Error=1 |45.20%         |40.87%            |
|Error<=1|93.03%         |93.81%            |


## Model evaluation

O modelo SVR com dados normalizados foi o escolhido porque apresentou o menor _MAE_ de todos (0.469), maior _Acurácia_ de todas (59.75%) e maior taxa _Accuracy/(Accuracy + (Error=1))_ de todas (0.639). A última métrica significa que o modelo é mais robusto a erros menores, este valor faz com que este modelo seja melhor que o SVR com dados originais mesmo que apresente um valor para _Error<=1_ superior ao obtido pelo SVR com dados normalizados.

O modelo SVR foi validado com o método KFold, no qual o dataset foi divido em 10 subconjuntos e cada um foi usado para teste enquanto os outros 9 foram usados para treino. Este método produziu 10 predições, uma para cada subconjunto, e a avaliação final foi obtida pela média destas predições:

* Max MAE = 0.474
* Min MAE = 0.423
* Mean MAE = 0.452
* Var of MAE = 0.00028
* Mean Accuracy = 60.88%
* Mean Error=1 = 33.54%

### Conclusion

O modelo SVR treinado com dados normalizados é capaz de predizer a qualidade de uma amostra de vinho com um erro absoluto esperado de 0.452, acurácia de 60.88%, erro esperado menor ou igual a 1 de 94.24% e taxa _Accuracy/(Accuracy + (Error=1))_ = 0.644.
