# Social Network Analysis — Avatar Co-appearance Network

Graph-theoretic analysis of the character co-appearance network of the film *Avatar*, modelled as an undirected weighted graph.

- **Course:** Social Network Analysis · BSc Artificial Intelligence & Management
- **Institution:** Luiss Guido Carli University, Rome
- **Period:** March – May 2026
- **Group:** C: Carla Di Renzo, Bianca Marino, Tommaso Moriondo, Eleonora Raimondo, Alessandro Torre

**Data:** J. Kaminski et al., *Moviegalaxies — Social Networks in Movies*,
Harvard Dataverse V3 (2018). https://doi.org/10.7910/DVN/T4HBA3

---

## The network

| Property | Value |
|---|---|
| Nodes | 30 characters |
| Edges | 105 co-appearances |
| Edge weight | Number of shared scenes |
| Type | Undirected, weighted |
| Average degree | 7.0 |

An average degree of 7 across only 30 nodes describes a densely connected cast, each character shares scenes with roughly a quarter of the others.

## Structure and clustering

Average clustering is **0.80**: four in five of a character's co-stars also appear together. Transitivity is **0.50**, and the gap between the two is itself informative, the unweighted average is inflated by low-degree characters sitting inside single tight groups.

Plotting the cumulative distribution of clustering (rather than a histogram, which would depend on arbitrary bin choices) shows a flat region from 0 to ~0.38 covering roughly 7% of nodes. These are minor characters — OPERATOR,
SUPERVISOR: who appear briefly alongside major cast members whose own connections never meet. They are structurally isolated connectors with no triangles around them.

Comparing each node's clustering against the average clustering of its neighbours reveals **disassortative clustering mixing**: the neighbour curve sits left of the node curve almost everywhere, meaning a character's co-stars are typically *less* embedded in tight groups than the character themselves.

## Centrality (who actually holds the story together)

Analysis runs on the largest connected component, with `inv_weight = 1/weight` as the distance attribute so that frequently co-appearing characters count as structurally *closer*.

**Betweenness centrality** produces the clearest result in the project:

| Character | Betweenness |
|---|---:|
| JAKE | 0.737 |
| QUARITCH | 0.173 |
| SELFRIDGE | 0.133 |

JAKE's score is more than four times the next character's. He is the sole structural bridge between the human military and the Na'vi — remove him and the two factions are almost entirely disconnected. This is a genuinely different question from "who appears most often", and it recovers the narrative architecture of the film from co-appearance data alone.

## Community detection

Two methods were implemented and compared on the unweighted, loop-free largest connected component.

**Modularity optimisation** (Q = 0.2675) produced three communities that map onto the plot:

- **Human military core** — JAKE, SELFRIDGE, WAINFLEET, TROOPER, AGENT 1, AGENT 2, OPERATOR, SUPERVISOR
- **Na'vi and conflict group** — QUARITCH, NEYTIRI, TSU'TEY, EYTUKAN, MO'AT, TRUDY, GUNNER, PILOTS
- **Science and support team** — GRACE, NORM, MAX, MED TECH, TECH

Each satisfies the weak community condition: internal degree exceeds external degree.

**Label propagation** collapsed almost the whole network into a single label with very low modularity. That isn't a bug — it's the known failure mode of the method on dense, strongly connected graphs, where one label floods the graph and the result depends heavily on random visit order and tie-breaking.

Modularity optimisation was therefore selected. The partition was exported to GEXF and visualised in Gephi Lite (`notebooks/avatar_communities.gexf`, `figure/`), with colour encoding community and size encoding degree.

## Link prediction

Framed as **link completion**: given the network as a snapshot, which pairs of characters are structurally likely to have shared a scene that was never recorded?

Ranked by common neighbours, with Adamic-Adar as tiebreaker, then combined into a single score after min-max rescaling both indices to [0, 1] — necessary because indices on different scales would otherwise contribute unequally.

The top candidates (PILOTS ↔ GRACE, GUNNER ↔ GRACE, TRUDY ↔ TSU'TEY) all share six common neighbours, the maximum observed. Adamic-Adar separates the ties by penalising shared neighbours that are high-degree hubs — a connection through JAKE, who is connected to everyone, is weaker evidence than a connection through a peripheral character.

## Random graph comparison

An Erdős–Rényi G(n, p) model was calibrated to match the Avatar network on node count and average degree, then compared on everything else.

The two diverge sharply. Average clustering is **0.80 in Avatar against 0.26 in ER** — roughly three times higher. The degree CDFs tell the same story: ER concentrates between degrees 5 and 13 with almost every node near 7, while Avatar spans 1 to 25.

The conclusion is that neither the group structure nor the hub-and-periphery degree heterogeneity of a film cast can be reproduced by a random process matched on density alone.

## Diffusion: SIS contagion

An SIS model was implemented and used to test whether structural centrality translates into influence, seeding infection from characters with deliberately different structural roles (β = 0.3, δ = 0.1).

All origins except SUPERVISOR reach the endemic level (25–29 nodes) within five steps. SUPERVISOR — a peripheral node of degree 2 — starts far more slowly, infecting only one node at step 0, but converges to the *same* endemic equilibrium by around step 10.

The nuance is the interesting part: in a network this dense, seed choice changes the **speed** of diffusion substantially but not the eventual **extent**. Central seeding buys time, not reach.

## My contribution

My work was the analytical and written layer of the project rather than the implementation. Specifically, I:

- **Wrote the full analytical narrative** — every interpretation section in the notebook, including the disassortative clustering mixing result, the reading of JAKE's betweenness dominance as narrative structure, the diagnosis of why label propagation fails on dense graphs, and the SIS conclusion that seed centrality governs diffusion speed but not eventual extent.
- **Set out the theoretical grounding** for each method — the formal definitions of the local clustering coefficient, modularity against a randomised baseline, the Erdős–Rényi construction, and the basic reproduction number R₀ — so that each technique was motivated before it was applied.
- **Interpreted every graph and visualisation** produced in the analysis, translating structural metrics into claims about the film's narrative architecture.
- **Built the Gephi community visualisation**, exporting the modularity partition to GEXF and producing the final rendering with colour encoding community membership and size encoding degree.

The consistent thread is turning computed output into an argument: deciding which measure answers which question, and stating what the numbers actually mean.

## Repository structure

```
data/           nodes.csv, edges.csv (Moviegalaxies)
figure/         Gephi Lite community visualisation
notebooks/      notebooks.ipynb — full analysis
                avatar_communities.gexf — Gephi export
```

## Stack

Python 3.12 · NetworkX · pandas · NumPy · matplotlib · Gephi Lite

## Original team repository

https://github.com/MoriondoTommaso/social-network-analysis-avatar-movie
