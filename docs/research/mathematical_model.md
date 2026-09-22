# Mathematical Model

This document states the principal graph, communication, and service-feasibility relations used by SWARMSYM. All display equations use GitHub's fenced `math` syntax to avoid raw-LaTeX rendering failures.

## 1. Time-varying graph

```math
G(t)=\bigl(V(t),E(t)\bigr).
```

Failed nodes:

```math
F(t)\subseteq V(t).
```

Active nodes:

```math
V_a(t)=V(t)\setminus F(t).
```

Inter-UAV distance:

```math
d_{ij}(t)
=
\left\|
p_i(t)-p_j(t)
\right\|_2.
```

---

## 2. Link budget

```math
P_{\mathrm{rx},ij}
=
P_{\mathrm{tx},i}
+
G_{\mathrm{tx},i}
+
G_{\mathrm{rx},j}
-
L_{\mathrm{total},ij}.
```

Thermal-noise approximation:

```math
N_{\mathrm{dBm}}
=
-174
+
10\log_{10}(B)
+
NF.
```

If signal and noise/interference terms are converted consistently, the selected SNR/SINR model produces `SINR_ij`.

---

## 3. Theoretical capacity

```math
C_{ij}
=
\eta B
\log_2
\left(
1+\mathrm{SINR}_{ij}
\right).
```

`C_ij` is a theoretical simulation quantity unless an experiment explicitly measures a corresponding application-layer rate.

---

## 4. Feasibility gate

```math
g_{ij}(t)
=
\mathbf 1_{\{d_{ij}\le d_{\max}\}}
\mathbf 1_{\{\mathrm{SINR}_{ij}\ge\gamma\}}
\mathbf 1_{\{C_{ij}\ge C_{\min}\}}
\mathbf 1_{\{\Delta t_{ij}\le T_{\mathrm{fresh}}\}}.
```

Failure-aware form:

```math
g^F_{ij}(t)
=
g_{ij}(t)
\mathbf 1_{\{i\in V_a(t)\}}
\mathbf 1_{\{j\in V_a(t)\}}.
```

---

## 5. Path feasibility

For path `P`,

```math
\chi_P(t)
=
\prod_{(i,j)\in P}
g^F_{ij}(t).
```

Hence

```math
\chi_P(t)=1
```

only when every hop on the path is feasible under the current model state.

---

## 6. Failure-aware graph

The active feasible graph is

```math
G_F(t)
=
\left(
V_a(t),
E_F(t)
\right),
```

with

```math
E_F(t)
=
\left\{
(i,j)\in E(t):
g^F_{ij}(t)=1
\right\}.
```

A recovery event must therefore restore a feasible service path in `G_F(t)`, not merely create a candidate edge in the visual topology.

---

## 7. Graph-level connectivity metrics

For an undirected adjacency matrix `A`, degree matrix `D`, and graph Laplacian

```math
L=D-A,
```

the algebraic connectivity is

```math
\lambda_2(L),
```

the second-smallest eigenvalue of `L`.

This quantity is a topology indicator only; it does not replace the link-level SNR/capacity/feasibility model.

---

## 8. Service semantics

- **Chain:** all hops on the active route must remain feasible.
- **Parallel:** each branch maintains independent progression state.
- **Forest:** state is maintained per subtree/branch.
- **Manual:** operator-selected edges remain subject to the same feasibility model.

The key distinction is

```math
\text{topological connectivity}
\neq
\text{communication-feasible service}.
```

---

## 9. Fidelity boundary

The same equations can be populated by different model sources:

- analytical path-loss/link models;
- stochastic channel models;
- geometry-aware Sionna outputs;
- external protocol-aware co-simulation.

Model provenance must therefore accompany metrics when results from different fidelity levels are compared.
