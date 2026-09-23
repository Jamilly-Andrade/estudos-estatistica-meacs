# GUIA INTUITIVO — Extração e Análise de Dados
### Prof. Pestana · FGV 2026.2 · Aulas 1–13

> Cada seção explica: **o que é**, **quando usar** e **o mínimo que você precisa saber**.

---

## 📦 PANDAS — a ferramenta de tabelas

**O que é:** Pandas é como uma planilha do Excel, só que em Python. Você carrega dados, filtra, calcula colunas novas e agrupa.

**Quando usar:** SEMPRE. É a base de tudo no curso.

### Ler o arquivo
```python
df = pd.read_csv('arquivo.csv')          # CSV com vírgula
df = pd.read_csv('arquivo.csv', sep=';') # CSV com ponto-e-vírgula (exportações BR)
df = pd.read_excel('arquivo.xlsx')       # Excel
```

### Inspecionar antes de qualquer coisa
```python
df.shape           # quantas linhas e colunas: (200, 8)
df.dtypes          # tipo de cada coluna: object=texto, int/float=número
df.head()          # 5 primeiras linhas
df.isna().sum()    # quantos valores faltando por coluna
df.duplicated().sum() # quantas linhas repetidas
```

### Filtrar linhas
```python
df[df['plays'] > 1000]              # só linhas com plays maior que 1000
df[df['author'] == '@joao']        # só linhas de um autor
df[df['texto'].str.contains('IA', na=False)]  # linhas que contêm 'IA'
```

### Criar coluna nova (sem loop!)
```python
df['taxa_eng'] = (df['likes'] + df['comments']) / (df['plays'] + 1)
df['tamanho'] = df['body'].str.len()
```

### Agrupar e resumir
```python
resumo = df.groupby('author').agg(
    total_posts=('likes', 'count'),
    media_likes=('likes', 'mean'),
    total_shares=('shares', 'sum')
).reset_index()
```
> `groupby` = "divida a tabela por esse grupo". `agg` = "calcule isso dentro de cada grupo". `reset_index` = transforma o índice em coluna normal.

### Remover duplicatas
```python
df = df.drop_duplicates().copy()
```
> Sempre chame `.copy()` depois de filtrar. Evita o aviso chato do Pandas sobre "slice".

---

## #️⃣ HASHTAGS — explodir lista em linhas separadas

**O que é:** Os dados de redes sociais geralmente vêm com várias hashtags numa só célula, separadas por vírgula: `"politica,brasil,eleicoes"`. Você precisa transformar isso em uma linha por hashtag para contar e analisar.

**Quando usar:** Sempre que tiver uma coluna com múltiplos valores separados por vírgula numa célula.

### O padrão completo (sempre nessa ordem)
```python
df_h = df.dropna(subset=['hashtags']).copy()  # 1. remove linhas sem hashtag
df_h = df_h[df_h['hashtags'] != '']           # 2. remove hashtag vazia
df_h['hashtags'] = df_h['hashtags'].str.split(',')  # 3. string → lista
df_h = df_h.explode('hashtags')               # 4. cada item da lista vira uma linha
df_h['hashtags'] = df_h['hashtags'].str.strip()     # 5. remove espaços sobrando
```

Depois disso, você pode fazer `value_counts()` ou `groupby('hashtags').agg(...)` normalmente.

---

## 📊 MATPLOTLIB — gráficos

**O que é:** Biblioteca de gráficos em Python. No curso, usa-se para barras, linhas e dispersão.

**Quando usar:** Para visualizar e comunicar os dados. Sempre inclua: título, rótulos dos eixos e fonte dos dados.

### Estrutura base (sempre igual)
```python
fig, ax = plt.subplots(figsize=(10, 5))  # cria a "tela" do gráfico

# ... desenha o gráfico ...

ax.set_title('Título do gráfico')    # obrigatório
ax.set_xlabel('Rótulo do eixo X')    # obrigatório
ax.set_ylabel('Rótulo do eixo Y')    # obrigatório com unidade
ax.tick_params(axis='x', rotation=45) # gira os rótulos se ficarem sobrepostos
fig.text(0.01, -0.04, 'Fonte: ...', fontsize=8, color='gray')  # fonte dos dados
fig.tight_layout()
plt.show()
```

### Tipos de gráfico
```python
ax.bar(categorias, valores)          # BARRAS — para comparar categorias
ax.plot(x, y, marker='o')           # LINHA — para mostrar evolução no tempo
ax.scatter(x, y, alpha=0.5)         # DISPERSÃO — para ver relação entre 2 variáveis
ax.set_xscale('log')                 # escala logarítmica — quando os valores variam muito
```

> **Regra de ouro:** se é uma categoria (hashtag, autor, tipo), usa **barra**. Se é tempo (data, mês, ano), usa **linha**. Se são dois números contínuos, usa **dispersão**.

---

## 🕸️ REQUESTS + BEAUTIFULSOUP — web scraping básico

**O que é:** `requests` faz a requisição HTTP (como um navegador pedindo a página). `BeautifulSoup` lê o HTML que veio e deixa você navegar pelos elementos como se fossem objetos Python.

**Quando usar:** Para coletar dados de sites **estáticos** — páginas que carregam tudo de uma vez no HTML, sem precisar de JavaScript. Se a página fica carregando após abrir, provavelmente precisa do Playwright.

### Fluxo completo
```python
# 1. Pedir a página
resposta = requests.get(url, headers={'User-Agent': 'MeuScraper/1.0'})

# 2. SEMPRE checar se funcionou antes de continuar
if resposta.status_code == 200:
    resposta.encoding = resposta.apparent_encoding  # corrige caracteres trocados
    html = resposta.text
else:
    print("Erro:", resposta.status_code)

# 3. Parsear o HTML
sopa = BeautifulSoup(html, 'html.parser')

# 4. Extrair os dados
itens = []
for card in sopa.find_all('article', {'class': 'product_pod'}):
    titulo = card.find('h3').find('a')['title']        # pega atributo
    preco  = card.find('p', {'class': 'price_color'}).get_text(strip=True)  # pega texto
    itens.append({'titulo': titulo, 'preco': preco})

# 5. Salvar em CSV
import csv
with open('resultado.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=['titulo', 'preco'])
    writer.writeheader()
    writer.writerows(itens)
```

### Comandos de busca no HTML
| Comando | O que faz |
|---|---|
| `sopa.find('tag', {'class': 'nome'})` | Acha o **primeiro** elemento |
| `sopa.find_all('tag', {'class': 'nome'})` | Acha **todos** os elementos (retorna lista) |
| `elemento.find('tag')` | Busca **dentro** de um elemento já encontrado |
| `elemento.get_text(strip=True)` | Pega o texto visível (sem HTML) |
| `elemento['href']` | Pega o valor de um atributo (href, title, src...) |

> **Boas práticas:** checar `robots.txt` antes de coletar. Adicionar `time.sleep(1)` entre requisições. Identificar a coleta com `User-Agent`.

---

## 🎭 PLAYWRIGHT — páginas dinâmicas (JavaScript)

**O que é:** Um robô que abre um navegador de verdade (Chromium) e interage com a página como um humano — clica, rola, espera carregar. Necessário quando a página usa JavaScript para mostrar os dados.

**Quando usar:** Quando `requests` traz um HTML vazio ou sem os dados que você quer visualizar na tela. Exemplo: feeds do TikTok, Ground News, páginas com "Carregar mais".

**ATENÇÃO:** Não roda dentro do Jupyter. Salve num arquivo `.py` e rode no terminal:
```bash
uv run meu_script.py
```

### Estrutura básica
```python
from playwright.sync_api import sync_playwright
import time, csv

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)   # headless=True = sem janela
    page = browser.new_page()
    page.goto('https://site.com')
    page.wait_for_selector('[data-testid="item"]')  # espera carregar

    # Clicar em "Carregar mais" várias vezes
    for i in range(5):
        botao = page.query_selector('[data-testid="load-more"]')
        if botao:
            botao.scroll_into_view_if_needed()
            botao.click()
            time.sleep(1.5)

    # Extrair elementos
    cards = page.query_selector_all('[data-testid="story-item"]')
    dados = []
    for card in cards:
        titulo = card.text_content().strip()
        link = card.query_selector('a').get_attribute('href')
        dados.append({'titulo': titulo, 'link': link})

    page.screenshot(path='screenshot.png')
    browser.close()
```

### Diferença entre requests e Playwright
| | requests + BS4 | Playwright |
|---|---|---|
| Executa JavaScript? | ❌ Não | ✅ Sim |
| Velocidade | Rápido | Mais lento |
| Roda no Jupyter? | ✅ Sim | ❌ Não (só terminal) |
| Quando usar? | HTML estático | Página dinâmica, feeds, paginação com JS |

---

## 🧹 LIMPEZA E PIPELINE (Aula 10)

**O que é:** Processo de transformar dados brutos (com sujeira, erros de tipo, formatos diferentes) em dados prontos para análise.

**Quando usar:** Sempre depois de coletar dados. Dados brutos quase nunca estão prontos para usar.

### Sequência obrigatória
```python
df = df_bruto.copy()          # 1. NUNCA altere o bruto diretamente

df = df.drop_duplicates()     # 2. remove duplicatas

df = df.dropna(subset=['coluna_obrigatoria'])  # 3. remove linhas sem dado essencial

# 4. Normalizar texto
df['titulo'] = df['titulo'].str.strip()   # remove espaços do início/fim
df['titulo'] = df['titulo'].str.lower()   # tudo minúsculo
df['titulo'] = df['titulo'].str.title()   # Primeira Letra Maiúscula

# 5. Converter tipos
df['preco'] = df['preco'].str.replace('£', '', regex=False)  # limpa símbolo
df['preco'] = pd.to_numeric(df['preco'], errors='coerce')    # texto → número

df['data'] = pd.to_datetime(df['data'], dayfirst=True, errors='coerce')  # texto → data

# 6. Salvar resultado limpo
df.to_csv('dados/processed/resultado_limpo.csv', index=False)
```

> **`errors='coerce'`** = o que não conseguir converter vira `NaN` (valor ausente), em vez de travar tudo.

### Estrutura de pastas do pipeline
```
dados/
  raw/        ← arquivo bruto — NUNCA sobrescreva
  processed/  ← arquivo limpo — salva aqui
```

---

## 🤖 MACHINE LEARNING — REGRESSÃO (Aula 11)

**O que é:** O modelo aprende a prever um **número** a partir de outros dados. Exemplo: prever a taxa de engajamento de um post com base no tamanho da legenda, número de hashtags e horário.

**Quando usar:** Quando a pergunta é "quanto?" — prever um valor numérico contínuo.

### Esqueleto completo
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.tree import DecisionTreeRegressor
from sklearn.metrics import mean_absolute_error, r2_score

# 1. Definir y (alvo) e X (features)
y = df['taxa_engajamento']   # o que quero prever
X = df[['tamanho_legenda', 'num_hashtags', 'hora']]  # com o que vou prever
# NUNCA coloque y (ou algo derivado de y) dentro de X — isso é VAZAMENTO

# 2. Split treino/teste
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42)

# 3. Modelo bobo (referência mínima)
import numpy as np
bobo = np.full(len(y_te), y_tr.mean())
print("MAE bobo:", mean_absolute_error(y_te, bobo).round(4))

# 4. Treinar e prever
modelo = LinearRegression()
modelo.fit(X_tr, y_tr)
prev = modelo.predict(X_te)

# 5. Medir
print("MAE:", mean_absolute_error(y_te, prev).round(4))
print("R²: ", r2_score(y_te, prev).round(4))
```

### Interpretando os resultados
| Métrica | O que é | Bom sinal |
|---|---|---|
| **MAE** | Erro médio na unidade do alvo | Menor que o modelo bobo |
| **R²** | Quanto da variação o modelo explica | Perto de 1.0 |
| **R² = 0** | Igual a chutar a média | Não aprendeu nada |
| **R² < 0** | Pior que chutar a média | Algo errado |

### Quando usar log1p no alvo
Se o histograma do alvo tiver cauda muito longa (curtidas, views, contagens):
```python
y_log = np.log1p(y)        # comprime antes de treinar
# ... treina com y_log ...
prev_original = np.expm1(prev_log)  # desfaz para comparar
```

---

## 🏷️ MACHINE LEARNING — CLASSIFICAÇÃO (Aula 12)

**O que é:** O modelo aprende a prever uma **categoria** — sim ou não, viral ou não viral, churn ou não churn.

**Quando usar:** Quando a pergunta é "qual categoria?" ou "vai acontecer ou não?".

### O rótulo é uma decisão sua
```python
corte = df['plays'].quantile(0.90)       # top 10% de plays
y = (df['plays'] > corte).astype(int)    # 1 = viralizou, 0 = não viralizou
# Documente: "considerei viral todo post acima do percentil 90"
```

### Esqueleto completo
```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, precision_score, recall_score, f1_score

# Split com stratify (mantém proporção de 0s e 1s nos dois lados)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42, stratify=y)

modelo = LogisticRegression(max_iter=1000)
modelo.fit(X_tr, y_tr)
decisao = modelo.predict(X_te)

# Avaliar
print(confusion_matrix(y_te, decisao))
print("Precisão:", precision_score(y_te, decisao, zero_division=0).round(3))
print("Recall:  ", recall_score(y_te, decisao, zero_division=0).round(3))
print("F1:      ", f1_score(y_te, decisao, zero_division=0).round(3))
```

### A matriz de confusão — leia assim
```
          Previu 0    Previu 1
Real 0  [   VN    ,    FP   ]   ← VN = acertou o negativo | FP = falso alarme
Real 1  [   FN    ,    VP   ]   ← FN = perdeu o positivo  | VP = acertou o positivo
```

### Quando cada métrica importa
| Métrica | Quando usar |
|---|---|
| **Acurácia** | Só quando as classes são equilibradas (50/50 aprox.) |
| **Precisão** | Quando falso alarme é caro (ex: recomendar produto errado) |
| **Recall** | Quando não pode perder um positivo (ex: detectar fraude, doença) |
| **F1** | Quando as duas importam — é o equilíbrio entre precisão e recall |

### Mexer no threshold
```python
prob = modelo.predict_proba(X_te)[:, 1]  # probabilidade de ser 1
decisao_03 = (prob >= 0.30).astype(int)  # mais sensível (pega mais positivos)
```
> Threshold padrão é 0.5, mas não é obrigatório. Abaixa para pegar mais positivos (mais recall). Sobe para ser mais certeiro (mais precisão).

---

## 🔵 MACHINE LEARNING — CLUSTERIZAÇÃO KMEANS (Aula 13)

**O que é:** Agrupa dados que se parecem entre si, sem nenhum rótulo prévio. O algoritmo descobre os grupos sozinho.

**Quando usar:** Quando você quer descobrir perfis ou segmentos nos seus dados. Exemplos: perfis de usuário, tipos de post, segmentos de cliente.

### Esqueleto completo
```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

# 1. Selecionar colunas numéricas
X = df[['idade', 'renda', 'gasto', 'num_compras']]

# 2. PADRONIZAR — obrigatório! Sem isso, coluna de maior escala domina tudo
X_pad = StandardScaler().fit_transform(X)

# 3. Escolher k — curva do cotovelo + silhueta
for k in range(2, 8):
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    labels = km.fit_predict(X_pad)
    print(f"k={k} | inércia={km.inertia_:.0f} | silhueta={silhouette_score(X_pad, labels):.3f}")

# 4. Aplicar com o k escolhido
kmeans = KMeans(n_clusters=3, n_init=10, random_state=42)
df['cluster'] = kmeans.fit_predict(X_pad)

# 5. Interpretar — ver o perfil médio de cada grupo
print(df.groupby('cluster')[['idade','renda','gasto','num_compras']].mean().round(2))

# 6. Dar nomes com base no perfil
df['grupo'] = df['cluster'].map({0: 'Jovem econômico', 1: 'Maduro gasta muito', 2: 'Novo cliente'})
```

### Como escolher o k
- **Curva do cotovelo (inércia):** plota a inércia para vários k. O joelho da curva é um bom candidato.
- **Silhueta:** escolhe o k com a silhueta mais alta. Vai de -1 a 1, perto de 1 = grupos bem separados.
- **Regra geral:** prefira o menor k que ainda faz sentido de negócio.

### Silhueta — interpretação rápida
| Valor | O que significa |
|---|---|
| > 0.7 | Grupos muito bem separados |
| 0.5 – 0.7 | Razoavelmente separados |
| 0.3 – 0.5 | Estrutura fraca |
| < 0.3 | Não há estrutura clara |

### Para visualizar (sempre com PCA)
```python
from sklearn.decomposition import PCA
coords = PCA(n_components=2).fit_transform(X_pad)
plt.scatter(coords[:, 0], coords[:, 1], c=df['cluster'], cmap='tab10', alpha=0.6)
plt.title('Clusters (PCA 2D)')
plt.show()
```
> PCA reduz N colunas para 2 só para você conseguir plotar. Não é o cluster em si, é só a visualização.

---

## ⚠️ REGRAS QUE O PROFESSOR SEMPRE COBRA

1. **Sem vazamento:** nunca coloque no `X` uma coluna que é o alvo ou foi calculada a partir dele
2. **Compare com o modelo bobo:** se não bate a média, não aprendeu nada útil
3. **Meça no teste, não no treino:** erro de treino não diz nada
4. **Padronize antes de clusterizar:** `StandardScaler` sempre
5. **Justifique o k:** cotovelo + silhueta, não chute
6. **Coeficiente não é causa:** descreve associação nos dados vistos, nada mais
7. **Documente o rótulo:** o corte que você escolheu muda o resultado — explique o porquê
8. **Pipeline tem estrutura:** `raw/` (bruto intocado) e `processed/` (limpo)

---

## 🆘 EMERGÊNCIA — quando travar

| Situação | O que fazer |
|---|---|
| Não sei o nome das colunas | `df.columns.tolist()` |
| Não sei o tipo da coluna | `df.dtypes` |
| Erro de encoding (caracteres trocados) | `resposta.encoding = resposta.apparent_encoding` |
| Playwright não acha o elemento | Testa o seletor no DevTools do navegador (F12 → Console → `document.querySelector(...)`) |
| requests retorna HTML vazio | A página usa JavaScript → usa Playwright |
| Erro de dimensão no sklearn | Coluna com texto no X → `pd.get_dummies(X, columns=['col'])` |
| Dado faltando antes do sklearn | `X.isna().sum()` → `fillna(0)` ou `dropna()` |
| Playwright não roda no Jupyter | Salva num `.py` e roda no terminal com `uv run script.py` |
| Silhueta deu baixa para todos os k | Os dados talvez não tenham estrutura de cluster com essas features |
