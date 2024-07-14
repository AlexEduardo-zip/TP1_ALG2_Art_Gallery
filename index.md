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
