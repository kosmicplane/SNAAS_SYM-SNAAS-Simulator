# Mathematical model

This document summarizes the mathematical abstractions used by SWARMSYM for graph state, communication feasibility, and failure-aware packet progression. These equations describe **simulation models**; they should not be interpreted as measured real-world wireless performance unless a specific experiment provides that evidence.

## 1. Time-varying swarm graph

At time $t$, the swarm is represented by

$$
G(t)=\bigl(V(t),E(t)\bigr),
$$

where $V(t)$ is the set of UAVs and $E(t)$ the candidate communication edges.

If $F(t)\subseteq V(t)$ is the set of failed or unavailable UAVs, the active node set is

$$
V_a(t)=V(t)\setminus F(t).
$$

For UAV positions $\mathbf p_i(t)$ and $\mathbf p_j(t)$,

$$
d_{ij}(t)
=
\left\|
\mathbf p_i(t)-\mathbf p_j(t)
\right\|_2.
$$

This geometric relation is only one input to the link decision; proximity alone does not imply communication feasibility.

## 2. Analytical link budget

The analytical received-power model is

$$
P_{\mathrm{rx},ij}
=
P_{\mathrm{tx},i}
+
G_{\mathrm{tx},i}
+
G_{\mathrm{rx},j}
-
L_{\mathrm{total},ij}.
$$

Thermal-noise power is approximated in dBm by

$$
N_{\mathrm{dBm}}
=
-174
+
10\log_{10}(B)
+
NF,
$$

where:

- $B$ is the configured receiver bandwidth;
- $NF$ is the receiver noise figure.

The corresponding signal-quality quantity is evaluated through the selected SNR/SINR model.

## 3. Theoretical capacity

The link-capacity abstraction is

$$
C_{ij}
=
\eta B
\log_2\!\left(
1+\mathrm{SINR}_{ij}
\right),
$$

with efficiency factor $\eta$.

The resulting $C_{ij}$ is retained as a **theoretical channel metric**. It is not labeled as achieved application throughput unless a higher-fidelity experiment explicitly measures and reports that quantity.

## 4. Communication feasibility

A compact nominal gate is

$$
g_{ij}(t)
=
\mathbf 1_{\{d_{ij}(t)\le d_{\max}\}}
\mathbf 1_{\{\mathrm{SINR}_{ij}(t)\ge \gamma\}}
\mathbf 1_{\{C_{ij}(t)\ge C_{\min}\}}
\mathbf 1_{\{\Delta t_{ij}(t)\le T_{\mathrm{fresh}}\}}.
$$

Here:

- $d_{\max}$ is the configured operational or outage range;
- $\gamma$ is the required SNR/SINR threshold;
- $C_{\min}$ is the minimum configured capacity;
- $T_{\mathrm{fresh}}$ is the maximum accepted age of the current metric sample.

Failure-aware feasibility requires active endpoints:

$$
g^F_{ij}(t)
=
g_{ij}(t)
\mathbf 1_{\{i\in V_a(t)\}}
\mathbf 1_{\{j\in V_a(t)\}}.
$$

Only links satisfying the full runtime gate are allowed to advance authoritative packet state.

## 5. Topology-dependent packet semantics

The graph can be interpreted under several runtime modes:

- **Chain:** one infeasible active hop pauses the end-to-end stream.
- **Parallel:** branch cursors progress independently when their own paths remain feasible.
- **Forest:** packet/service state is maintained per subtree.
- **Manual:** operator-defined edges are preserved, but every active edge remains subject to the same physical and communication feasibility conditions.

## 6. Failure and recovery

Let $G_a(t)$ denote the subgraph induced by active nodes and feasible edges. A fault changes either the node set, the edge set, or both. Recovery therefore requires more than drawing a replacement path:

$$
G(t)
\;\longrightarrow\;
G_a(t)
\;\longrightarrow\;
G_{\mathrm{candidate}}(t)
\;\longrightarrow\;
G_{\mathrm{feasible}}(t).
$$

Service is resumed only after the replacement topology has been acknowledged by the participating subsystems and the active route satisfies the configured communication gate.

## 7. Interpretation

The mathematical layer is intentionally separated from model fidelity:

- analytical link calculations support fast deterministic studies;
- stochastic or geometry-aware Sionna models add propagation detail;
- Isaac Sim supplies embodied vehicle/world state;
- optional PX4 execution adds autopilot-level dynamics.

Keeping these levels explicit prevents analytical, simulated, and externally measured quantities from being treated as interchangeable evidence.
