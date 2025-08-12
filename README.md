# RHINO: Hypergraph-based Motion Generation with Multi-modal Interaction & Relational Reasoning

**RHINO** is a two-stage framework for **multi-agent, multi-modal motion prediction** that performs explicit **group-wise relational reasoning**. It couples a graph-based predictor (**GIRAFFE**) with a **multi-scale hypergraph** encoder and a **CVAE**-driven generator to produce diverse, socially consistent futures.

> 📄 Paper (preprint): *Hypergraph-based Motion Generation with Multi-modal Interaction Relational Reasoning*  
> 💻 Code: https://github.com/keshuw95/RHINO-Hypergraph-Motion-Generation

<p align="center">
  <img src="docs/figs/framework_architecture.jpg" width="85%" alt="Framework">
</p>

---

## Highlights

- Introduces **RHINO** for hypergraph-based, multi-agent, multi-modal motion prediction with explicit relational reasoning.  
- Captures **group-wise interactions** via a **multi-scale hypergraph** that models joint influence.  
- Uses an **agent–behavior graph** to disentangle the multi-modality of driving behaviors.  
- Performs **intention-conditioned reasoning** over hyperedges.  
- Learns **higher-order interaction patterns** and yields **human-interpretable explanations**.  
- Delivers **accurate multi-future trajectory predictions** with an **efficient residual decoder**.

---

## Framework at a Glance

RHINO is organized as **predict → reason → generate**:

1) **GIRAFFE (Predict)**  
   Learns pairwise interactions and outputs **multi-modal candidate futures** for all agents. It expands each agent into **behavior nodes** (e.g., L/K/R) to keep modes disentangled.

2) **RHINO (Reason & Generate)**  
   Lifts candidates into **multi-scale hypergraphs** (hyperedges = groups), performs **intention-conditioned message passing**, and uses a **CVAE** with a **residual decoder** to sample **K** plausible futures per scene.

<p align="center">
  <img src="docs/figs/RHINO.jpg" width="85%" alt="RHINO overview">
</p>

---

## Why Hypergraphs (vs. pairwise graphs)?

Real traffic is often **many-to-many**: a single maneuver can induce coordinated responses across several vehicles and lanes. Pairwise graphs approximate this with multiple edges; **hypergraphs** encode it **directly** as one hyperedge connecting the whole group.

- **Many-to-many influence:** one hyperedge models a coordinated behavior.  
- **Multi-scale context:** small groups (gap acceptance) → larger platoons (shockwave propagation).  
- **Cleaner reasoning:** messages flow through group contexts rather than ad-hoc pairwise mixing.

---
## Agent–Behavior Graph & Agent–Behavior Hypergraph

**Agent–Behavior Graph (ABG).**  
Each agent is expanded into **behavior nodes** (e.g., L/K/R lateral modes). This disentangles multi-modality at the node level, so messages are passed **per intention** rather than averaged across incompatible futures. Edges still encode pairwise influence, but now **conditioned on mode**. Infeasible modes are masked (no edges/messages), preventing impossible behaviors from contaminating reasoning.

**Agent–Behavior Hypergraph (ABH).**  
We lift ABG into a hypergraph where **hyperedges connect behavior nodes across multiple agents** that are likely to act jointly (e.g., zipper-merge group, platoon, courtesy-yield set). This directly encodes **many-to-many, intention-aware** influence. In RHINO:
- Hyperedges are inferred at **multiple scales** (small to large groups) from trajectory-affinity.
- **Intention-conditioned** message passing flows node→hyperedge→node.
- The **incidence matrix** exposes which behavior nodes participate in which groups, aiding interpretability.

<p align="center">
  <img src="docs/figs/def_hypergraph.jpg" width="70%" alt="Graphs vs. hypergraphs (definitions)">
</p>

---

## Core Components

### 1) GIRAFFE: Graph-based Multi-modal Predictor
- **Interaction encoder:** DGCN over forward/backward transitions for asymmetric, multi-hop influence.  
- **Intention head:** L/K/R with **feasibility masks**.  
- **Multi-modal decoder:** GRU+MLP yields per-mode trajectory distributions.  
- **Agent–Behavior Graph:** Expands each agent into behavior nodes to prevent “mode averaging.”

> Output: multi-agent, multi-modal candidates + intention probabilities.

### 2) Multi-scale Hypergraph Relational Encoder
- **Topology inference:** Builds **S** hypergraph scales by grouping agents with high trajectory-affinity.  
- **Node→Hyperedge→Node** message passing with:
  - **Neural interaction strength** per hyperedge,  
  - **Gumbel-softmax interaction type** selection (intention-conditioned),  
  - **Per-type MLPs** capturing motifs (cooperative merge, competitive cut-in, platooning).  
- **Outputs:** scale-stacked embeddings encoding both **fine** (pairwise) and **coarse** (group) dynamics.

<p align="center">
  <img src="docs/figs/hypergraph_encoder_1.jpg" width="75%" alt="Multi-scale topology">
  <img src="docs/figs/hypergraph_encoder_2.jpg" width="75%" alt="Hypergraph message passing">
</p>

### 3) Posterior Distribution Learner (CVAE)
- Learns a **latent space** $Z$ for stochastic futures given history + candidates.  
- **ELBO** training balances reconstruction fidelity and calibrated uncertainty.  
- At inference, sample $Z$ to draw **K** diverse, plausible trajectories.

<p align="center">
  <img src="docs/figs/posterior_distribution_learner.jpg" width="80%" alt="Posterior learner">
</p>

### 4) Residual Motion Generator
- Two **residual** GRU+MLP blocks:
  - **Block 1:** initial past reconstruction + coarse future.  
  - **Block 2:** refinement using reconstruction residuals (stabilizes long horizons).  
- Produces **K** futures per agent; enforces temporal smoothness and physical plausibility.

<p align="center">
  <img src="docs/figs/motion_generator.jpg" width="70%" alt="Motion generator">
</p>

---

## Intention-Conditioned Reasoning

- The agent–behavior graph provides **mode-specific nodes** per agent.  
- RHINO conditions hyperedge reasoning on **predicted intentions** (e.g., neighbors’ L/K/R).  
- This aligns group messages with **maneuver combinations** likely to occur, avoiding “blurry” averages over incompatible futures.

---

## Interpretability by Design

RHINO exposes several **human-readable** artifacts:

- **Hyperedge polygons**: which vehicles grouped together at each scale.  
- **Affinity matrix (N×N)**: pairwise correlation structure.  
- **Incidence matrix (nodes×hyperedges)**: vehicle–group membership.  
- **Interaction types** (Gumbel-softmax): soft labels for motifs (yielding, zipper merges, platooning).  

These views help explain **why** a forecast was produced (and flag odd groupings).

<p align="center">
  <img src="docs/figs/result_hypergraph_22.jpg" width="75%" alt="Hypergraph inference example">
</p>

---


## Citation

If you find **RHINO** (and **GIRAFFE**) useful in your research, please cite:

```bibtex
@article{wu2024hypergraph,
  title={Hypergraph-based motion generation with multi-modal interaction relational reasoning},
  author={Wu, Keshu and Zhou, Yang and Shi, Haotian and Lord, Dominique and Ran, Bin and Ye, Xinyue},
  journal={arXiv preprint arXiv:2409.11676},
  year={2024}
}
```

---

## Acknowledgments

Our multi-scale hypergraph design and group-wise interaction reasoning were inspired in part by **GroupNet** from MediaBrain@SJTU.  
Repo: https://github.com/MediaBrain-SJTU/GroupNet