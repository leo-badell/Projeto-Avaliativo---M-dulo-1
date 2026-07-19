# 🛒 Projeto de Predição de Churn em E-Commerce

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![Machine Learning](https://img.shields.io/badge/Machine%20Learning-Scikit--learn-orange.svg)
![Status](https://img.shields.io/badge/Status-Concluído-success.svg)

---

## 📖 Sobre o Projeto

Este projeto foi desenvolvido como parte do **Módulo 1 do Curso de Machine Learning e Visão Computacional**, com o objetivo de aplicar técnicas de **aprendizado supervisionado** para prever o **churn de clientes** em uma plataforma de e-commerce.

### 🎯 Objetivo

Desenvolver um modelo de classificação capaz de identificar clientes com **alta probabilidade de abandono** (churn), permitindo que a empresa tome ações preventivas como:
* Envio de cupons de desconto personalizados
* Campanhas de retenção direcionadas
* Melhoria da experiência do cliente

### 💡 Por que este projeto existe?

No setor de e-commerce, **reter clientes é 5-25x mais barato que adquirir novos**. Um modelo preditivo de churn permite:
* **Maximizar o LTV** (Lifetime Value) dos clientes
* **Reduzir custos operacionais** com aquisição
* **Otimizar campanhas de marketing** com foco em clientes em risco
* **Proteger a receita** evitando perda de base de clientes

---

## 🏗️ Estrutura do Projeto

O projeto foi organizado em **6 fases** sequenciais, seguindo boas práticas de ciência de dados:

### **Fase 1: Análise Exploratória de Dados (EDA)** 📊
* Carregamento e inspeção inicial do dataset
* Análise de distribuição de classes (desbalanceamento 83%/17%)
* Identificação de correlações com a variável target
* Visualizações: gráficos de barras, violin plots, boxplots

**Principais descobertas:**
* **Tenure** (tempo de cliente) é a variável mais preditiva (-0.35)
* **Complain** (reclamações) aumenta drasticamente o risco (+0.25)
* Dataset desbalanceado: 4.682 retidos vs 948 churn

---

### **Fase 2: Tratamento e Limpeza de Dados** 🧹
* Remoção de 556 duplicatas (9.9% do dataset)
* Tratamento de valores nulos em 7 colunas (4-6% missings)
  - **Mediana** para variáveis assimétricas (Tenure, DaySinceLastOrder)
  - **Média** para variáveis simétricas (HourSpendOnApp, OrderAmountHikeFromlastYear)
* Análise e manutenção de outliers (representam comportamentos reais)

**Resultado:** Dataset limpo com **5.074 linhas**, 0 nulos, 0 duplicatas.

---

### **Fase 3: Feature Engineering** ⚙️
* Teste de criação da feature `cashback_por_pedido` (CashbackAmount / OrderCount)
* **Análise empírica:** A feature derivada **piora o modelo**
  - ROC-AUC: 0.9362 (original) → 0.9069 (derivada) ❌
  - Correlação com Churn: -0.15 → -0.06 ❌
* **Decisão:** Manter apenas `CashbackAmount` original

**Conclusão:** Dividir uma variável útil por uma irrelevante dilui o sinal preditivo.

---

### **Fase 4: Preparação dos Dados para Modelagem** 🔐
* **Encoding de variáveis categóricas:** One-Hot Encoding com `drop_first=True`
  - 20 → 30+ colunas criadas
* **Split estratificado (80/20):** `train_test_split` com `stratify=y`
  - Mantém proporção de classes (17% churn) em treino E teste
* **Filtro de colunas contínuas:** 12 variáveis para KNN (precisa de escalonamento)
* **Proteção contra data leakage:** SMOTE e StandardScaler aplicados **DENTRO das pipelines** (Fase 5)

---

### **Fase 5: Modelagem e Validação** 🔬
* **Validação cruzada (5-fold CV) com pipeline seguro:**
  - Pipeline: `SMOTE → StandardScaler → Modelo`
  - SMOTE aplicado **dentro de cada fold** → sem vazamento de informação
* **Modelos testados:**
  - **KNN:** k ∈ {3, 5, 7, 9}
  - **Árvore de Decisão:** depth=7, split=20, leaf=10 (regularizada)
* **Avaliação no holdout (20%):** Comparação CV vs Holdout

**Resultados Iniciais (Fase 5):**

| Modelo | Acurácia Holdout | Gap Treino-Teste | Recall Churn |
|--------|------------------|------------------|---------------|
| KNN (k=3) | 83.15% | 11.77% ⚠️ | **88.69%** 🏆 |
| Árvore de Decisão | 83.45% | **4.65%** ✅ | 63.69% |

---

### **Fase 6: Avaliação de Negócio e Veredito Final** 💼
* **Análise de custos:**
  - **Falso Negativo (FN):** R\$2.000 de LTV perdido (cliente sai sem incentivo) 🔴
  - **Falso Positivo (FP):** R\$50 de cupom desperdiçado (cliente já ia ficar) 🟡
  - **Razão de custo:** FN é **40x mais caro** que FP → **Prioridade: Minimizar FN!**

* **Avaliação expandida:** Testados 7 modelos/configurações (KNN, Tree, Tree balanced, RandomForest × thresholds)

* **Escolha final:** **KNN (k=3)** 🏆
  - **Melhor recall (88.69%):** Detecta 149 de 168 churns → **Apenas 19 FN**
  - **Economia de R\$84.000** vs Árvore de Decisão (salva 42 clientes a mais)
  - **Score final mais alto** (0.8664 em métrica composta: 40% recall + 35% acurácia + 25% gap)
  - **ROC-AUC de 0.9135:** Excelente poder discriminatório
  - Gap de 11.77% é gerenciável com **monitoramento ativo + retreinamento mensal**

**Métricas do KNN:**
* **Acurácia:** 83.15%
* **Recall (Churn):** **88.69%** ✅ (apenas 19 FN de 168)
* **Precision (Churn):** ~49.5%
* **F1-Score (Churn):** 0.6354
* **ROC-AUC:** 0.9135

### **Comparação dos 3 Finalistas**

| Modelo | Recall | Acurácia | Gap | FN (de 168) | Score |
|--------|--------|----------|-----|-------------|-------|
| **🥇 KNN (k=3)** | **88.69%** | 83.15% | 11.77% ⚠️ | **19** | **0.8664** |
| 🥈 RandomForest @ 0.35 | 80.36% | 78.72% | 5.59% | 33 | 0.8330 |
| 🥉 Tree (d=7) | 63.69% | 83.45% | **4.65%** ✅ | 61 | 0.7852 |

**Por que KNN venceu?**
* ✅ **Cumpre o objetivo:** Minimizar FN (19 vs 61 da Tree = 42 clientes salvos)
* ✅ **ROI superior:** R\$84.000 a mais em receita protegida
* ✅ **Score composto mais alto:** Melhor balanço entre recall, acurácia e gap
* ⚠️ **Mitigação do gap:** Monitoramento diário + retreinamento mensal + A/B test inicial

**Alternativas disponíveis:**
* Se gap degradar em produção → **RandomForest @ 0.35** (recall 80%, gap 5.59%)
* Se precisar interpretabilidade → **Tree @ 0.35** (recall 71%, gap 4.87%)

---

## 📂 Organização do Repositório

```
Projeto-Avaliativo---MODULO-1/
│
├── README.md                          # Você está aqui! 📍
├── E Commerce Dataset.xlsx            # Dataset original (CSV)
└── Projeto Avaliativo - Módulo 1.ipynb # Notebook principal (6 fases)
```

---

## 🚀 Como Utilizar Este Projeto

### **Pré-requisitos**

```bash
Python 3.9+
Jupyter Notebook ou Databricks
```

### **Bibliotecas Necessárias**

```python
pandas
numpy
scikit-learn
imbalanced-learn  # Para SMOTE
seaborn
matplotlib
```

**Instalação:**

```bash
pip install pandas numpy scikit-learn imbalanced-learn seaborn matplotlib
```

---

### **Executando o Projeto**

#### **Opção 1: Jupyter Notebook (Local)**

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/projeto-churn-ecommerce.git
cd projeto-churn-ecommerce

# Execute o notebook
jupyter notebook "Projeto Avaliativo - Módulo 1.ipynb"
```

#### **Opção 2: Databricks (Recomendado)**

1. Faça upload do notebook `.ipynb` para o Databricks Workspace
2. Faça upload do dataset `E Commerce Dataset.xlsx - E Comm.csv`
3. Ajuste o caminho do arquivo na **Célula 2**:
   ```python
   df = pd.read_csv("/dbfs/FileStore/seu-caminho/E Commerce Dataset.xlsx - E Comm.csv")
   ```
4. Execute as células sequencialmente (Fase 1 → Fase 6)

---

### **Sequência de Execução**

**⚠️ IMPORTANTE:** Execute as células **NA ORDEM** (células 1-25), pois:
* Cada fase depende das transformações da fase anterior
* Variáveis são criadas e reutilizadas (`df`, `X_train`, `best_k`, etc.)
* Executar fora de ordem causará erros de variáveis não definidas

**Fases:**
1. **Células 1-4:** EDA e visualizações
2. **Células 5-8:** Tratamento de dados (duplicatas, nulos, outliers)
3. **Células 10-11:** Feature Engineering (análise)
4. **Células 12-16:** Preparação para modelagem (encoding, split)
5. **Células 17-22:** Modelagem e validação (CV, holdout, visualizações)
6. **Células 23-26:** Avaliação expandida com foco em recall, matriz de confusão, veredito final

---

## 🌿 Boas Práticas: Branches e Commits

Este projeto foi desenvolvido seguindo **práticas de entrega contínua e versionamento semântico**, com múltiplas branches e commits organizados:

### **Estrutura de Branches**

```
main (produção)
  ├── develop (desenvolvimento)
  │     ├── feature/eda-visualizations
  │     ├── feature/data-cleaning
  │     ├── feature/feature-engineering
  │     ├── feature/model-training
  │     └── feature/business-evaluation
  └── hotfix/* (correções críticas)
```

### **Padrão de Commits**

Seguimos a convenção **Conventional Commits** para clareza:

```
feat: adiciona análise de correlação com heatmap
```

**Benefícios:**
* ✅ **Histórico limpo e rastreável** de todas as mudanças
* ✅ **Rollback seguro** em caso de bugs
* ✅ **Colaboração eficiente** (mesmo em projeto individual)
* ✅ **Documentação automática** via mensagens de commit

---

## 📊 Resultados e Métricas Finais

### **Modelo Escolhido: KNN (k=3)** 🏆

| Métrica | Valor | Interpretação |
|---------|-------|---------------|
| **Acurácia** | 83.15% | 8 em cada 10 previsões corretas |
| **Recall (Churn)** | **88.69%** ✅ | Detecta 89% dos clientes em risco (apenas 19 FN) |
| **Precision (Churn)** | ~49.5% | Metade dos alertas são churns reais (trade-off aceitável) |
| **F1-Score (Churn)** | 0.6354 | Equilíbrio entre precision e recall |
| **ROC-AUC** | 0.9135 | Excelente poder discriminatório |
| **Gap Treino-Teste** | 11.77% ⚠️ | Overfitting gerenciável com monitoramento |
| **Score Final** | 0.8664 | Melhor em métrica composta (40% recall + 35% acc + 25% gap) |

### **Impacto de Negócio Estimado**

**Cenário: 1.000 clientes no período**
* Clientes em risco real: ~170 (17%)
* **Clientes detectados pelo KNN:** ~151 (88.69% recall) 🎯
* **Receita protegida:** 151 × R\$2.000 = **R\$302.000**
* Custo de intervenção: ~250 cupons × R\$50 = **R\$12.500**
* **ROI da campanha:** (302k - 12.5k) / 12.5k = **2.316%** 🚀

**Comparação com Árvore de Decisão:**
* KNN detecta **42 clientes a mais** (151 vs 109)
* **Economia líquida: R\$84.000** em receita protegida vs Tree

---

## 🎓 Aprendizados e Próximos Passos

### **Lições Aprendidas**

1. ✅ **Pipeline seguro evita data leakage:** SMOTE dentro do CV é essencial
2. ✅ **Feature engineering nem sempre ajuda:** Testar empiricamente é obrigatório
3. ✅ **Métricas de negócio > métricas técnicas:** Entender custos de FN vs FP é crítico
4. ✅ **Trade-offs são inevitáveis:** Gap alto (11.77%) é aceitável quando recall salva R\$84k
5. ✅ **Monitoramento ativo é essencial:** Retreinamento mensal mitiga risco de overfitting
6. ✅ **Score composto orienta decisão:** Balancear recall, acurácia e gap em métrica única

---

## 🛠️ Tecnologias Utilizadas

* **Linguagem:** Python 3.9+
* **Ambiente:** Databricks (Serverless CPU)
* **Bibliotecas principais:**
  - `scikit-learn` (modelagem)
  - `imbalanced-learn` (SMOTE)
  - `pandas` / `numpy` (manipulação de dados)
  - `seaborn` / `matplotlib` (visualização)
* **Versionamento:** Git / GitHub

---

## 📝 Licença

Este projeto foi desenvolvido para fins **educacionais** como parte do curso de Machine Learning e Visão Computacional do SCTEC.

---

## 👤 Autor

**Leonardo Badell**  
📧 leonardobadell@protonmail.com  
🎓 Estudante de Machine Learning e Visão Computacional

---

## ☕ Agradecimentos

Este projeto foi desenvolvido com muito **carinho** e **café** ❤️☕

*"Machine Learning é 10% algoritmos e 90% preparação de dados... e 100% café!"*

---

**⭐ Se este projeto te ajudou, considere deixar uma estrela no repositório!**