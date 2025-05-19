# Relatório do Trabalho 01: Análise de Dados em Grafos

- **Aluno:** José Freitas Alves Neto
- **Matrícula:** 2519203

## 1. Introdução e Objetivos

Este trabalho tem como objetivo principal a análise de um grafo de coautoria científica, construído a partir de publicações relacionadas ao PPGIA UNIFOR. A análise envolve a coleta e preparação dos dados, a construção do grafo, o cálculo de diversas medidas de centralidade e a interpretação dos resultados para identificar os pesquisadores mais influentes ou centrais na rede de colaboração.

## 2. Metodologia

O trabalho foi desenvolvido em um ambiente Jupyter Notebook utilizando a linguagem Python e bibliotecas especializadas em análise de grafos e manipulação de dados.

### 2.1. Coleta e Preparação dos Dados (Passo 01)

#### 2.1.1. Fonte dos Dados
Os dados de publicações científicas foram coletados a partir de uma busca no **Google Scholar** pelo termo "ppgia unifor". Foram extraídos os nomes dos autores dos primeiros 185 artigos encontrados.

#### 2.1.2. Ponto de Atenção sobre a Coleta
É crucial destacar algumas limitações da coleta:
- A extração focou na página inicial de resultados, não aprofundando em cada link de artigo, o que pode não capturar todos os autores de cada publicação.
- A amostra de 185 artigos não representa a totalidade da produção científica do PPGIA UNIFOR, introduzindo um possível viés no estudo.
- Nem todos os professores do programa podem estar presentes no grafo devido a essa amostragem.
- A análise se baseia estritamente no grafo gerado a partir desta amostra, sem intenção de exaltar ou depreciar a imagem de qualquer pesquisador.

#### 2.1.3. Processamento e Criação do Grafo
1.  **Carregamento Inicial**: Os dados foram inicialmente carregados a partir de um arquivo `grafo_coautoria_ppgia_unifor.graphml`.
2.  **Limpeza Preliminar**:
    *   Um auto-loop (aresta de um nó para ele mesmo) foi removido do nó `YR Serpa`.
    *   Nós que representavam anos (ex: "1900" a "2025") foram removidos, pois provavelmente foram interpretados erroneamente como autores durante a coleta.
3.  **Normalização e Agrupamento de Nomes**:
    *   O grafo foi convertido para uma matriz de adjacência utilizando `pandas`.
    *   Os nomes dos autores (índices e colunas da matriz) foram convertidos para letras maiúsculas.
    *   Foi utilizada a função `normalizar_e_agrupar_com_cedilha` para:
        *   Remover acentos (ex: "MENDONÇA" -> "MENDONCA") usando `unicodedata`.
        *   Substituir 'ç' por 'c'.
        *   Agrupar (somar as colaborações) de autores cujos nomes se tornaram idênticos após a normalização.
    *   Um tratamento adicional foi realizado em um loop para nomes no formato "NOME_AUTOR - NOME_REVISTA...", extraindo apenas o nome do autor e agregando suas colaborações.
    *   Duplicatas de linhas/colunas (nomes de autores) resultantes da normalização foram tratadas, mantendo a primeira ocorrência e somando os valores, com avisos caso os dados fossem conflitantes.
4.  **Reconstrução do Grafo**: A matriz de adjacência, agora limpa e normalizada, foi usada para reconstruir o grafo final de colaboração utilizando `networkx`.

Após o pré-processamento, o grafo resultante continha **239 nós (autores)** e **267 arestas (colaborações)**.

### 2.2. Funções Auxiliares e Bibliotecas (Passo 02 - Implementação)

#### 2.2.1. Bibliotecas Utilizadas
-   `networkx (nx)`: Para criação, manipulação e estudo da estrutura de grafos.
-   `matplotlib.pyplot (plt)` e `matplotlib.cm (cm)`: Para visualização dos grafos e uso de mapas de cores.
-   `numpy (np)`: Para operações numéricas.
-   `pandas (pd)`: Para manipulação de dados, especialmente a matriz de adjacência.
-   `unicodedata`: Para normalização de texto (remoção de acentos).
-   `networkx.algorithms.community`: Para detecção de comunidades em grafos.

#### 2.2.2. Funções de Normalização e Visualização Definidas

1.  **`remover_acentos(texto)`**:
    *   **Objetivo**: Normalizar strings, removendo acentuações e substituindo 'ç' por 'c'.
    *   **Importância**: Padroniza nomes de autores para evitar duplicidade.

2.  **`normalizar_e_agrupar_com_cedilha(data)`**:
    *   **Objetivo**: Aplica `remover_acentos` aos nomes em um DataFrame (matriz de adjacência) e agrupa entradas idênticas após normalização.
    *   **Importância**: Garante consistência dos nós do grafo.

3.  **`plotar_grafo_legivel(G, title, layout_type, ...)`**:
    *   **Objetivo**: Visualizar um grafo completo, com detecção e coloração de comunidades, e ajustes para legibilidade (espaçamento, tamanho de nó por grau, controle de rótulos).
    *   **Parâmetros Notáveis**:
        *   `layout_type`: Escolha de algoritmos de layout (`spring`, `kamada_kawai`, `sfdp`, etc.).
        *   `k_spring`, `spring_layout_scale`, `graphviz_layout_args`: Controle de espaçamento.
        *   `show_labels`, `label_font_size`, `label_bbox_alpha`: Controle de rótulos.
    *   **Funcionalidades**: Detecção de comunidades (`greedy_modularity_communities`), coloração de nós por comunidade, tamanho de nó dinâmico.

4.  **`plotar_componente_conectado_com_destaque(G_original, target_node_name, ...)`**:
    *   **Objetivo**: Isolar e visualizar o componente conectado de um nó específico, destacando o nó alvo e seus vizinhos diretos.
    *   **Funcionalidades**: Aplica-se a um subgrafo, detecta comunidades no subgrafo, customiza cores/tamanhos para nó alvo, vizinhos, e demais elementos.

5.  **`plotar_componente_conectado_simples(G_original, target_node_name, ...)`**:
    *   **Objetivo**: Visualizar o componente conectado de um nó de forma simples, sem destaque especial, mas com nós coloridos por comunidades do subgrafo.

### 2.3. Cálculo das Medidas de Centralidade (Passo 02 - Aplicação)

Foram calculadas seis medidas de centralidade para cada pesquisador (nó) no grafo de colaboração, utilizando as funções disponíveis na biblioteca `NetworkX`.

## 3. Resultados das Medidas de Centralidade

A seguir, são apresentados os top 15 pesquisadores para cada medida de centralidade calculada.

### 3.1. Degree Centrality (Centralidade de Grau)
-   **O que mede**: Quantas conexões diretas (colaborações) um autor tem.
-   **Cálculo**: `nx.degree_centrality(G)` (normalizada) e `G.degree()` (não normalizada).
-   **Top 15 Nós (Não Normalizado / Contagem Bruta de Conexões):**
    ```
    Rank  | Nó                 | Degree Centrality (Contagem)
    -------------------------------------------------------
    1     | MAF RODRIGUES      | 26.0000
    2     | JJPC RODRIGUES     | 13.0000
    3     | YR SERPA           | 10.0000
    4     | A SAMPAIO          | 10.0000
    5     | NC MENDONCA        | 10.0000
    6     | N MENDONCA         | 8.0000
    7     | A BRAYNER          | 8.0000
    8     | RH FILHO           | 7.0000
    9     | C CAMINHA          | 7.0000
    10    | JVV SOBRAL         | 7.0000
    11    | V FURTADO          | 6.0000
    12    | PHM MAIA           | 6.0000
    13    | VHC DE ALBUQUERQUE | 6.0000
    14    | CM ADERALDO        | 5.0000
    15    | HP PONTES          | 5.0000
    ```

### 3.2. Closeness Centrality (Centralidade de Proximidade)
-   **O que mede**: Quão perto (em termos de caminhos mais curtos) um nó está de todos os outros nós alcançáveis no seu componente conectado.
-   **Cálculo**: `nx.closeness_centrality(G)`.
-   **Top 15 Nós:**
    ```
    Rank  | Nó                 | Closeness Centrality
    -------------------------------------------------------
    1     | MAF RODRIGUES      | 0.1361
    2     | NC MENDONCA        | 0.1212
    3     | YR SERPA           | 0.1000
    4     | HP PONTES          | 0.0995
    5     | DV MACEDO          | 0.0985
    6     | VM CARVALHO        | 0.0976
    7     | JH FONTELES        | 0.0971
    8     | PPM NETO           | 0.0966
    9     | DV DE MACEDO       | 0.0966
    10    | RG BARBOSA         | 0.0962
    11    | A SAMPAIO          | 0.0957
    12    | PATTERNS AND …     | 0.0953  (Nota: Este parece ser um nome de entidade truncado/agrupado)
    13    | EF DUTRA           | 0.0944
    14    | TRC DE OLIVEIRA    | 0.0944
    15    | ES SILVA           | 0.0939
    ```

### 3.3. Betweenness Centrality (Centralidade de Intermediação)
-   **O que mede**: Quantos caminhos mais curtos entre pares de nós distintos passam por esse nó. Indica o controle do nó sobre o fluxo de informação.
-   **Cálculo**: `nx.betweenness_centrality(G)`.
-   **Top 15 Nós:**
    ```
    Rank  | Nó                 | Betweenness Centrality
    -------------------------------------------------------
    1     | MAF RODRIGUES      | 0.0667
    2     | NC MENDONCA        | 0.0456
    3     | A SAMPAIO          | 0.0173
    4     | HP PONTES          | 0.0134
    5     | VM CARVALHO        | 0.0113
    6     | N MENDONCA         | 0.0094
    7     | JBF DUARTE         | 0.0092
    8     | ES FURTADO         | 0.0092
    9     | YR SERPA           | 0.0087
    10    | CM ADERALDO        | 0.0071
    11    | APF BASTOS         | 0.0070
    12    | M CUNHA            | 0.0061
    13    | JJPC RODRIGUES     | 0.0052
    14    | PHM MAIA           | 0.0048
    15    | E FURTADO          | 0.0048
    ```

### 3.4. Eigenvector Centrality (Centralidade de Autovetor)
-   **O que mede**: Importância de um nó levando em conta a importância dos seus vizinhos. Nós conectados a nós de alta importância têm maior centralidade de autovetor.
-   **Cálculo**: `nx.eigenvector_centrality(G, max_iter=1000)`.
-   **Top 15 Nós:**
    ```
    Rank  | Nó                 | Eigenvector Centrality
    -------------------------------------------------------
    1     | MAF RODRIGUES      | 0.6169
    2     | YR SERPA           | 0.2908
    3     | JH FONTELES        | 0.2121
    4     | RG BARBOSA         | 0.1872
    5     | DV MACEDO          | 0.1803
    6     | NC MENDONCA        | 0.1800
    7     | HP PONTES          | 0.1708
    8     | PPM NETO           | 0.1693
    9     | DV DE MACEDO       | 0.1532
    10    | PATTERNS AND …     | 0.1517
    11    | EF DUTRA           | 0.1464
    12    | ES SILVA           | 0.1385
    13    | TRC DE OLIVEIRA    | 0.1312
    14    | EBS LUSTOSA        | 0.1287
    15    | PF LINHARES        | 0.1276
    ```

### 3.5. Katz Centrality
-   **O que mede**: Importância de um nó levando em conta todas as conexões (diretas e indiretas), com um fator de atenuação para conexões mais distantes.
-   **Cálculo**: `nx.katz_centrality(G, beta=1.0, normalized=True)`.
-   **Top 15 Nós:**
    ```
    Rank  | Nó                 | Katz Centrality
    -------------------------------------------------------
    1     | MAF RODRIGUES      | 0.2618
    2     | JJPC RODRIGUES     | 0.1479
    3     | NC MENDONCA        | 0.1430
    4     | YR SERPA           | 0.1388
    5     | A SAMPAIO          | 0.1245
    6     | JVV SOBRAL         | 0.1071
    7     | RH FILHO           | 0.1053
    8     | JH FONTELES        | 0.1050
    9     | N MENDONCA         | 0.1042
    10    | A BRAYNER          | 0.1022
    11    | HP PONTES          | 0.0995
    12    | RAL RABELO         | 0.0956
    13    | M CUNHA            | 0.0956
    14    | HS ARAUJO          | 0.0943
    15    | PPM NETO           | 0.0942
    ```

## 4. Análise e Comparação dos Resultados

**Nota Importante:** As conclusões são baseadas na amostra de dados (185 artigos) e podem não refletir a totalidade da rede de colaboração do PPGIA UNIFOR.

-   **Degree Centrality**: **MAF RODRIGUES** destaca-se com o maior número de conexões diretas (26), indicando um papel ativo e central em colaborações dentro desta amostra.
-   **Closeness Centrality**: **MAF RODRIGUES** novamente lidera, sugerindo que, em média, está mais próxima (menor caminho) de todos os outros autores na rede, facilitando a disseminação de informações. **NC MENDONCA** também aparece com alta proximidade.
-   **Betweenness Centrality**: **MAF RODRIGUES** possui a maior centralidade de intermediação, atuando como uma "ponte" crucial nos caminhos mais curtos entre outros pares de nós. Isso reforça sua importância estrutural para o fluxo de informação e conexão entre diferentes grupos de pesquisadores. **NC MENDONCA** também figura com alta intermediação.
-   **Eigenvector Centrality**: **MAF RODRIGUES** é a mais influente, indicando que está conectada a outros autores que também são altamente conectados e influentes, amplificando sua relevância. **YR SERPA** e **JH FONTELES** seguem com valores expressivos.
-   **Katz Centrality**: **MAF RODRIGUES** lidera, confirmando sua posição central e estratégica, considerando tanto conexões diretas quanto indiretas com um fator de atenuação. **JJPC RODRIGUES** e **NC MENDONCA** também apresentam alta centralidade Katz.

De forma geral, a autora **MAF RODRIGUES** consistentemente aparece no topo ou entre os primeiros em todas as medidas de centralidade analisadas, sugerindo uma posição de grande destaque, influência e conectividade na rede de coautorias estudada a partir da amostra coletada. Autores como **NC MENDONCA**, **YR SERPA** e **JJPC RODRIGUES** também demonstram papéis importantes em diferentes aspectos da centralidade da rede.

## 5. Visualizações do Grafo

### 5.1. Grafo do PPGIA UNIFOR (Completo)
O grafo completo, após o pré-processamento, foi visualizado utilizando a função `plotar_grafo_legivel`.
-   A saída do código informou: "Plotando grafo 'Grafo do PPGIA UNIFOR' com 239 nós e 267 arestas."
-   O layout `spring` foi utilizado, e os nós foram coloridos de acordo com as comunidades detectadas.

*(Neste ponto, a imagem do grafo completo seria inserida se estivéssemos em um ambiente que a renderizasse.)*

### 5.2. Componente Conectado de Destaque: MAF RODRIGUES
Para analisar mais de perto a rede em torno da autora mais central identificada, foi plotado o componente conectado de **MAF RODRIGUES** usando `plotar_componente_conectado_com_destaque`.
-   A saída do código informou: "Plotando o componente conectado de 'MAF RODRIGUES' com 70 nós e 108 arestas."
-   Nesta visualização, o nó 'MAF RODRIGUES' e suas conexões diretas são destacados, e os demais nós do seu componente são coloridos por comunidade.

*(Neste ponto, a imagem do componente conectado de MAF RODRIGUES seria inserida.)*

## 6. Conclusão

A análise das medidas de centralidade no grafo de coautoria, construído a partir de uma amostra de publicações do Google Scholar, revelou padrões interessantes de colaboração. A pesquisadora **MAF RODRIGUES** emergiu consistentemente como uma figura central em todas as métricas, indicando um papel significativo tanto em termos de volume de colaborações diretas quanto em sua posição estratégica na rede. Outros pesquisadores também mostraram importância em métricas específicas.

É fundamental reiterar que estes resultados são específicos da amostra de dados utilizada. Uma análise mais abrangente, com um dataset mais completo, poderia refinar ou alterar algumas dessas conclusões. O trabalho demonstrou com sucesso a aplicação de técnicas de análise de grafos para entender redes de colaboração científica.
