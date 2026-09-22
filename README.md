# SWARMSYM / NETLAB — Swarm Network-as-a-Service Research Platform

<p align="center">
  <strong>Communication-aware multi-UAV simulation · ROS 2 coordination · Sionna link evaluation · Isaac Sim embodiment · failure-aware recovery</strong>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-architecture.webp" alt="SWARMSYM architecture" width="900">
</p>

SWARMSYM/NETLAB is a modular research platform for studying UAV swarms that act as reconfigurable airborne communication infrastructure. The central research question is whether the swarm can maintain a communication service while vehicles move, links degrade, nodes fail, and the network topology is reconfigured.

The platform couples four layers that are often evaluated separately:

1. vehicle state and motion through ROS 2 and Isaac Sim;
2. wireless-link evaluation through Sionna-compatible channel models;
3. routing, topology, and service logic through failure-aware graph operations;
4. experiment evidence through synchronized revisions, telemetry, metrics, and reproducible run artifacts.

---

## Research model

At time t, the swarm is represented by the time-varying graph

[
G(t)=(V(t),E(t)),
]

with failed nodes F(t) and active nodes

[
V_a(t)=V(t)setminus F(t).
]

For UAV positions p_i(t) and p_j(t), the Euclidean separation is

[
d_{ij}(t)=|p_i(t)-p_j(t)|_2.
]

A link is not treated as usable merely because two vehicles are geometrically close. The execution gate also checks endpoint activity, route validity, metric freshness, operational and hard-outage ranges, SNR or SINR, and capacity thresholds.

### Analytical link budget

The analytical model documented in this repository uses

[
P_{rx}=P_{tx}+G_{tx}+G_{rx}-L_{total},
]

with thermal-noise power approximated in dBm by

[
N=-174+10log_{10}(B)+NF,
]

and theoretical link capacity

[
C=eta Blog_2(1+mathrm{SINR}).
]

Here B is channel bandwidth, NF is receiver noise figure, and eta is an efficiency factor associated with the configured abstraction. Capacity is retained as a theoretical link metric; it is not labeled as achieved application throughput.

A compact interpretation of the feasibility gate is

[
g_{ij}(t)=
mathbf{1}{	ext{range valid}}
mathbf{1}{mathrm{SNR/SINR}gegamma}
mathbf{1}{C_{ij}ge C_{min}}
mathbf{1}{	ext{metric fresh}},
]

followed by failure-aware endpoint activity,

[
g^{F}_{ij}(t)=g_{ij}(t),
mathbf{1}{i,jin V_a(t)}.
]

Only feasible active hops can advance the authoritative packet cursor.

---

## Why the communication gate matters

A path drawn in a visualization is not automatically considered operational. For each active hop, packet advancement requires:

- active, non-failed endpoints;
- a valid route;
- current link metrics;
- operational and hard-outage range compliance;
- SNR/SINR above the configured threshold;
- capacity above the configured threshold;
- valid antenna and world state.

A failed predicate pauses the authoritative packet progression and records the reason.

| Mode | Runtime interpretation |
|---|---|
| Chain | one infeasible active hop pauses the end-to-end stream |
| Parallel | independent branch cursors continue when their own paths remain feasible |
| Forest | state is maintained per subtree / branch |
| Manual | operator-defined edges are preserved but remain subject to the same physical and communication gate |

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

The architecture is transactional: topology, swarm, antenna, world, traffic, failure, and recovery changes produce a revision that is validated, applied, acknowledged, and only then considered committed.

---

## Visual experiments

<table>
<tr>
<td width="33%" align="center">
<a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-city.mp4">
<img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-preview.webp" width="100%" alt="Urban swarm simulation">
</a><br><b>Urban multi-UAV scenario</b>
</td>
<td width="33%" align="center">
<a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-topology.mp4">
<img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-topology.webp" width="100%" alt="Dynamic topology">
</a><br><b>Topology evolution</b>
</td>
<td width="33%" align="center">
<a href="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/media/projects/swarmsym-relay-state.mp4">
<img src="https://raw.githubusercontent.com/kosmicplane/kosmicplane.github.io/main/assets/images/research/kaust-recovery.webp" width="100%" alt="Failure recovery">
</a><br><b>Relay state and recovery</b>
</td>
</tr>
</table>

Click a panel to open the corresponding MP4.

---

## Runtime stack

- Mission Control — scenario design, topology, swarm control, traffic, failures, telemetry, synchronization, diagnostics, and evidence.
- ROS 2 — typed state coordination, packet runtime, revision application, vehicle state, failure events, and acknowledgements.
- Sionna-compatible link service — path loss, received power, SNR/SINR, capacity, delay, feasibility, and model provenance.
- Isaac Sim — embodied vehicle/world execution and scene acknowledgement.
- Evidence layer — versioned configuration, hashes, events, CSV metrics, manifests, logs, plots, and support bundles.

PX4 SITL is optional and isolated from the core execution path.

---

## Model fidelity and scientific interpretation

The platform labels results by model fidelity rather than merging outputs from different abstractions as if they were equivalent:

- F0 — layout preview;
- F1 — analytical communication and motion abstractions;
- F2 — stochastic channel, traffic, failure, and uncertainty studies;
- F3 — geometry-aware Sionna RT adapter;
- F4 — external protocol-aware co-simulation;
- F5 — optional autopilot execution.

Each model is intended to record its intended use, assumptions, validity domain, units, uncertainty, calibration status, verification evidence, and limitations.

---

## Metrics

Representative quantities include:

Communication: packet state, PDR, throughput, goodput, capacity, delay, jitter, queues, SNR/SINR, received power, path loss, spectral efficiency, outage, utilization, handover, and Age of Information.

Topology: connected components, hop count, path stretch, churn, graph diameter, degree, density, articulation points, bridges, disjoint paths, algebraic connectivity, path diversity, redundancy, continuity, and failed-node tolerance.

Each metric is associated with source, units, fidelity, timestamp, freshness, quality, model version, and uncertainty metadata.

---

## Failure and recovery sequence

~~~text
nominal service
→ fault injected / detected
→ failed endpoint removed from active graph
→ affected path becomes infeasible
→ candidate topology / standby state proposed
→ link metrics recomputed
→ feasibility gate applied
→ ROS 2 + link service + Isaac Sim acknowledge one revision
→ service resumes only after the replacement path is feasible
~~~

This distinction prevents a visual route change from being reported as a successful recovery before the underlying communication constraints are satisfied.

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

Every wait is bounded and exposes the expected signal, last observation, elapsed time, timeout, retry state, and relevant logs.

---

## Repository structure

~~~text
apps/mission_control/       Mission Control backend and frontend
netlab/                     State, synchronization, models, runtime and feasibility shield
Docker/                     Compose, Isaac, ROS 2, Sionna, optional PX4 profile
plugins/                    Research algorithm packages
scenarios/                  Validated experiments and regression scenarios
schemas/                    Experiment, plugin, and API contracts
openapi/                    Mission Control API specification
reports/                    Validation, performance and security reports
tests/                      Unit, integration and scientific tests
docs/                       Architecture, operator, developer and research documentation
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

## License

See [LICENSE](LICENSE).
