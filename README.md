

---

# 1. Inferência Causal em Grafos por Intervenção em Nós

<div align="justify">

A pergunta:

> “Esse nó causou a divisão do grupo?”

pode ser tratada como um problema de **efeito causal de remoção de nós em um grafo**.

A ideia é simples:

> remover um nó e observar como a estrutura do grafo muda.

Se a estrutura muda muito, esse nó tem influência causal na organização do sistema.

</div>

---

# 2. Formalização do Problema

<div align="justify">

Considere um grafo ( G = (V, E) ), onde:

* ( V ): nós
* ( E ): conexões

Definimos:

* ( G_0 ): grafo original
* ( G_{-v} ): grafo sem o nó ( v )
* ( Y(G) ): medida estrutural

O efeito causal do nó é:

[
\Delta_v = Y(G_0) - Y(G_{-v})
]

Se ( \Delta_v ) for grande, o nó tem forte impacto estrutural.

</div>

---

# 3. Medidas Estruturais

<div align="justify">

## 3.1 Número de componentes conectados

[
Y(G) = \text{número de componentes conectados}
]

Interpretação:

* aumenta após remoção → o nó mantinha o grafo unido
* indica papel de ponte estrutural

---

## 3.2 Modularidade

[
Q = \frac{1}{2m} \sum_{ij} \left(A_{ij} - \frac{k_i k_j}{2m}\right)\delta(c_i, c_j)
]

Interpretação:

* aumenta → o nó conectava comunidades diferentes
* diminui → o nó estava dentro de uma comunidade

</div>

---

# 4. Procedimento de Intervenção

<div align="justify">

1. calcular métricas do grafo original
2. remover um nó
3. recalcular métricas
4. comparar diferenças
5. repetir para todos os nós

Isso gera um ranking de influência causal estrutural.

</div>

---

# 5. Código Completo

```python
import networkx as nx
from community import community_louvain

# -----------------------------
# Métricas do grafo
# -----------------------------

def calcular_metricas(G):
    componentes = nx.number_connected_components(G)

    if G.number_of_edges() > 0:
        particao = community_louvain.best_partition(G)
        modularidade = community_louvain.modularity(particao, G)
    else:
        modularidade = 0

    return {
        "componentes": componentes,
        "modularidade": modularidade
    }

# -----------------------------
# Intervenção (remoção de nó)
# -----------------------------

def remover_no(G, no):
    G2 = G.copy()
    if no in G2:
        G2.remove_node(no)
    return G2

# -----------------------------
# Efeito causal
# -----------------------------

def efeito_causal(G, no):
    base = calcular_metricas(G)
    novo = calcular_metricas(remover_no(G, no))

    return {
        "no": no,
        "delta_componentes": novo["componentes"] - base["componentes"],
        "delta_modularidade": novo["modularidade"] - base["modularidade"]
    }

# -----------------------------
# Ranking causal
# -----------------------------

def ranking_causal(G):
    resultados = []

    for no in G.nodes():
        resultados.append(efeito_causal(G, no))

    for r in resultados:
        r["score"] = abs(r["delta_componentes"]) * 2 + abs(r["delta_modularidade"])

    return sorted(resultados, key=lambda x: x["score"], reverse=True)

# -----------------------------
# Exemplo
# -----------------------------

G = nx.erdos_renyi_graph(50, 0.05, seed=42)

ranking = ranking_causal(G)

for r in ranking[:10]:
    print(r)
```

---

# 6. Interpretação

<div align="justify">

Esse método mede algo diferente de centralidade comum:

* não é só “quem é importante localmente”
* é “quem muda a estrutura global quando removido”

Ou seja:

> uma aproximação de causalidade estrutural em grafos

---

Nós com alto impacto geralmente são:

* pontes entre regiões
* conectores de comunidades
* estruturas críticas de coesão

</div>

---

# 7. Extensões

<div align="justify">

* remoção de múltiplos nós ao mesmo tempo
* grafos dinâmicos no tempo
* aprendizado automático da importância causal
* simulações de propagação em redes

</div>

---

# 8. Conclusão

<div align="justify">

O método transforma grafos em um sistema de análise causal:

* nós = elementos manipuláveis
* remoção = intervenção
* mudança estrutural = efeito observado

Isso permite identificar influência estrutural de forma direta e interpretável.

</div>
