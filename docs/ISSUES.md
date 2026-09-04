# Issues do Projeto instanciairis

> Gerado a partir dos comentários `#TODO`, `#bugfix` e pendências identificadas nos arquivos:
> `src/api/main.py`, `notebooks/fabricaia_example.ipynb` e `Makefile`.
>
> **Revisor padrão:** @diegocordeirodeoliveira
> **Branch base:** `develop`

---

## Resumo

| # | Arquivo | Linha/Célula | Título | Severidade | Status |
|---|---------|-------------|--------|------------|--------|
| 1 | `src/api/main.py` | 137 | Pipeline de preprocessamento no `/predict` | 🟡 Média | 🔴 Pendente |
| 2 | `src/api/main.py` | 160-162 | Serialização `numpy.int64` no Pydantic | 🔴 Alta | 🟢 Resolvido |
| 3 | `src/api/main.py` | 173-176 | Endpoint `/predict_batch_json` para JSON | 🔴 Alta | 🔴 Pendente |
| 4 | `src/api/main.py` | 241-242 | Implementar endpoint `/retrain` | 🟡 Média | 🔴 Pendente |
| 5 | `notebooks/fabricaia_example.ipynb` | Célula 1 | Função utilitária para localizar raiz do projeto | 🟢 Baixa | 🔴 Pendente |
| 6 | `notebooks/fabricaia_example.ipynb` | Célula 3 | Atualizar caminho do arquivo de dados | 🟢 Baixa | 🔴 Pendente |
| 7 | `notebooks/fabricaia_example.ipynb` | Célula 3 | Nomear coluna alvo como `target` | 🟢 Baixa | 🔴 Pendente |
| 8 | `notebooks/fabricaia_example.ipynb` | Célula 5 | Ajustar caminho do `config.yaml` | 🟢 Baixa | 🔴 Pendente |
| 9 | `Makefile` | 87 | Porta MLflow server 5000 → 5001 | 🟡 Média | 🟢 Resolvido |
| 10 | `Makefile` | 91 | Porta MLflow UI 5000 → 5001 | 🟡 Média | 🟢 Resolvido |

---

## Issues Detalhadas

---

### #1 — Pipeline de preprocessamento no `/predict`

- **Arquivo:** `src/api/main.py`
- **Linha:** 137
- **Severidade:** 🟡 Média
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/preprocessing-pipeline`

**Descrição:**

No endpoint `POST /predict`, os dados de entrada são convertidos para DataFrame mas **não passam por nenhum pipeline de preprocessamento** (ex: scaling, encoding). O código atual simplesmente atribui `processed_features = features_df` sem nenhuma transformação.

```python
# Linha 137 - Comentário atual:
# Preprocess features if needed
# This would depend on your specific preprocessing requirements
processed_features = features_df
```

**Critério de aceitação:**
- [ ] Carregar um pipeline de preprocessamento salvo (ex: `preprocessor.pkl`) no `startup_event`
- [ ] Aplicar o pipeline em `processed_features` antes da predição
- [ ] Garantir compatibilidade com modelos treinados com dados pré-processados
- [ ] Adicionar tratamento de erro caso o pipeline não exista

---

### #2 — Serialização `numpy.int64` no Pydantic

- **Arquivo:** `src/api/main.py`
- **Linhas:** 160-162
- **Severidade:** 🔴 Alta
- **Status:** 🟢 Resolvido
- **Branch sugerida:** `bugfix/numpy-serialization`

**Descrição:**

O endpoint `POST /predict` retornava `prediction[0]` (um `numpy.int64`) quando havia apenas 1 predição, causando:

```
PydanticSerializationError: Unable to serialize unknown type: <class 'numpy.int64'>
```

**Solução aplicada:**

```python
# Antes (linha 161 - comentada):
# prediction=prediction[0] if len(prediction) == 1 else prediction.tolist(),

# Depois (linha 162):
prediction=prediction[0].item() if len(prediction) == 1 else prediction.tolist(),
```

`.item()` converte `numpy.int64` → `int` nativo do Python, serializável pelo Pydantic.

**Critério de aceitação:**
- [x] Predição individual retorna 200 (não mais 500)
- [x] Resposta JSON contém tipo nativo (`int`), não `numpy.int64`

---

### #3 — Endpoint `/predict_batch_json` para aceitar JSON

- **Arquivo:** `src/api/main.py`
- **Linhas:** 173-176
- **Severidade:** 🔴 Alta
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/predict-batch-json`

**Descrição:**

O endpoint `POST /predict_batch` atual **só aceita upload de arquivo CSV** (`multipart/form-data`). Não há suporte para envio de múltiplas instâncias em JSON. Tentativas de enviar `{"instances": [...], "model_name": "..."}` resultam em erro 422:

```json
{"detail":[{"type":"missing","loc":["body","file"],"msg":"Field required","input":null}]}
```

```python
# Assinatura atual:
@app.post("/predict_batch")
async def predict_batch(
    file: UploadFile = File(...), model_name: str = "random_forest"
):
```

**Critério de aceitação:**
- [ ] Criar modelo Pydantic `BatchPredictionRequest` com `instances: List[Dict[str, Any]]` e `model_name: str`
- [ ] Adicionar novo endpoint `POST /predict_batch_json` que aceite JSON
- [ ] Manter o endpoint `/predict_batch` existente (compatibilidade reversa)
- [ ] Testar com curl:

```bash
curl -X POST "http://localhost:8000/predict_batch_json" \
     -H "Content-Type: application/json" \
     -d '{"instances": [{"features": {"sepal_length": 5.1, ...}}], "model_name": "model"}'
```

---

### #4 — Implementar endpoint `/retrain`

- **Arquivo:** `src/api/main.py`
- **Linhas:** 241-242
- **Severidade:** 🟡 Média
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/retrain-endpoint`

**Descrição:**

O endpoint `POST /model/{model_name}/retrain` está implementado como **placeholder** — não executa nenhum retreinamento real. Apenas retorna uma mensagem de confirmação simulada.

```python
@app.post("/model/{model_name}/retrain")
async def retrain_model(model_name: str, data_path: str):
    """Retrain a model with new data."""
    try:
        # This would implement retraining logic
        # For now, just return a placeholder response
        return {
            "message": f"Retraining {model_name} with data from {data_path}",
            "status": "initiated",
        }
```

**Critério de aceitação:**
- [ ] Carregar dados do `data_path` informado
- [ ] Executar pipeline completo: processamento → treinamento → avaliação
- [ ] Salvar novo modelo e atualizar `models` em memória
- [ ] Registrar nova versão no MLflow
- [ ] Retornar métricas do novo modelo (accuracy, f1, etc.)
- [ ] Tratar erros (arquivo não encontrado, coluna target ausente, etc.)

---

### #5 — Função utilitária para localizar raiz do projeto

- **Arquivo:** `notebooks/fabricaia_example.ipynb`
- **Local:** Célula 1 (code)
- **Severidade:** 🟢 Baixa
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/notebook-utils`

**Descrição:**

```python
#TODO: Sugerir alteração
# - Funções utilitárias para localizar a raiz do projeto dinamicamente
```

O notebook usa caminhos relativos que dependem do diretório de execução. Uma função utilitária `get_project_root()` evitaria problemas de path.

**Critério de aceitação:**
- [ ] Criar função `get_project_root()` que identifique a raiz do projeto (ex: pelo `pyproject.toml`)
- [ ] Substituir caminhos hardcoded por chamadas à função
- [ ] Garantir que funcione independentemente do diretório de execução do notebook

---

### #6 — Atualizar caminho do arquivo de dados no notebook

- **Arquivo:** `notebooks/fabricaia_example.ipynb`
- **Local:** Célula 3 (code)
- **Severidade:** 🟢 Baixa
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/notebook-paths`

**Descrição:**

```python
#TODO: Sugerir alteração
# - Atualizar caminho do arquivo de dados
```

O caminho atual para o dataset (`data/raw/iris.csv`) pode não existir ou estar desatualizado. Deve ser validado e, se necessário, usar `get_project_root()`.

**Critério de aceitação:**
- [ ] Verificar se o arquivo `data/raw/iris.csv` existe
- [ ] Se não existir, criar célula para download automático do dataset Iris
- [ ] Usar caminho relativo à raiz do projeto

---

### #7 — Nomear coluna alvo como `target`

- **Arquivo:** `notebooks/fabricaia_example.ipynb`
- **Local:** Célula 3 (code)
- **Severidade:** 🟢 Baixa
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/notebook-paths`

**Descrição:**

```python
#TODO: Sugerir alteração no comentário
# - Nomear coluna alvo como 'target'
```

O notebook deve deixar explícito que a coluna alvo deve se chamar `target` (ou tornar isso configurável), para consistência com o restante do pipeline.

**Critério de aceitação:**
- [ ] Documentar que a coluna alvo esperada é `target`
- [ ] Adicionar verificação: se coluna `target` não existir, exibir mensagem clara
- [ ] Ou tornar o nome da coluna alvo um parâmetro da célula

---

### #8 — Ajustar caminho do `config.yaml` no notebook

- **Arquivo:** `notebooks/fabricaia_example.ipynb`
- **Local:** Célula 5 (code)
- **Severidade:** 🟢 Baixa
- **Status:** 🔴 Pendente
- **Branch sugerida:** `feature/notebook-config-path`

**Descrição:**

```python
#TODO: Sugerir alteração
# - Ajustar caminho do config.yaml
```

O caminho para `config/config.yaml` deve ser validado e usar `get_project_root()` para robustez.

**Critério de aceitação:**
- [ ] Usar caminho absoluto baseado na raiz do projeto
- [ ] Exibir mensagem de erro clara se o arquivo não for encontrado

---

### #9 — Porta MLflow server de 5000 para 5001

- **Arquivo:** `Makefile`
- **Linha:** 87
- **Severidade:** 🟡 Média
- **Status:** 🟢 Resolvido
- **Branch sugerida:** `bugfix/mlflow-port-macos`

**Descrição:**

A porta 5000 conflita com o AirPlay Receiver no macOS. O TODO indicava a troca para 5001.

```makefile
# Antes (comentário na linha 87):
# TODO: trocar porta 5000 para 5001 (AirPlay conflita no macOS)
mlflow-server:
	mlflow server --backend-store-uri sqlite:///mlflow.db --host 0.0.0.0 --port 5001
```

**Solução aplicada:** Porta alterada de `--port 5000` para `--port 5001`.

**Critério de aceitação:**
- [x] `make mlflow-server` sobe na porta 5001
- [x] Não há conflito com AirPlay Receiver

---

### #10 — Porta MLflow UI de 5000 para 5001

- **Arquivo:** `Makefile`
- **Linha:** 91
- **Severidade:** 🟡 Média
- **Status:** 🟢 Resolvido
- **Branch sugerida:** `bugfix/mlflow-port-macos`

**Descrição:**

Mesmo caso da issue #9, afetando o target `mlflow-ui` do Makefile.

```makefile
# Antes (comentário na linha 91):
# TODO: trocar porta 5000 para 5001 (AirPlay conflita no macOS)
mlflow-ui:
	mlflow ui --backend-store-uri sqlite:///mlflow.db --host 0.0.0.0 --port 5001
```

**Solução aplicada:** Porta alterada de `--port 5000` para `--port 5001`.

**Critério de aceitação:**
- [x] `make mlflow-ui` sobe na porta 5001
- [x] Não há conflito com AirPlay Receiver

---

## Fluxo de trabalho sugerido

Para cada issue pendente (#1, #3, #4, #5, #6, #7, #8):

```bash
# 1. Criar branch a partir da develop
git checkout develop
git pull origin develop
git checkout -b <branch-sugerida>

# 2. Implementar a correção/melhoria
# ...

# 3. Commit seguindo conventional commits
git add .
git commit -m "feat(api): implementa endpoint /predict_batch_json

Closes #3"

# 4. Push e abrir PR
git push -u origin <branch-sugerida>
gh pr create --base develop --title "Título da issue" --body "Closes #N" --reviewer diegocordeirodeoliveira
```

---

## Progresso geral

| Status | Quantidade |
|--------|-----------|
| 🔴 Pendente | 6 |
| 🟢 Resolvido | 4 |
| **Total** | **10** |

---

> Última atualização: 09/06/2026