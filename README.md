# InstanciaIris

Pipeline reproduzível de aprendizado de máquina para o **dataset Iris**, instanciado a partir do **template FabricaIA**. O projeto cobre o fluxo completo: pré-processamento (codificação do rótulo e padronização das características), treinamento de um classificador **Random Forest** com rastreamento de experimentos em **MLflow**, persistência do modelo e exposição de um endpoint de predição REST via **FastAPI**.

Repositório: [https://github.com/diegocordeiro/InstanciaIris.git](https://github.com/diegocordeiro/InstanciaIris.git)

## Destaques

- Notebook executado de ponta a ponta sem erros.
- Dois `runs` registrados no experimento `instanciairis_experiments` do MLflow (backend SQLite `sqlite:///mlflow.db`):
  - `random_forest_baseline` — `RandomForestClassifier` (100 estimadores, profundidade 10) com **acurácia 0,9333** e **F1 0,90**.
  - `entrega_topicos_avancados` — Random Forest com hiperparâmetros padrão, alcançando **acurácia 0,9667**, precisão **1,00**, recall **0,90** e F1 **0,9474**.
- Modelo persistido em `models/trained/model.pkl` e registrado como artefato no experimento `instanciairis_experiments`.
- API `POST /predict` respondeu à requisição de teste com a classe e as probabilidades associadas.
- Validação do fluxo ponta a ponta por meio de **18 testes automatizados aprovados** e de uma chamada de predição real.

## Ambiente

- Python 3.12.9
- scikit-learn 1.9.0
- MLflow 3.16.0 (backend SQLite local)
- pandas, numpy e FastAPI

## Estrutura do Projeto utilizada

```text
InstanciaIris/
├── config/             # Arquivos de configuração (config.yaml)
├── data/               # Dados (raw, processed, external)
├── docs/               # Documentação técnica e relatório SBC
├── models/             # Modelos treinados e artefatos
├── notebooks/          # Notebook de experimentação (fabricaia_example.ipynb)
├── src/                # Código-fonte (componentes do pipeline)
└── tests/              # Testes automatizados
```

## Pipeline e Resultados

- **Dataset:** `data/raw/iris.csv` — 150 linhas, 4 características numéricas e a coluna `target` (setosa, versicolor, virginica).
- **Pré-processamento:** limpeza, `LabelEncoder` no alvo (0/1/2), `StandardScaler` nas características e divisão 80/20 (119 amostras de treino).
- **Modelo:** `RandomForestClassifier(n_estimators=100, max_depth=10)` treinado via `ModelTrainer` e rastreado no experimento `instanciairis_experiments`.
- **Métricas registradas no MLflow:**

| Run | Acurácia | Precisão | Recall | F1 |
| --- | --- | --- | --- | --- |
| `random_forest_baseline` | 0,9333 | 0,90 | 0,90 | 0,90 |
| `entrega_topicos_avancados` | 0,9667 | 1,00 | 0,90 | 0,9474 |

O run `random_forest_baseline` (100 estimadores, profundidade máxima 10) forneceu desempenho estável, com acurácia de 93,33% e F1 de 0,90. Já o run `entrega_topicos_avancados`, treinado com os hiperparâmetros padrão do Random Forest, alcançou métricas superiores (acurácia 0,9667, precisão 1,00, recall 0,90 e F1 0,9474), evidenciando o efeito da configuração de hiperparâmetros.

- **Artefato:** `models/trained/model.pkl` (RandomForestClassifier).

### Exemplo de predição (API)

Na inicialização, a aplicação FastAPI carrega os modelos de `models/trained/*.pkl` e deriva o nome do modelo removendo o sufixo `_model`. O endpoint `POST /predict` recebe um corpo JSON com as características e o nome do modelo e retorna a classe predita, as probabilidades e a confiança.

Requisição:
```json
{
  "features": {
    "sepal_length": 5.1,
    "sepal_width": 3.5,
    "petal_length": 1.4,
    "petal_width": 0.2
  },
  "model_name": "model"
}
```

Resposta:
```json
{
  "prediction": 2,
  "probability": {
    "0": 0.01,
    "1": 0.24,
    "2": 0.75
  },
  "model_name": "model",
  "confidence": 0.75
}
```

## Documentação relacionada

- Relatório técnico (padrão SBC): `docs/relatorios/relatorio.tex`.
- Guia de início rápido: `docs/QUICKSTART.md`.

