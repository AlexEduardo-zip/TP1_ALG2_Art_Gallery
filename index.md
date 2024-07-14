---
layout: default
title: TP1 - Algoritmos 2 - 2024/01
---

## Trabalho Prático 1 - Algoritmos 2

### Nome: Alex Eduardo Alves dos Santos --- Matrícula: 2021032145
### Nome: Paula D'Agostini Alvares Maciel --- Matrícula: 2022422524

Este projeto implementa a triangulação de polígonos pelo método de corte de orelhas e a coloração de um grafo de triângulos para obter uma coloração 3-cromática. Utiliza a biblioteca Plotly para gerar gráficos interativos que ilustram cada passo do algoritmo.

### Classe Ponto

A classe `Ponto` representa um ponto em um plano cartesiano com coordenadas x e y. Ela inclui métodos para somar, subtrair, calcular o produto vetorial e verificar igualdade entre pontos, além de métodos para exibir o ponto como string.

```python
class Ponto:
    """ 
    Classe para trabalhar com pontos. 
    Representa um ponto em um plano cartesiano com coordenadas x e y.
    
    """
    def __init__(self, x, y):
        """
        Inicializa um ponto com coordenadas x e y.
        """
        self.x = x
        self.y = y

    def __add__(self, outro):
        """
        Soma as coordenadas de dois pontos.
        outro: outro ponto
        """
        return Ponto(self.x + outro.x, self.y + outro.y)

    def __sub__(self, outro):
        """
        Subtrai as coordenadas de dois pontos.
        """
        return Ponto(self.x - outro.x, self.y - outro.y)

    def __mul__(self, outro):
        """
        Calcula o produto vetorial (determinante) de dois pontos.
        """
        return self.x * outro.y - outro.x * self.y
    
    def __str__(self):
        """
        Retorna uma string representando o ponto com duas casas decimais.
        """
        return "(%.2f, %.2f) " % (self.x, self.y)

    def __eq__(self, outro):
        """
        Verifica se dois pontos são iguais (mesmas coordenadas).
        """
        return self.x == outro.x and self.y == outro.y

    def __repr__(self):
        """
        Retorna uma string representando o ponto com duas casas decimais (para uso em listas, por exemplo).
        """
        return "(%.2f, %.2f) " % (self.x, self.y)

    def __hash__(self):
        """
        Retorna o valor hash do ponto, permitindo que ele seja usado em conjuntos e como chave em dicionários.
        """
        return hash((self.x, self.y))
```

# Funções Auxiliares
## converter_para_lista_de_pontos
Converte duas listas de valores x e y em uma lista de objetos Ponto.

```python
def converter_para_lista_de_pontos(lista_x, lista_y):
    """ 
    Converte duas listas de valores x e valores y em uma lista de pontos.
    """
    pontos = []
    for x, y in zip(lista_x, lista_y):
        pontos.append(Ponto(x, y))
    return pontos
```

## dentro_do_triangulo
Verifica se um ponto está dentro de um triângulo formado por três vértices.
```python
def dentro_do_triangulo(p, v1, v2, v3):
    """ 
    Verifica se o ponto p está dentro do triângulo formado pelos vértices v1, v2 e v3.
    """
    return (v1 - p) * (v2 - v1) <= 0 and (v2 - p) * (v3 - v2) <= 0 and (v3 - p) * (v1 - v3) <= 0
```
## anti_horario
Verifica se uma lista de pontos está em ordem anti-horári
```python
def anti_horario(pontos):
    """ 
    Verifica se a lista de pontos está em ordem anti-horária.
    """
    y_minimo = pontos[0][1]
    index_minimo = 0

    for i, e in enumerate(pontos):
        if e[1] < y_minimo:
            y_minimo = e[1]
            index_minimo = i
        elif e[1] == y_minimo:
            if e[0] > pontos[index_minimo][0]:
                y_minimo = e[1]
                index_minimo = i

    a = pontos[index_minimo]
    a = Ponto(a[0], a[1])
    b = pontos[(index_minimo - 1) % len(pontos)]
    b = Ponto(b[0], b[1])
    c = pontos[(index_minimo + 1) % len(pontos)]
    c = Ponto(c[0], c[1])

    #  produto vetorial para determinar a orientação
    produto = (a - b) * (c - a)
    return produto > 0
```
# Classe CorteDeOrelhas
A classe 'CorteDeOrelhas' implementa o algoritmo de triangulação de polígonos pelo método de corte de orelhas.
```python
class CorteDeOrelhas:
    """ 
    Computa a triangulação de polígonos pelo método de corte de orelhas.
    """
    def __init__(self, xs, ys):
        self._pontos_iniciais = converter_para_lista_de_pontos(xs, ys)
        self.pontos = self._pontos_iniciais.copy()
        self._solucao = []
        self.indice = 0
        self.xs = xs
        self.ys = ys

        self.triangulos = []
        while len(self.pontos) > 3:
            self.cortedeorelha_aux(
                self.indice % len(self.pontos)
            )
        self.triangulos.append([p for p in self.pontos])

    def ponta_de_orelha(self, indice):
        """ 
        Método auxiliar para verificar se um ponto é uma ponta de orelha.
        """
        indice_anterior = (indice - 1) % len(self.pontos)
        indice_atual = indice
        indice_proximo = (indice + 1) % len(self.pontos)

        segmento1 = self.pontos[indice_atual] - self.pontos[indice_anterior]
        segmento2 = self.pontos[indice_proximo] - self.pontos[indice_atual]

        # verifica se há uma curva à esquerda
        if segmento1 * segmento2 < 0:
            # define os vértices do triângulo
            v1 = self.pontos[indice_anterior]
            v2 = self.pontos[indice_atual]
            v3 = self.pontos[indice_proximo]

            # verifica se há um ponto dentro do triângulo
            for i in range(len(self.pontos)):
                if i in [indice_anterior, indice_atual, indice_proximo]:
                    continue
                if dentro_do_triangulo(p=self.pontos[i], v1=v1, v2=v2, v3=v3):
                    return False

            self.triangulos.append([v1, v2, v3])
            return True

        return False


    def cortedeorelha_aux(self, idx):
        """ 
        Controla a remoção de pontos e a lista de soluções.
        
        """
        if self.ponta_de_orelha(idx):
            self._solucao.append([
                self.pontos[(idx - 1) % len(self.pontos)], 
                self.pontos[(idx + 1) % len(self.pontos)]
            ])
            self.pontos.pop(idx)
        else:
            self._solucao.append(None)
            self.indice += 1

    def solucao(self):
        return self._solucao

    def construir_graficos(self):
        """ 
        Constrói uma lista de etapas de gráfico.
        
        Returns:
        tuple: Três listas - uma de listas de pontos, uma de segmentos e outra com mensagens detalhando cada etapa do algoritmo.
        """
        # Começa pelo polígono original
        self.pontos = self._pontos_iniciais
        indices_pontos = {
            pt: i for i, pt in enumerate(self._pontos_iniciais)
        }

        pontos = [(xi, yi, 'blue') for xi, yi in zip(self.xs, self.ys)]
        segmentos = []
        for i in range(len(pontos)):
            p = pontos[i]
            q = pontos[(i + 1) % len(pontos)]
            segmentos.append(([p[0], p[1]], [q[0], q[1]], 'blue'))
        segmentos = [segmentos]
        pontos = [pontos]
        mensagens = ["Polígono inicial"]

        i = 0
        for s in self._solucao:
            # Indica qual ponto é a referência na etapa
            segmentos.append([])
            pontos.append([])
            pontos[-1] = pontos[-2].copy()
            pt_indice = indices_pontos[self.pontos[i % len(self.pontos)]]
            pontos[-1][pt_indice] = (
                pontos[-1][pt_indice][0],
                pontos[-1][pt_indice][1],
                'red'
            )
            segmentos[-1] = segmentos[-2].copy()
            mensagens.append('Analisando vértice %s ...' % self.pontos[i % len(self.pontos)])

            # Índice i é uma ponta de orelha
            if s is not None:
                # Plota o segmento adicionado
                segmentos.append([])
                pontos.append([])
                pontos[-1] = pontos[-2].copy()
                segmentos[-1] = segmentos[-2].copy()
                segmentos[-1].append(([s[0].x, s[0].y], [s[1].x, s[1].y], '-k'))
                mensagens.append('Vértice %s é uma ponta de orelha' % self.pontos[i % len(self.pontos)])
                self.pontos.pop(i % len(self.pontos))

            else:
                # Se não, reverte a cor do vértice e adiciona uma mensagem
                segmentos.append([])
                pontos.append([])
                pontos[-1] = pontos[-2].copy()
                pontos[-1][pt_indice] = (
                    pontos[-1][pt_indice][0],
                    pontos[-1][pt_indice][1],
                    'blue'
                )
                segmentos[-1] = segmentos[-2].copy()
                mensagens.append('Vértice %s não é uma ponta de orelha' % self.pontos[i % len(self.pontos)])
                i += 1

        # Constrói um gráfico intermediário mostrando o resultado final da triangulação
        pontos.append(pontos[-1].copy())
        pontos[-1] = [(e[0], e[1], 'red') for e in pontos[-1]]
        segmentos.append(segmentos[-1].copy())
        segmentos[-1] = [([e[0][0], e[0][1]], [e[1][0], e[1][1]], 'black') for e in segmentos[-1]]
        mensagens.append("Triangulação final")


        return pontos, segmentos, mensagens
```
## Classe ArvoreDeTriangulos
A classe 'ArvoreDeTriangulos' implementa um grafo com triângulos para obter uma coloração 3-cromática.
```python
class ArvoreDeTriangulos:
    """ 
    Grafo com triângulos para obter coloração 3-cromática.
    """
    def __init__(self, triangulos):
        """ 
        Constrói a matriz de adjacência e inicia todos os vértices com a mesma cor no passo 0.
        
        """
        self._triangulos = triangulos
        self._matriz_adjacencia = [[] for e in triangulos]
        self._vertices = set()
        for e in triangulos:
            for v in e:
                self._vertices.add(v)
        self.cor_padrao = {
            v: 0 for v in self._vertices
        }

        self._cores_vertices = [self.cor_padrao.copy()]

        for i in range(len(triangulos)):
            for j in range(len(triangulos)):
                if i == j:
                    continue
                conjunto_vertices = set()
                for e in triangulos[i]: conjunto_vertices.add(e)
                for e in triangulos[j]: conjunto_vertices.add(e)
                
                # dois vértices em comum devem ser vizinhos
                if len(conjunto_vertices) == 4:
                    self._matriz_adjacencia[i].append(j)

    def colorir_triangulo(self, triangulo):
        """ 
        Colore os vértices do triângulo.
        
        """
        para_colorir = None
        soma = 0
        for v in triangulo:
            cor = self._cores_vertices[-1][v]
            if cor == 0:
                para_colorir = v
            soma += cor
        if para_colorir is not None:
            self._cores_vertices.append(self._cores_vertices[-1].copy())
            self._cores_vertices[-1][para_colorir] = 6 - soma

    def dfs_aux(self, vertice, visitados):
        visitados.add(vertice)
        for vizinho in self._matriz_adjacencia[vertice]:
            if vizinho not in visitados:
                self.colorir_triangulo(self._triangulos[vizinho])
                self.dfs_aux(vizinho, visitados)

    def dfs(self):
        visitados = set()
        # inicia os primeiros vértices com 3 cores diferentes
        self._cores_vertices.append(self._cores_vertices[-1].copy())
        for i, v in enumerate(self._triangulos[0]):
            self._cores_vertices[-1][v] = i + 1
        self.dfs_aux(0, visitados)

        return self._cores_vertices
```
# Funções de Visualização
## grafico interatico
Função para criar gráficos usando Plotly.

```python
def grafico_interativo(pontos_dados, segmentos_linhas, mensagens):
    """ 
    Cria um gráfico interativo a partir dos dados fornecidos.
    """
    fig = go.Figure()

    # dados iniciais 
    pontos_iniciais = pontos_dados[0]
    x = [ponto[0] for ponto in pontos_iniciais]
    y = [ponto[1] for ponto in pontos_iniciais]
    cores = [ponto[2] for ponto in pontos_iniciais]  # Supondo que as cores já estão na notação correta

    # pontos iniciais
    fig.add_trace(go.Scatter(x=x, y=y, mode='markers', marker=dict(color=cores, size=12)))

    passos = []

    # cada step altera a cor dos pontos e adiciona as linhas correspondentes
    for i in range(len(pontos_dados)):
        pontos = pontos_dados[i]
        x = [ponto[0] for ponto in pontos]
        y = [ponto[1] for ponto in pontos]
        cores = [ponto[2] for ponto in pontos]  # Supondo que as cores já estão na notação correta

        # define os shapes (linhas) para o step atual
        formas = []
        for segmento in segmentos_linhas[i]:
            formas.append({
                'type': 'line',
                'x0': segmento[0][0],
                'y0': segmento[0][1],
                'x1': segmento[1][0],
                'y1': segmento[1][1],
                'line': {
                    'color': segmento[2],  # Supondo que as cores dos segmentos já estão corretas
                    'width': 2
                }
            })

        # define o step do slider
        passo = {
            'method': 'update',
            'args': [
                {'marker.color': [cores]},  # Restyle: atualiza as cores dos pontos
                {'shapes': formas},          # Relayout: atualiza os shapes (linhas)
                {'annotations': [dict(
                    xref='paper',
                    yref='paper',
                    x=0.5,
                    y=-0.1,
                    xanchor='center',
                    yanchor='top',
                    text=mensagens[i],
                    showarrow=True
                )]}  # atualiza a mensagem
            ],
            'label': mensagens[i]
        }

        passos.append(passo)

    fig.update_layout(
        sliders=[{
            'active': 0,  # step ativo no início
            'currentvalue': {'prefix': 'Passo: '},  # prefixo exibido com o valor atual do slider
            'pad': {"t": 50}, 
            'steps': passos  # passa a lista de steps configurada acima
        }],
        showlegend=False 
    )

    fig.write_html("coloring.html")

    return fig
```
## solucionador
Envolve a solução para o problema de coloração de galeria usando a triangulação e coloração 3-cromática.

```python
def solucionador(points):
    """ 
    Solução para o problema de coloração de galeria usando a triangulação e coloração 3-cromática.
    """
    # verifica se os pontos estão em ordem anti-horária
    if anti_horario(points):
        points = points[::-1]
    
    # extrai as coordenadas x e y dos pontos
    xs = [e[0] for e in points]
    ys = [e[1] for e in points]

    ear_clipper = CorteDeOrelhas(xs, ys)
    pontos, segmentos, mensagens = ear_clipper.construir_graficos()

    # triângulos resultantes da triangulação
    triangulos = ear_clipper.triangulos
    resultado_tres_cores = ArvoreDeTriangulos(triangulos).dfs()

    # mapa de cores para a coloração 3-cromática
    mapa_cores = {0: 'blue', 1: 'black', 2: 'yellow', 3: 'magenta'}

    """ 
    Gera uma lista de pontos e suas cores para o passo de coloração 3-cromática 
    """
    for d in resultado_tres_cores:
        lista_pontos = []
        for e in pontos[-1]:
            lista_pontos.append((e[0], e[1], mapa_cores[d[Ponto(e[0], e[1])]]))
        pontos.append(lista_pontos)
        segmentos.append(segmentos[-1].copy())
        mensagens.append("Colorindo...")

        qt = {1: 0, 2: 0, 3: 0}
        d = resultado_tres_cores[-1]
        for k, v in d.items():
            qt[v] += 1
    
    mensagens[-1] = "Concluído. O número mínimo de câmeras nesta galeria é %d" % min(qt.values())

    grafico_interativo(pontos, segmentos, mensagens).show()
```

# Exemplo: (veja mais na implementação no diretorio do GitHub)

```python
input = [
(3,4),
(2,2), 
(3.5025,1.02125), 
(3.8025,2.64125), 
(4.7825,1.22125), 
(6.2225,1.20125), 
(6.5225,2.42125), 
(5.6025,3.58125), 
(5.0625,2.52125), 
(4.3425,3.48125), 
(5.3825,4.68125)
]

solucionador(input)
```
{% raw %}
<iframe src="coloring.html" width="600" height="400"></iframe>
{% endraw %}

[Download do Notebook Jupyter](./ALG2_TP1_202401.ipynb)
