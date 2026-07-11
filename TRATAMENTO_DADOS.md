# 🧹 Tratamento e Limpeza de Dados

## 📋 Resumo Executivo

Dataset de E-Commerce preparado para modelagem preditiva de churn através de remoção de duplicatas, imputação de valores nulos e análise de outliers.

**Resultado:** Dataset limpo com 5.074 linhas, 19 colunas, 0 valores nulos, 0 duplicatas.

---

## 🔄 Etapa 1: Remoção de Duplicatas

**Localização:** Cell 6

**Diagnóstico:** 556 linhas duplicadas (9.9% do dataset)

**Método:**
```python
df = df.drop_duplicates()
```

**Resultado:** 5.630 → 5.074 linhas (-556)

**Justificativa:** Eliminar data leakage e preservar independência entre observações.

---

## 💧 Etapa 2: Tratamento de Valores Nulos

**Localização:** Cell 7

### Valores Nulos Identificados:

| Coluna | Nulos | % |
|--------|-------|---|
| Tenure | 231 | 4.6% |
| WarehouseToHome | 221 | 4.4% |
| HourSpendOnApp | 230 | 4.5% |
| DaySinceLastOrder | 288 | 5.7% |
| OrderAmountHikeFromlastYear | 252 | 5.0% |
| CouponUsed | 210 | 4.1% |
| OrderCount | 243 | 4.8% |

### Estratégias Aplicadas:

**MEDIANA** (5 variáveis - distribuições assimétricas ou discretas):
```python
df['Tenure'] = df['Tenure'].fillna(df['Tenure'].median())
df['DaySinceLastOrder'] = df['DaySinceLastOrder'].fillna(df['DaySinceLastOrder'].median())
df['WarehouseToHome'] = df['WarehouseToHome'].fillna(df['WarehouseToHome'].median())
df['CouponUsed'] = df['CouponUsed'].fillna(df['CouponUsed'].median())
df['OrderCount'] = df['OrderCount'].fillna(df['OrderCount'].median())
```

**MÉDIA** (2 variáveis - distribuições simétricas):
```python
df['HourSpendOnApp'] = df['HourSpendOnApp'].fillna(df['HourSpendOnApp'].mean())
df['OrderAmountHikeFromlastYear'] = df['OrderAmountHikeFromlastYear'].fillna(df['OrderAmountHikeFromlastYear'].mean())
```

**Critério de Escolha:**
* **Mediana:** Robusta contra outliers, ideal para distribuições assimétricas e variáveis discretas
* **Média:** Apropriada para distribuições simétricas sem outliers

---

## ⚠️ Etapa 3: Decisão sobre Outliers

**Localização:** Cell 8 (Boxplots)

**Decisão:** **MANTER TODOS OS OUTLIERS**

**Justificativa:**
* Outliers representam comportamentos reais de clientes (tenure baixo = alto risco, cashback alto = VIP)
* Valores extremos são os mais preditivos de churn
* Random Forest/XGBoost são resistentes a outliers
* Remover = perder informação crítica

**Técnicas NÃO utilizadas:** Z-Score, IQR Method

---

## 📊 Resultado Final

| Métrica | Valor |
|---------|-------|
| Registros | 5.074 |
| Colunas | 19 |
| Valores nulos | 0 ✅ |
| Duplicatas | 0 ✅ |
| Status | Pronto para ML |

---

## 🎯 Próximas Etapas

1. Feature Engineering
2. Balanceamento de classes (SMOTE)
3. Normalização (StandardScaler)
4. Treinamento de modelos (Random Forest, XGBoost)

---

**📅 Data:** Julho 2026  
**✅ Status:** Dataset limpo e validado
