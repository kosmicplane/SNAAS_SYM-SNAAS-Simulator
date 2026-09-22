# SWARMSYM / NETLAB — Swarm Network-as-a-Service Research Platform

<p align="center">
  <strong>Communication-aware multi-UAV simulation · ROS 2 coordination · Isaac Sim embodiment · Sionna link evaluation · failure-aware recovery</strong>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-architecture.webp" alt="SWARMSYM system architecture" width="920">
</p>

SWARMSYM is a modular research platform for studying **UAV swarms that act as reconfigurable airborne communication infrastructure**. The central question is not only whether the vehicles can move through a scene, but whether the swarm can continue to provide an end-to-end communication service while vehicles move, wireless links degrade, nodes fail, and topology is reconfigured.

The platform couples four layers that are often evaluated independently:

1. **vehicle motion and world state** through ROS 2 and Isaac Sim;
2. **wireless-link evaluation** through Sionna-compatible propagation models;
3. **routing and topology logic** through a time-varying graph with failure-aware feasibility checks;
4. **experiment evidence** through synchronized revisions, telemetry, metrics, logs, and reproducible run artifacts.

---

## Research question

A visual route is not automatically an operational route. SWARMSYM therefore treats the communication state as part of the autonomy problem.

At time $t$, the swarm is represented by the graph

$$
G(t)=\bigl(V(t),E(t)\bigr),
$$

with failed vehicles

$$
F(t)\subseteq V(t)
$$

and active vehicles

$$
V_a(t)=V(t)\setminus F(t).
$$

For UAV positions $\mathbf p_i(t)$ and $\mathbf p_j(t)$, their separation is

$$
d_{ij}(t)=\left\|\mathbf p_i(t)-\mathbf p_j(t)\right\|_2.
$$

A candidate edge $e_{ij}$ is useful only when its endpoints are active **and** the current link satisfies the configured physical and communication constraints.

---

## Communication model

### Link budget

The analytical layer uses the received-power balance

$$
P_{\mathrm{rx}}
=
P_{\mathrm{tx}}
+
G_{\mathrm{tx}}
+
G_{\mathrm{rx}}
-
L_{\mathrm{total}},
$$

where all gain/loss terms are represented consistently in the configured logarithmic units.

Thermal-noise power is approximated as

$$
N_{\mathrm{dBm}}
=
-174
+
10\log_{10}(B)
+
NF,
$$

where $B$ is channel bandwidth and $NF$ is receiver noise figure.

The theoretical channel-capacity abstraction is

$$
C_{ij}
=
\eta B
\log_2\!\left(1+\mathrm{SINR}_{ij}\right),
$$

where $\eta$ is an efficiency factor associated with the selected model.

> **Interpretation:** $C_{ij}$ is a theoretical link metric used by the simulator. It is not presented as measured application goodput.

### Link-feasibility gate

A compact representation of the runtime gate is

$$
g_{ij}(t)
=
\mathbf 1_{\{d_{ij}\le d_{\max}\}}
\mathbf 1_{\{\mathrm{SINR}_{ij}\ge \gamma\}}
\mathbf 1_{\{C_{ij}\ge C_{\min}\}}
\mathbf 1_{\{\Delta t_{ij}\le T_{\mathrm{fresh}}\}}.
$$

Failure-aware feasibility additionally requires active endpoints:

$$
g^{F}_{ij}(t)
=
g_{ij}(t)
\mathbf 1_{\{i\in V_a(t)\}}
\mathbf 1_{\{j\in V_a(t)\}}.
$$

Only feasible active hops can advance the authoritative packet state.

---

## Why the feasibility gate matters

For each active hop, packet progression can depend on:

- endpoint activity;
- route validity;
- operational and hard-outage range;
- metric freshness;
- SNR/SINR threshold;
- capacity threshold;
- antenna and world-state validity.

A failed predicate pauses the corresponding packet progression and records the reason rather than reporting a visually connected route as an operational one.

| Topology mode | Runtime interpretation |
|---|---|
| **Chain** | one infeasible active hop pauses the end-to-end stream |
| **Parallel** | independent branch cursors continue while their own paths remain feasible |
| **Forest** | state is maintained per subtree / branch |
| **Manual** | operator-defined edges are preserved but remain subject to the same physical and communication gate |

---

## System architecture

~~~mermaid
flowchart LR
    A[Mission / Scenario] --> B[Mission Control]
    B --> C[ROS 2 state + revisions]
    C --> D[Isaac Sim vehicle/world state]
    C --> E[Link-service request]
    D --> E
    E --> F[Sionna-compatible propagation]
    F --> G[SNR / SINR / capacity / delay]
    G --> H[Feasibility gate]
    H --> I[Routing + packet runtime]
    I --> J[Failure / recovery logic]
    J --> C
    C --> K[Evidence + metrics]
    G --> K
    I --> K
~~~

The architecture is transactional: topology, swarm, antenna, traffic, world, failure, and recovery changes create a revision that is validated, applied, acknowledged, and only then considered committed.

---

## Operational evidence

The following demonstrations are linked to distinct parts of the runtime rather than repeated as a generic gallery.

### 1. Embodied multi-UAV scenario

<p align="center">
  <a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-city.mp4">
    <img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-preview.webp" width="760" alt="Urban multi-UAV SWARMSYM simulation">
  </a>
</p>

This clip shows the swarm operating inside the simulated environment used to couple vehicle state, network evaluation, and mission logic.

### 2. Dynamic topology

<p align="center">
  <a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-topology.mp4">
    <img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-topology.webp" width="760" alt="SWARMSYM dynamic topology visualization">
  </a>
</p>

Topology changes are evaluated against the current communication state rather than accepted only because a new edge has been drawn.

### 3. Relay state and recovery

<p align="center">
  <a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-relay-state.mp4">
    <img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-recovery.webp" width="760" alt="SWARMSYM failure and recovery behavior">
  </a>
</p>

A recovery is considered successful only after the replacement path is synchronized and satisfies the configured feasibility conditions.

> Click an image to open the associated MP4.

---

## Failure and recovery sequence

~~~text
nominal service
→ fault injected / detected
→ failed endpoint removed from active graph
→ affected path becomes infeasible
→ candidate topology / standby state proposed
→ link metrics recomputed
→ feasibility gate evaluated
→ ROS 2 + link service + Isaac Sim acknowledge the revision
→ service resumes only after the replacement path is feasible
~~~

This sequence distinguishes **topological reconfiguration** from **validated service recovery**.

---

## Model fidelity

The platform keeps model fidelity explicit:

| Level | Interpretation |
|---|---|
| **F0** | layout / mission preview |
| **F1** | analytical communication and motion abstractions |
| **F2** | stochastic channel, traffic, failure, and uncertainty studies |
| **F3** | geometry-aware Sionna RT adapter |
| **F4** | external protocol-aware co-simulation |
| **F5** | optional autopilot execution |

Results from different fidelity levels should not be merged without preserving their model provenance and assumptions.

---

## Metrics

Representative outputs include:

**Communication**
- received power and path loss;
- SNR / SINR;
- theoretical capacity;
- delay and jitter;
- packet delivery ratio;
- throughput / goodput when produced by the corresponding runtime layer;
- outage and utilization;
- queue state and Age of Information.

**Topology**
- connected components;
- hop count;
- graph diameter and degree;
- articulation points and bridges;
- path diversity and disjoint paths;
- algebraic connectivity;
- topology churn;
- failed-node tolerance.

Each metric is associated with timestamp, source, units, model/fidelity, freshness, and experiment provenance whenever those fields are available.

---

## Runtime stack

- **Mission Control** — scenario design, topology, swarm control, traffic, failures, telemetry, synchronization, diagnostics, and evidence.
- **ROS 2** — typed state coordination, revisions, packet runtime, vehicle state, failure events, and acknowledgements.
- **Sionna-compatible link service** — propagation, received power, SNR/SINR, capacity, delay, feasibility, and model provenance.
- **Isaac Sim** — vehicle/world embodiment and scene acknowledgement.
- **Evidence layer** — configurations, hashes, events, CSV metrics, manifests, logs, plots, and support bundles.

PX4 SITL is optional and isolated from the core execution path.

---

## Running the platform

### Clean installation

~~~bash
cd ~/NETLAB
chmod +x scripts/netlab scripts/*.sh Docker/scripts/*.sh Docker/workspace/ros2/*.sh
./scripts/bootstrap_host.sh --non-interactive
~~~

### Daily operation

~~~bash
cd ~/NETLAB
./scripts/netlab launch
./scripts/netlab status
./scripts/netlab packet-doctor
./scripts/netlab sync-doctor
./scripts/netlab smoke-test
./scripts/netlab stop
~~~

Mission Control is served on port 8765.

---

## Authoritative lifecycle

~~~text
PREFLIGHT
→ REPAIRING
→ BUILDING
→ STARTING_MISSION_CONTROL
→ STARTING_SIONNA
→ WAITING_FOR_SIONNA
→ STARTING_ROS
→ WAITING_FOR_ROS_GRAPH
→ WAITING_FOR_PACKET_RUNTIME
→ STARTING_ISAAC
→ WAITING_FOR_ISAAC_SCENE
→ SYNCHRONIZING
→ SMOKE_TESTING
→ READY / RUNNING
~~~

Every bounded wait exposes the expected signal, latest observation, elapsed time, timeout, retry state, and relevant logs.

---

## Repository structure

~~~text
apps/mission_control/       Mission Control backend and frontend
netlab/                     State, synchronization, models, runtime and feasibility logic
Docker/                     Compose, Isaac, ROS 2, Sionna, optional PX4 profile
plugins/                    Research algorithm packages
scenarios/                  Validated experiments and regression scenarios
schemas/                    Experiment, plugin, and API contracts
openapi/                    Mission Control API specification
reports/                    Validation, performance, and security reports
tests/                      Unit, integration, and scientific tests
docs/                       Architecture, operator, developer, and research documentation
~~~

---

## Verification

~~~bash
PYTHONPATH=. python3 -m unittest discover -s tests -p 'test_*.py' -v
python3 tests/run_all.py
./scripts/diagnostics/validate_release.sh
./scripts/netlab target-acceptance --embedded
~~~

On the target system:

~~~bash
./scripts/netlab target-acceptance
~~~

---

## Documentation

- [System architecture](docs/architecture/system_architecture.md)
- [Synchronization protocol](docs/architecture/synchronization_protocol.md)
- [Algorithm benchmark protocol](docs/research/algorithm_benchmark_protocol.md)
- [Research playbook](docs/research/research_playbook.md)
- [Mathematical model](docs/research/mathematical_model.md)
- [Model credibility](docs/research/model_credibility.md)
- [Metrics catalog](docs/reference/metrics_catalog.md)
- [Known limitations](docs/research/known_limitations.md)
- [Validation report](VALIDATION.md)

## Scientific scope

SWARMSYM is a simulation and experimentation framework. Analytical capacity, modeled link quality, simulated packet behavior, and embodied vehicle motion are kept distinct so that model outputs are not presented as measured real-world network performance unless an experiment explicitly supports that interpretation.

## License

See [LICENSE](LICENSE).
