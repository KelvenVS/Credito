## 📋 Relatório Executivo - Modelo Preditivo de Inadimplência

### 1. Objetivo do Projeto
O projeto visa **identificar clientes com maior risco de inadimplência** a partir de dados históricos de operações financeiras, demográficas e comportamento de crédito. O modelo serve como suporte para **decisão de concessão de crédito** e **gestão de risco** em uma instituição financeira.

### 2. Conjunto de Dados Utilizado
Foram utilizados **arquivos no formato .parquet**, todos relacionados ao histórico financeiro e perfil do cliente:

- `train_base.parquet`: base principal com variável-alvo `target` (1 = inadimplente, 0 = pagador).
- `train_person_1.parquet`: dados pessoais (gênero, idade, número de filhos).
- `train_deposit_1.parquet`: informações sobre valores e datas de depósitos.
- `train_credit_bureau_b_1.parquet`: histórico de crédito externo (Bureau).

Esses dados foram integrados e tratados em um **dataframe mestre** para alimentação do modelo.

### 3. Engenharia de Variáveis
Foi realizada a extração de variáveis relevantes, incluindo:

- **Demografia**: idade média (`person_age_mean`), percentual feminino, média de filhos.
- **Depósitos**: valor médio (`dep_amt_mean`), valor máximo (`dep_amt_max`), duração do contrato.
- **Crédito externo**: valor total, valor vencido, dias desde último contrato (`cb_b1_recency_days`).
- **Temporais**: `WEEK_NUM` (semana da decisão), `MONTH` (mês).

Foi identificado **data leakage com a variável `case_id`**, pois ela refletia a ordem temporal. Ela foi removida antes do modelo final.

### 4. Modelo Utilizado
O modelo foi treinado usando **LightGBM**, com divisão estratificada (80% treino / 20% validação). As principais configurações:

- Objetivo: `binary`
- Métrica: `AUC`
- `early_stopping`: 50 rodadas
- Balanceamento de classes ativado (`is_unbalance=True`)

### 5. Principais Resultados

- **AUC na validação**: 0.625
- **Gini**: 0.25 (interpretação: quanto maior, melhor distinção entre inadimplentes e bons pagadores)
- **Top 10 variáveis mais importantes**:
  1. `WEEK_NUM`
  2. `person_age_mean`
  3. `dep_amt_max`
  4. `dep_amt_mean`
  5. `cb_b1_recency_days`
  6. `dep_dur_mean`
  7. `person_record_count`
  8. `MONTH`
  9. `cb_b1_amt_max`
  10. `cb_b1_amt_sum`

A importância dessas variáveis mostra que **tempo da operação, perfil do cliente e histórico financeiro** são cruciais para prever inadimplência.

### 6. Curva ROC

A curva ROC revelou desempenho acima do acaso (linha diagonal). A área sob a curva (AUC = 0.625) indica **discriminação moderada** do modelo. Idealmente, AUC > 0.70 é preferível para produção, mas já é um bom ponto de partida.

### 7. Estabilidade Temporal

A análise de Gini semanal mostra que o modelo **mantém performance consistente ao longo das semanas**, com oscilações normais. Não há sinais graves de instabilidade ou overfitting temporal.

### 8. Principais Insights

- Clientes com **operações recentes e valores mais altos** de depósito tendem a ser melhores pagadores.
- **Idade média mais alta** também está associada a menor risco.
- **Recência no crédito externo** tem impacto: quanto mais recente o crédito anterior, maior o risco percebido.

### 9. Recomendações

- ❌ **Excluir `case_id` e proxies temporais** para evitar vazamento de dados no modelo final.
- ⏳ **Monitorar o modelo ao longo do tempo** (principalmente se novas semanas forem adicionadas).
- 🌐 **Aplicar re-treinamentos periódicos** com novos dados para manter estabilidade.
- 🔍 Investigar novas features (ex: comportamentais, scores externos).
- 🚀 Testar outras abordagens como ensemble models e otimização de hiperparâmetros (GridSearch ou Optuna).

### 10. Conclusão

O modelo LightGBM desenvolvido mostra **capacidade real de identificar inadimplência** com boa estabilidade temporal. Embora o desempenho ainda possa melhorar, ele é funcional e fornece insights valiosos para negócio e futuras evoluções.

