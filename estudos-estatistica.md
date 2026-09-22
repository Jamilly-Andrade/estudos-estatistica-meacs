# Guia e Estudos MEACS - Métodos Estatísticos (Referência Rápida para Prova)

---

## 1. Tipos de Variáveis
- **Qualitativa Nominal**: Categorias sem ordem (ex: Cor/Raça, Sexo, Estado, Bairro, Se tem internet).
- **Qualitativa Ordinal**: Categorias com hierarquia natural (ex: Escolaridade, Faixa Etária, Satisfação: Baixo/Médio/Alto).
- **Quantitativa Discreta**: Contagem em números inteiros (ex: Número de filhos, quantidade de homicídios).
- **Quantitativa Contínua**: Medição contínua com casas decimais (ex: Renda per capita, temperatura, taxa de mortalidade).

---

## 2. Medidas de Centro e Forma da Distribuição
- **Média**: Soma de tudo / N. **NÃO RESISTENTE** (é arrastada por outliers/valores extremos).
- **Mediana**: Valor do meio (50%). **RESISTENTE** (não se altera com extremos). Representa o valor típico.
- **Média > Mediana**: **Assimétrica à Direita** (positiva). Cauda longa para a direita. Concentração em valores baixos; poucos valores muito altos puxam a média (ex: renda, mortes).
- **Média < Mediana**: **Assimétrica à Esquerda** (negativa). Cauda longa para a esquerda. Concentração em valores altos; poucos valores muito baixos puxam a média.
- **Média ≈ Mediana**: **Aproximadamente Simétrica** (formato de sino/normal). Ex: esperança de vida.

---

## 3. Dispersão, Quartis e Valores Atípicos (Outliers)
- **Desvio Padrão**: Dispersão em torno da média. Não resistente.
- **Amplitude Interquartil (AIQ)**: $AIQ = Q3 - Q1$. Dispersão dos 50% centrais. Medida resistente.
- **Regra 1.5 × AIQ para Outliers**:
  - Limite Inferior = $Q1 - 1.5 \times AIQ$
  - Limite Superior = $Q3 + 1.5 \times AIQ$
  - Qualquer ponto fora desses limites é **valor atípico (outlier)**.

---

## 4. Correlação Linear de Pearson ($r$)
- Varia de **-1 a +1**:
  - Próximo de **+1**: Relação linear positiva forte (se X sobe, Y sobe).
  - Próximo de **-1**: Relação linear negativa forte (se X sobe, Y desce).
  - Próximo de **0**: Ausência de relação linear.
  - $|r| < 0.3$: Fraca | $0.3 \le |r| \le 0.7$: Moderada | $|r| > 0.7$: Forte.
- **Importante**: Pearson é **não resistente** a outliers. Um único ponto extremo (ex: Distrito Federal em renda) pode distorcer $r$.
- **Datasaurus Dozen**: Pearson só mede relação linear. Olhe sempre o gráfico de dispersão antes de concluir!

---

## 5. Configuração Inicial no Colab (Sempre rodar primeiro)
```python
import pandas as pd
import altair as alt

# Permite que o Altair plote bases com mais de 5000 linhas
alt.data_transformers.disable_max_rows()
pd.set_option('display.max_columns', None)
```

---

## 6. Código: Gráfico de Barras e Setores (Qualitativa Univariada)
```python
# Gráfico de barras ordenado decrescente
alt.Chart(df).mark_bar().encode(
    x=alt.X('COLUNA_CATEGORICA', sort='-y', title='Categoria'),
    y=alt.Y('count()', title='Contagem'),
    tooltip=['COLUNA_CATEGORICA', 'count()']
).properties(width=400, height=300)

# Gráfico de setores (pizza) - Somente quando categorias somam 100% de um todo
alt.Chart(df).mark_arc().encode(
    theta='count()',
    color='COLUNA_CATEGORICA',
    tooltip=['COLUNA_CATEGORICA', 'count()']
)
```

---

## 7. Código: Análise Bivariada Categórica (Tabela Cruzada e Barras 100%)
```python
# 1. Tabela de frequências absolutas (wide)
tabela_dupla = df.groupby(['CATEGORIA_1', 'CATEGORIA_2']).size().unstack(1)
tabela_dupla.loc['Total', :] = tabela_dupla.sum(axis=0)
tabela_dupla.loc[:, 'Total'] = tabela_dupla.sum(axis=1)
tabela_dupla

# 2. Distribuição condicional percentual (%)
pivot = pd.pivot_table(df, index=['CATEGORIA_1', 'CATEGORIA_2'], aggfunc='size')
dist_condicional = (pivot / pivot.groupby(level=0).transform('sum')) * 100
dist_condicional

# 3. Gráfico de Barras Segmentadas 100% (Normalizado)
alt.Chart(pivot.reset_index().rename(columns={0: 'contagem'})).mark_bar().encode(
    x=alt.X('contagem', stack='normalize', title='Proporção'),
    y=alt.Y('CATEGORIA_1:N', title='Categoria 1'),
    color=alt.Color('CATEGORIA_2:N', title='Categoria 2')
).properties(width=500, height=300)
```

---

## 8. Código: Séries Temporais (Linha e Agregação Mensal)
```python
# Converter coluna para datetime
df['DATA'] = pd.to_datetime(df['DATA'])

# Gráfico de linha simples
alt.Chart(df).mark_line().encode(
    x='DATA:T',
    y='VARIAVEL_NUMERICA:Q',
    tooltip=['DATA:T', 'VARIAVEL_NUMERICA:Q']
).properties(width=800, height=350)

# Agregação mensal (Média por mês via resample)
df_idx = df.set_index('DATA')
media_mensal = df_idx.resample('M')['VARIAVEL_NUMERICA'].mean().reset_index()

# Gráfico da série agregada mensal
alt.Chart(media_mensal).mark_line().encode(
    x='DATA:T',
    y='VARIAVEL_NUMERICA:Q'
).properties(width=800, height=350)
```

---

## 9. Código: Histograma e `.describe()` (Quantitativa Univariada)
```python
# Estatísticas descritivas completas (Média, DP, Min, Q1, Mediana, Q3, Max)
df['VARIAVEL'].describe()

# Amplitude Interquartil (AIQ)
q1 = df['VARIAVEL'].quantile(0.25)
q3 = df['VARIAVEL'].quantile(0.75)
aiq = q3 - q1
print(f"Q1: {q1}, Q3: {q3}, AIQ: {aiq}")

# Histograma no Altair
alt.Chart(df).mark_bar().encode(
    x=alt.X('VARIAVEL:Q', bin=alt.Bin(step=100)), # ajuste step ou use bin=True
    y=alt.Y('count()', title='Frequência')
).properties(width=500, height=300)
```

---

## 10. Código: Dispersão e Correlação de Pearson (Quantitativa Bivariada)
```python
# Gráfico de Dispersão (Scatter Plot)
alt.Chart(df).mark_circle(size=60).encode(
    x=alt.X('VARIAVEL_X:Q', title='Variável Explicativa'),
    y=alt.Y('VARIAVEL_Y:Q', title='Variável Resposta'),
    tooltip=['IDENTIFICADOR', 'VARIAVEL_X', 'VARIAVEL_Y']
).properties(width=500, height=400)

# Correlação de Pearson
df[['VARIAVEL_X', 'VARIAVEL_Y']].corr(numeric_only=True)

# Correlação removendo um Outlier (ex: DF)
df[df['IDENTIFICADOR'] != 'Distrito Federal'][['VARIAVEL_X', 'VARIAVEL_Y']].corr(numeric_only=True)
```

---

## 11. Código: Boxplots Comparativos Lado a Lado (Grupos / Desagregações)
```python
# Transformar dados para formato longo (melt)
melted = df.melt(
    id_vars=['IDENTIFICADOR'],
    value_vars=['COLUNA_GRUPO_1', 'COLUNA_GRUPO_2'],
    var_name='Grupo',
    value_name='Valor'
)

# Boxplot com regra 1.5xAIQ (extent=1.5)
alt.Chart(melted).mark_boxplot(extent=1.5, size=40).encode(
    x=alt.X('Valor:Q', title='Medida'),
    y=alt.Y('Grupo:N', title='Grupo'),
    tooltip=['IDENTIFICADOR', 'Valor']
).properties(width=600, height=250)
```

---

## 12. O que alterar nos códigos na hora da prova
- **Nome do DataFrame**: se chamou seu dataframe de `dados`, troque `df` por `dados`.
- **Nome das Colunas**: rode `print(df.columns.tolist())` e copie o nome exato da coluna da sua prova.
- **Sintaxe do Altair**: NUNCA altere comandos como `alt.Chart`, `mark_bar()`, `mark_circle()`, `mark_boxplot()`, `stack="normalize"`, `count()`. Eles são a sintaxe da biblioteca.

---

## 13. Casos Clássicos das Aulas de MEACS
- **Assassino Harold Shipman**: Distribuição bimodal anômala no horário das mortes (pico à tarde nas visitas domiciliares, vs médicos comuns com mortes espalhadas pelo dia). Vítimas em sua maioria mulheres idosas (75+ anos) via tabela cruzada.
- **Bebês de Bristol**: Mortalidade em cirurgias cardíacas pediátricas. Gráfico de dispersão provou que o hospital de Bristol era um outlier negativo severo em relação à taxa de sobrevivência nacional.
- **Parceiros Sexuais (Natsal)**: Homens relataram média de 15 parceiras e mulheres 6. A média masculina foi inflada por poucos homens relatando centenas/milhares (não resistente). A mediana foi idêntica entre os sexos (resistente).
- **Indicadores Per Capita**: Dividir pelo número de habitantes equaliza comparações territoriais entre cidades/países de populações muito desiguais.

---

## 14. Roteiro de Interpretação para a Prova (Textos Padrão)
- **Ao interpretar histograma / variável univariada:**
  1. Falar da **forma**: simétrica, assimétrica à direita ou à esquerda.
  2. Comparar **média e mediana**: explicar se a média é maior ou menor e por que (valores altos arrastam a média).
  3. Falar da **dispersão**: amplitude total (Min até Max) e amplitude interquartil (onde estão os 50% centrais dos dados). Citar os casos extremos.
- **Ao interpretar correlação / gráfico de dispersão:**
  1. Direção: positiva (conforme X cresce, Y cresce) ou negativa (conforme X cresce, Y decresce).
  2. Intensidade do coeficiente de Pearson ($r$): fraco, moderado ou forte.
  3. Outlier: apontar se há algum ponto isolado e dizer se ele enfraquece ou reforça a correlação ao ser removido.
- **Ao interpretar boxplots comparativos:**
  1. Comparar as **medianas** (linhas centrais): qual grupo tem valor típico mais alto.
  2. Comparar as **caixas (AIQ)**: qual grupo tem maior variabilidade/dispersão central.
  3. Apontar **valores atípicos** (pontos além dos bigodes de 1.5xAIQ) e quem são.
