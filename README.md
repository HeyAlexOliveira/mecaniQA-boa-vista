# mecaniQA-boa-vista

## Contexto

Este repositório reúne as entregas do time **Boa Vista** para o desafio **MecâniQA**.

A MecâniQA, rede de oficinas e auto centers de Feira de Santana e região, enfrenta picos inesperados de demanda de manutenção preventiva, o que gera falta de peças em alguns dias e ociosidade da equipe de mecânicos em outros. O time atua como o esquadrão de Ciência de Dados responsável por construir o núcleo do sistema preditivo da empresa, prevendo a demanda diária de **Trocas de Óleo** e **Manutenções de Motor**.

## Time: Boa Vista

| Membro   | Nome                        |
| -------- | --------------------------- |
| Membro 1 | Alex Oliveira Santos        |
| Membro 2 | Alice Gomes Aragão          |
| Membro 3 | Ana Clara Ribeiro da Silva  |
| Membro 4 | Lorran Edley Matos Ribeiro  |
| Membro 5 | Rodrigo dos Anjos Lamartine |

## Conteúdo do repositório

| Arquivo | Descrição |
| --- | --- |
| `mecaniqa_dataset.xlsx` | Histórico diário de manutenções (01/01/2024 a 31/12/2025), usado nas três entregas |
| `mecaniQA_boa-vista.ipynb` | Notebook da OAT 1 |
| `mecaniQA_boa-vista_oat2.ipynb` | Notebook da OAT 2 |
| `mecaniQA_boa-vista_oat3.ipynb` | Notebook da OAT 3 |
| `mecaniQA_oat1_boa-vista.pdf`, `mecaniQA_oat2_boa-vista.pdf`, `mecaniQA_oat3_boa-vista.pdf` | Apresentações de cada entrega |
| `requirements.txt` | Dependências para executar os notebooks |

---

## OAT 1: Compreensão e Baseline

**Objetivo:** compreender a estrutura de dados temporais e criar o primeiro motor de previsão "ingênuo" (baseline).

- Indexação temporal e verificação da integridade da linha do tempo.
- Tratamento de dados ausentes (interpolação temporal) e de outliers (método IQR, com clipping).
- Análise exploratória: decomposição da série em Tendência, Sazonalidade e Ruído.
- Primeiros baselines: Naive e Médias Móveis (7 e 30 dias).

## OAT 2: Baseline e Métricas de Avaliação

**Objetivo:** avaliar o baseline com rigor, sem vazamento de dados, e preparar as features temporais.

- Baselines como funções puras, com `.shift(1)` sempre depois do `.rolling()`, e QA automatizado contra Data Leakage.
- Motor de avaliação com MAE, RMSE e MAPE.
- Validação com `TimeSeriesSplit` (5 folds cronológicos) no lugar de divisões aleatórias.
- Feature Engineering: Lags (1, 7 e 30 dias) e Rolling Window (7 dias).

## OAT 3: Tunando o Meu Modelo

**Objetivo:** construir, otimizar e comparar modelos de Machine Learning contra os baselines.

1. **Modelos clássicos de regressão** com hiperparâmetros padrão: Regressão Linear, Árvore de Decisão, Random Forest e Gradient Boosting (`fit()` só no treino, `predict()` no teste).
2. **Grid Search** do Random Forest (`n_estimators`, `max_depth`, `min_samples_leaf`: 48 combinações, 240 treinos por variável), integrado ao `TimeSeriesSplit`.
3. **Random Search** com `n_iter = 10` sobre o mesmo espaço de busca, cronometrado contra o Grid Search.
4. **Comparação rigorosa** entre Baseline antigo, ML inicial e ML tunado, com MAE, RMSE e MAPE calculados ao mesmo tempo e gráfico de linha sobreposto (Real × Baseline × Modelo tunado).

**Divisão temporal:** os primeiros 80% dos dias (31/01/2024 a 12/08/2025, 560 dias) para treino e os 20% finais (13/08 a 31/12/2025, 141 dias) para teste. A busca de hiperparâmetros e a escolha do modelo usam só o treino; o teste só é usado no final.

### Resultados (conjunto de teste)

| Variável | Cenário | MAE | RMSE | MAPE |
| --- | --- | --- | --- | --- |
| Trocas de Óleo | Baseline antigo (Naive) | 6,48 | 8,92 | 26,8% |
| Trocas de Óleo | ML inicial (Random Forest padrão) | 3,04 | 3,81 | 13,3% |
| Trocas de Óleo | ML tunado (Random Forest, Grid Search) | **3,13** | **3,82** | **13,9%** |
| Manutenção de Motor | Baseline antigo (Média Móvel 7d) | 3,23 | 3,82 | 41,1% |
| Manutenção de Motor | ML inicial (Random Forest padrão) | 2,05 | 2,48 | 28,4% |
| Manutenção de Motor | ML tunado (Random Forest, Grid Search) | **2,00** | **2,39** | **27,6%** |

- O modelo tunado reduz o MAE do baseline em **52%** (Trocas de Óleo) e **38%** (Manutenção de Motor).
- O Grid Search reduz o erro de validação (RMSE) em cerca de 11% (Óleo) e 8% (Motor), mas no teste o ganho sobre o Random Forest padrão é pequeno: o maior salto vem de trocar o baseline por Machine Learning.
- O **Random Search** chega a praticamente o mesmo erro do Grid Search (RMSE no teste de 3,84 contra 3,82 em Óleo e de 2,44 contra 2,39 em Motor) em cerca de **um quinto do tempo**.
- O atributo mais importante do modelo é o **lag de 7 dias**, o que confirma a sazonalidade semanal da oficina.

### Decisões de anti-leakage

- Divisão cronológica, sem embaralhar os dias.
- Features com `shift`: cada linha só carrega informação do passado.
- Um QA automatizado confirma, no notebook, que o treino vem antes do teste e que as features só usam o passado.
- `GridSearchCV` e `RandomizedSearchCV` usam `TimeSeriesSplit` e rodam só no treino.

---

## Como executar

Python 3.10 ou superior.

```
pip install -r requirements.txt
python -m notebook
```

Abra o notebook desejado e use **Kernel > Restart Kernel and Run All Cells**. O `mecaniqa_dataset.xlsx` precisa estar na mesma pasta do notebook. O notebook da OAT 3 leva de 2 a 3 minutos para rodar, por causa do Grid Search (240 treinos por variável).
