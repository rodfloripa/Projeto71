

---

# 1. Inferência causal em grafos por remoção de nós

A ideia é responder:

> “Esse nó causou a divisão do grupo?”

fazendo uma intervenção:

* remover o nó
* observar o que muda no grafo

Se a estrutura muda muito, o nó tem influência causal.

---

# 2. Formalização

Considere um grafo:

G = (V, E)

Onde:

* V = nós
* E = conexões

Definimos:

* G₀ = grafo original
* G₋ᵥ = grafo sem o nó v
* Y(G) = medida estrutural

O efeito causal é:

Δᵥ = Y(G₀) − Y(G₋ᵥ)

---

# 3. Métricas estruturais

## 3.1 Componentes conectados

Y(G) = número de componentes conectados

Interpretação:

* aumenta após remoção → o nó mantinha o grafo unido
* indica papel de “ponte”

---

## 3.2 Modularidade

Q = (1 / 2m) Σᵢⱼ (Aᵢⱼ − (kᵢ kⱼ) / (2m)) δ(cᵢ, cⱼ)

Interpretação:

* aumenta → nó conectava comunidades diferentes
* diminui → nó era interno a um grupo

---

# 4. Procedimento

1. calcular métricas do grafo original
2. remover um nó
3. recalcular métricas
4. comparar diferença
5. repetir para todos os nós

---

# 5. Código completo

```python
import networkx as nx
from community import community_louvain

# -----------------------------
# métricas do grafo
# -----------------------------
def metricas(G):
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
# intervenção: remover nó
# -----------------------------
def remover_no(G, no):
    G2 = G.copy()
    G2.remove_node(no)
    return G2

# -----------------------------
# efeito causal
# -----------------------------
def efeito_causal(G, no):
    base = metricas(G)
    novo = metricas(remover_no(G, no))

    return {
        "no": no,
        "delta_componentes": novo["componentes"] - base["componentes"],
        "delta_modularidade": novo["modularidade"] - base["modularidade"]
    }

# -----------------------------
# ranking causal
# -----------------------------
def ranking_causal(G):
    resultados = []

    for no in G.nodes():
        resultados.append(efeito_causal(G, no))

    for r in resultados:
        r["score"] = abs(r["delta_componentes"]) * 2 + abs(r["delta_modularidade"])

    return sorted(resultados, key=lambda x: x["score"], reverse=True)


# -----------------------------
# exemplo
# -----------------------------
G = nx.erdos_renyi_graph(50, 0.05, seed=42)

ranking = ranking_causal(G)

for r in ranking[:10]:
    print(r)
```

---

# 6. Interpretação

Esse método mede algo mais forte que centralidade comum:

* não é “quem tem mais conexões”
* é “quem muda a estrutura do sistema quando removido”

Ou seja:

> uma forma simples de inferência causal em redes

---

# 7. Extensão

* remover múltiplos nós
* grafos dinâmicos
* simulação de propagação
* redes neurais em grafos

---

Se quiser, posso te mandar uma versão **nível paper**, com:

* formalização estilo artigo
* ligação com teoria do “do-operator”
* versão com grafos dinâmicos (tempo real)
* ou versão com redes neurais em grafos para aprender causalidade automaticamente
