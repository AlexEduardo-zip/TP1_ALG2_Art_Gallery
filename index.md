---
layout: default
title: Projeto de Análise de Dados
---

## Trabalho Prático 1 - Algoritmos 2

### Nome: Alex Eduardo Alves dos Santos --- Matrícula: 2021032145
### Nome: Paula

Este projeto implementa a triangulação de polígonos pelo método de corte de orelhas e a coloração de um grafo de triângulos para obter uma coloração 3-cromática. Utiliza a biblioteca Plotly para gerar gráficos interativos que ilustram cada passo do algoritmo.

### Classe Ponto

A classe `Ponto` representa um ponto em um plano cartesiano com coordenadas x e y. Ela inclui métodos para somar, subtrair, calcular o produto vetorial e verificar igualdade entre pontos, além de métodos para exibir o ponto como string.

```python
class Ponto:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, outro):
        return Ponto(self.x + outro.x, self.y + outro.y)

    def __sub__(self, outro):
        return Ponto(self.x - outro.x, self.y - outro.y)

    def __mul__(self, outro):
        return self.x * outro.y - outro.x * self.y

    def __str__(self):
        return "(%.2f, %.2f) " % (self.x, self.y)

    def __eq__(self, outro):
        return self.x == outro.x and self.y == outro.y

    def __repr__(self):
        return "(%.2f, %.2f) " % (self.x, self.y)

    def __hash__(self):
        return hash((self.x, self.y))
```

### Funções Auxiliares
converter_para_lista_de_pontos
Converte duas listas de valores x e y em uma lista de objetos Ponto.

```python
def converter_para_lista_de_pontos(lista_x, lista_y):
    pontos = []
    for x, y in zip(lista_x, lista_y):
        pontos.append(Ponto(x, y))
    return pontos
```

### dentro_do_triangulo
Verifica se um ponto está dentro de um triângulo formado por três vértices.
```python
def dentro_do_triangulo(p, v1, v2, v3):
    return (v1 - p) * (v2 - v1) <= 0 and (v2 - p) * (v3 - v2) <= 0 and (v3 - p) * (v1 - v3) <= 0

```
### esta_em_sentido_anti_horario
Verifica se uma lista de pontos está em ordem anti-horári
```python
def esta_em_sentido_anti_horario(pontos):
    min_y = pontos[0][1]
    min_idx = 0

    for i, e in enumerate(pontos):
        if e[1] < min_y:
            min_y = e[1]
            min_idx = i
        elif e[1] == min_y:
            if e[0] > pontos[min_idx][0]:
                min_y = e[1]
                min_idx = i

    a = pontos[min_idx]
    a = Ponto(a[0], a[1])
    b = pontos[(min_idx - 1) % len(pontos)]
    b = Ponto(b[0], b[1])
    c = pontos[(min_idx + 1) % len(pontos)]
    c = Ponto(c[0], c[1])

    produto = (a - b) * (c - a)
    return produto > 0
```
## Classe CortadorDeOrelhas
A classe 'CortadorDeOrelhas' implementa o algoritmo de triangulação de polígonos pelo método de corte de orelhas.
```python
class CortadorDeOrelhas:
    def __init__(self, xs, ys):
        self._pontos_iniciais = converter_para_lista_de_pontos(xs, ys)
        self.pontos = self._pontos_iniciais.copy()
        self._solucao = []
        self.indice = 0
        self.xs = xs
        self.ys = ys

        self.triangulos = []
        while len(self.pontos) > 3:
            self.corte_de_orelha_aux(
                self.indice % len(self.pontos)
            )
        self.triangulos.append([p for p in self.pontos])

    def eh_ponta_de_orelha(self, indice):
        segmento1 = (self.pontos[indice] - self.pontos[(indice - 1) % len(self.pontos)])
        segmento2 = (self.pontos[(indice + 1) % len(self.pontos)] - self.pontos[indice])

        if segmento1 * segmento2 < 0:
            v1 = self.pontos[(indice - 1) % len(self.pontos)]
            v2 = self.pontos[indice]
            v3 = self.pontos[(indice + 1) % len(self.pontos)]
            novo_indice = indice + 2
            while novo_indice % len(self.pontos) != (indice - 1) % len(self.pontos):
                if dentro_do_triangulo(
                    p = self.pontos[novo_indice % len(self.pontos)],
                    v1 = v1,
                    v2 = v2,
                    v3 = v3
                ):
                    return False
                novo_indice += 1
            self.triangulos.append([v1, v2, v3])
            return True
        return False

    def corte_de_orelha_aux(self, idx):
        if self.eh_ponta_de_orelha(idx):
            self._solucao.append(
                [
                    self.pontos[(idx - 1) % len(self.pontos)],
                    self.pontos[(idx + 1) % len(self.pontos)]
                ]
            )
            self.pontos.pop(idx)
        else:
            self._solucao.append(None)
            self.indice += 1

    def solucao(self):
        return self._solucao

    def construir_graficos(self):
        self.pontos = self._pontos_iniciais
        indices_pontos = {
            pt: i for i, pt in enumerate(self._pontos_iniciais)
        }

        pontos = [(xi, yi, '.b') for xi, yi in zip(self.xs, self.ys)]
        segmentos = []
        for i in range(len(pontos)):
            p = pontos[i]
            q = pontos[(i + 1) % len(pontos)]
            segmentos.append(([p[0], p[1]], [q[0], q[1]], '.b'))
        segmentos = [segmentos]
        pontos = [pontos]
        mensagens = ["Polígono inicial"]

        i = 0
        for s in self._solucao:
            segmentos.append([])
            pontos.append([])
            pontos[-1] = pontos[-2].copy()
            pt_indice = indices_pontos[self.pontos[i % len(self.pontos)]]
            pontos[-1][pt_indice] = (
                pontos[-1][pt_indice][0],
                pontos[-1][pt_indice][1],
                '.r'
            )
            segmentos[-1] = segmentos[-2].copy()
            mensagens.append('Analisando vértice %s ...' % self.pontos[i % len(self.pontos)])

            if s is not None:
                segmentos.append([])
                pontos.append([])
                pontos[-1] = pontos[-2].copy()
                segmentos[-1] = segmentos[-2].copy()
                segmentos[-1].append(([s[0].x, s[0].y], [s[1].x, s[1].y], '-k'))
                mensagens.append('Vértice %s é uma ponta de orelha' % self.pontos[i % len(self.pontos)])
                self.pontos.pop(i % len(self.pontos))
            else:
                segmentos.append([])
                pontos.append([])
                pontos[-1] = pontos[-2].copy()
                pontos[-1][pt_indice] = (
                    pontos[-1][pt_indice][0],
                    pontos[-1][pt_indice][1],
                    '.b'
                )
                segmentos[-1] = segmentos[-2].copy()
                mensagens.append('Vértice %s não é uma ponta de orelha' % self.pontos[i % len(self.pontos)])
                i += 1

        pontos.append(pontos[-1].copy())
        pontos[-1] = [(e[0], e[1], '.r') for e in pontos[-1]]
        segmentos.append(segmentos[-1].copy())
        segmentos[-1] = [([e[0][0], e[0][1]], [e[1][0], e[1][1]], '.k') for e in segmentos[-1]]
        mensagens.append("Triangulação final")

        return pontos, segmentos, mensagens
```
### Classe ArvoreDeTriangulos
A classe 'ArvoreDeTriangulos' implementa um grafo com triângulos para obter uma coloração 3-cromática.
```python
class ArvoreDeTriangulos:
    def __init__(self, triangulos):
        self._triangulos = triangulos
        self._matriz = defaultdict(set)
        self.criar_arestas()

    def criar_arestas(self):
        for t in self._triangulos:
            for i in range(3):
                self._matriz[t[i]].add(t[(i+1) % 3])
                self._matriz[t[i]].add(t[(i+2) % 3])

    def colorir(self):
        cores = {}
        for t in self._triangulos:
            cores_possiveis = {0, 1, 2}
            for i in range(3):
                if t[i] in cores:
                    cores_possiveis.discard(cores[t[i]])
            cor_usada = cores_possiveis.pop()
            for i in range(3):
                if t[i] not in cores:
                    cores[t[i]] = cor_usada
        return cores

    def __str__(self):
        return str(dict(self._matriz))

```
### Funções de Visualização
## plotar_graficos
Função para criar gráficos usando Plotly.

```python
def plotar_graficos(pontos, segmentos, mensagens):
    frames = []

    for i in range(len(pontos)):
        scatter = [
            go.Scatter(x=[p[0] for p in pontos[i]], y=[p[1] for p in pontos[i]],
                       mode='markers', marker=dict(color=[p[2] for p in pontos[i]]))
        ]

        lines = [
            go.Scatter(x=[e[0][0], e[1][0]], y=[e[0][1], e[1][1]],
                       mode='lines', line=dict(color=e[2]))
            for e in segmentos[i]
        ]

        frames.append(go.Frame(data=scatter + lines, name=str(i), layout=go.Layout(title=mensagens[i])))

    fig = go.Figure(
        data=frames[0].data,
        layout=go.Layout(
            updatemenus=[
                {
                    "buttons": [
                        {
                            "args": [None, {"frame": {"duration": 500, "redraw": True}, "fromcurrent": True}],
                            "label": "Play",
                            "method": "animate",
                        },
                        {
                            "args": [[None], {"frame": {"duration": 0, "redraw": True}, "mode": "immediate",
                                              "transition": {"duration": 0}}],
                            "label": "Pause",
                            "method": "animate",
                        },
                    ],
                    "direction": "left",
                    "pad": {"r": 10, "t": 87},
                    "showactive": False,
                    "type": "buttons",
                    "x": 0.1,
                    "xanchor": "right",
                    "y": 0,
                    "yanchor": "top",
                }
            ],
            sliders=[{
                "yanchor": "top",
                "xanchor": "left",
                "currentvalue": {
                    "font": {"size": 20},
                    "prefix": "Passo:",
                    "visible": True,
                    "xanchor": "right"
                },
                "transition": {"duration": 500},
                "pad": {"b": 10, "t": 50},
                "len": 0.9,
                "x": 0.1,
                "y": 0,
                "steps": [
                    {"args": [[k], {"frame": {"duration": 500, "redraw": True}, "mode": "immediate",
                                    "transition": {"duration": 500}}],
                     "label": str(k), "method": "animate"}
                    for k in range(len(frames))
                ]
            }]
        ),
        frames=frames
    )

    fig.show()
```
Exemplo 1:

```python
input_ = [(-4,0), (-3,1), (-3, 10), (-1, 6), (0, 6), (2,4), (3,5), (4,0),(1,2), (0,-2)]
solucionador(input_)
```
{% raw %}
<iframe src="coloring.html" width="600" height="400"></iframe>
{% endraw %}

```python

```

```python

```

```html
<iframe src="coloring.html" width="600" height="400"></iframe>
```

