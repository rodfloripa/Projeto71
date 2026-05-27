# 1. Inferência Causal em Grafos por Intervenção em Nós

<div align="justify">

A pergunta:

> “Esse nó causou a divisão do grupo?”

pode ser reformulada como um problema de **efeito causal de uma intervenção estrutural no grafo**, onde removemos um nó e observamos como a estrutura muda.

A ideia central é simples:

> Se eu remover um nó e o grafo se fragmenta ou muda significativamente, esse nó tem influência causal na coesão do sistema.

</div>

---

# 2. Formalização do Problema

<div align="justify">

Considere um grafo ( G = (V, E) ), onde:

* ( V ): conjunto de nós
* ( E ): conjunto de conexões

Definimos:

* ( G_0 ): grafo original
* ( G_{-v} ): grafo após remover o nó ( v )
* ( Y(G) ): medida estrutural do grafo

O efeito do nó ( v ) é:

[
\Delta_v = Y(G_0) - Y(G_{-v})
]

Se essa diferença for grande, o nó tem forte influência estrutural.

</div>

---

# 3. Medidas Estruturais

<div align="justify">

Aqui usamos duas medidas principais para avaliar o impacto:

## 3.1 Número de componentes conectados

[
Y(G) = \text{quantidade de componentes conectados}
]

Interpretação:

* aumento após remoção → o nó mantinha o grafo unido
* indica papel de ponte estrutural

---

## 3.2 Modularidade

A modularidade mede o quanto o grafo se organiza em grupos:

[
Q = \frac{1}{2m} \sum_{ij} \left(A_{ij} - \frac{k_i k_j}{2m}\right)\delta(c_i, c_j)
]

Interpretação:

* aumento → o nó conectava grupos diferentes
* redução → o nó estava dentro de um grupo específico

</div>

---

# 4. Procedimento de Intervenção

<div align="justify">

O procedimento geral é:

1. calcular as medidas do grafo original
2. remover um nó
3. recalcular as medidas
4. comparar a diferença
5. repetir para todos os nós

Isso gera uma ordem de importância causal estrutural.

</div>

---

# 5. Código Completo

```python
import networkx as nx
import numpy as np
from community import community_louvain

# -----------------------------
# 1. Medidas do grafo
# -----------------------------

def calcular_medidas(G):
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
# 2. Remoção de nó (intervenção)
# -----------------------------

def remover_no(G, no):
    copia = G.copy()
    if no in copia:
        copia.remove_node(no)
    return copia

# -----------------------------
# 3. Efeito causal de um nó
# -----------------------------

def efeito_causal_no(G, no):
    base = calcular_medidas(G)
    intervencionado = calcular_medidas(remover_no(G, no))

    return {
        "no": no,
        "delta_componentes": intervencionado["componentes"] - base["componentes"],
        "delta_modularidade": intervencionado["modularidade"] - base["modularidade"]
    }

# -----------------------------
# 4. Ranking de influência causal
# -----------------------------

def ranking_causal(G):
    resultados = []

    for no in G.nodes():
        resultados.append(efeito_causal_no(G, no))

    for r in resultados:
        r["pontuacao_causal"] = (
            abs(r["delta_componentes"]) * 2 +
            abs(r["delta_modularidade"])
        )

    return sorted(resultados, key=lambda x: x["pontuacao_causal"], reverse=True)

# -----------------------------
# 5. Exemplo
# -----------------------------

G = nx.erdos_renyi_graph(50, 0.05, seed=42)

ranking = ranking_causal(G)

for r in ranking[:10]:
    print(r)
```

---

# 6. Interpretação dos Resultados

<div align="justify">

Esse método não mede apenas centralidade ou importância local.

Ele mede:

> o quanto a estrutura global do grafo muda quando um nó é removido

Assim, conseguimos identificar:

* nós que conectam regiões diferentes do grafo
* nós que mantêm a coesão do sistema
* nós redundantes (baixa influência estrutural)

---

Diferente de medidas tradicionais, aqui existe uma ideia explícita de:

> intervenção e comparação contrafactual

</div>

---

# 7. Extensões Possíveis

<div align="justify">

Esse método pode ser expandido para:

* remoção simultânea de múltiplos nós
* grafos que mudam ao longo do tempo
* modelos neurais que aprendem influência estrutural
* simulação de difusão (como propagação de informação ou doenças)

</div>

---

# 8. Conclusão

<div align="justify">

A abordagem transforma o problema em uma análise estrutural baseada em intervenção:

* nós são elementos manipuláveis
* remoção é a intervenção
* mudanças estruturais são o efeito observado

Isso permite identificar influência causal dentro do grafo de forma direta e interpretável.

</div>
