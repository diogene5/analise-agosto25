
# Autoanálise do Notebook: Demanda UPA

Este documento detalha o passo a passo técnico da análise de dados realizada no notebook `demanda.ipynb`, com foco no aprendizado de Python, Pandas e na metodologia de análise.

---

### Estrutura da Análise

A análise seguiu um fluxo lógico, desde a preparação dos dados até a modelagem de cenários e visualização.

**1. Coleta e Consolidação de Dados:**
- **O que foi feito:** Múltiplas planilhas Excel (uma para cada mês) foram carregadas e unificadas em um único DataFrame.
- **Função chave:** `pd.read_excel()` para ler os arquivos e `pd.concat()` para juntá-los. O parâmetro `ignore_index=True` foi crucial para criar um novo índice contínuo.

**2. Limpeza e Preparação (Data Cleaning):**
- **Seleção de Colunas:** De 49 colunas, apenas 20 foram selecionadas para a análise. Isso reduz o ruído e melhora a performance. `df_filtrado = df_original[lista_de_colunas].copy()`.
- **Conversão de Tipos (Dtypes):**
    - **IDs para String:** Colunas como `prontuario_id` foram convertidas de número para texto (`.astype(str)`), pois não são usadas para cálculos matemáticos. Isso evita problemas como o Pandas tentar fazer agregações (soma, média) com elas.
    - **Datas para Datetime:** Colunas de data foram convertidas de texto para o tipo `datetime` (`pd.to_datetime(df['coluna'], dayfirst=True)`). O `dayfirst=True` foi essencial para interpretar o formato brasileiro (DD/MM/AAAA) corretamente. Isso habilita todas as operações de data e hora.
- **Tratamento de Nulos:**
    - **Diagnóstico:** `df['setor'].isna().sum()` foi usado para contar quantos valores nulos existiam na coluna `setor`.
    - **Estratégia:** Foi decidido preencher os nulos com `'CONSULTORIO CLINICO'` (`.fillna('valor')`). Essa foi uma decisão de negócio, baseada na premissa de que a maioria dos atendimentos não especificados era de clínica geral.

**3. Análise Exploratória e Segmentação:**
- **Cálculo de Demanda Geral:** A demanda média diária foi calculada dividindo o total de registros (`len(df)`) pelo número de dias no período. Os dias foram calculados com `(df['data'].max() - df['data'].min()).days + 1`.
- **Filtragem por Lógica de Exclusão:** Para isolar a "Clínica Médica", em vez de selecionar os setores que *são* de clínica, foram listados os que *não são* (pediatria, sutura, etc.). O filtro foi aplicado com o operador `~` (negação) e o método `.isin()`.
    - `df_clinica_medica = df[~df['setor'].isin(setores_excluir)]`
    - Essa é uma técnica robusta, pois novos setores não listados na exclusão são automaticamente incluídos como clínica médica.

**4. Engenharia de Features e Agregação (Tabelas Dinâmicas):**
- **Extração de Componentes de Data:** A hora e o dia da semana foram extraídos da coluna `data_chegada` usando o acessador `.dt`:
    - `df['hora'] = df['data_chegada'].dt.hour`
    - `df['dia_semana'] = df['data_chegada'].dt.day_name()`
- **Criação de Turnos:** Uma função `definir_turno` foi criada e aplicada à coluna de hora com `.apply()` para categorizar cada atendimento em 'Diurno' ou 'Noturno'.
- **Contagem por Categoria:** O método `.value_counts()` foi amplamente utilizado para criar tabelas de frequência (atendimentos por setor, por hora, por turno, por dia da semana). É o equivalente mais simples e direto de uma tabela dinâmica de contagem.

**5. Simulação de Cenários e Análise de Impacto:**
- A análise não parou nos dados históricos. Foi criado um cenário hipotético (redução de um médico) e as métricas de carga de trabalho foram recalculadas. Isso transforma a análise de um relatório do passado para uma ferramenta de decisão para o futuro.

**6. Visualização e Exportação:**
- **Visualização:** Matplotlib e Seaborn foram usados para criar um dashboard com múltiplos gráficos (`plt.subplot`). Isso resume visualmente os achados complexos em uma única imagem.
- **Exportação:** Os resultados (DataFrames) foram salvos em arquivos CSV (`.to_csv()`), permitindo que gestores e colegas não-técnicos possam explorar os dados no Excel.

---

### Flashcards para Aprendizado

**Flashcard 1: `pd.concat()`**
- **Frente:** Para que serve `pd.concat([df1, df2])` e por que usar `ignore_index=True`?
- **Verso:** Serve para "empilhar" ou "juntar" vários DataFrames. `ignore_index=True` descarta os índices originais dos DataFrames e cria um novo índice contínuo (0, 1, 2, ...), o que é essencial para evitar índices duplicados.

**Flashcard 2: `.astype()` vs. `pd.to_datetime()`**
- **Frente:** Qual a diferença entre usar `df['col'].astype(str)` e `pd.to_datetime(df['col'])`?
- **Verso:** `.astype()` é um método genérico para forçar a conversão de um tipo para outro (ex: `int` para `str`). `pd.to_datetime()` é uma função especializada e muito mais poderosa para converter strings (ou números) em datas, conseguindo interpretar dezenas de formatos diferentes (ex: 'DD/MM/AAAA', 'YYYY-MM-DD').

**Flashcard 3: O Acessor `.dt`**
- **Frente:** O que o acessador `.dt` permite fazer em uma coluna de data/hora no Pandas?
- **Verso:** Permite acessar propriedades de data/hora de cada elemento da Série, como `df['data'].dt.hour`, `.dt.day`, `.dt.month`, `.dt.year`, `.dt.day_name()`, `.dt.date`.

**Flashcard 4: Filtragem com `.isin()` e `~`**
- **Frente:** Como você filtra um DataFrame para manter apenas as linhas cujo valor na `coluna_A` NÃO está em uma `lista_de_valores`?
- **Verso:** Usando o operador de negação `~` junto com `.isin()`: `df[~df['coluna_A'].isin(lista_de_valores)]`.

**Flashcard 5: `.value_counts()`**
- **Frente:** Qual é a forma mais rápida de contar as ocorrências de cada valor único em uma coluna do Pandas?
- **Verso:** O método `.value_counts()`. Ex: `df['setor'].value_counts()` retorna uma Série com os setores como índice e a contagem de cada um como valor.

**Flashcard 6: `.apply()`**
- **Frente:** Quando você usaria o método `.apply()` em uma coluna do Pandas?
- **Verso:** Quando você precisa aplicar uma função customizada a cada valor da coluna. Ex: `df['hora'].apply(minha_funcao_de_turno)`, onde `minha_funcao_de_turno` classifica cada hora em 'Diurno' ou 'Noturno'.

**Flashcard 7: `.fillna()`**
- **Frente:** Como substituir todos os valores `NaN` (nulos) em uma coluna por um valor específico, como a string 'Padrão'?
- **Verso:** Usando o método `.fillna()`. Ex: `df['coluna'].fillna('Padrão', inplace=True)`. O `inplace=True` modifica o DataFrame diretamente.
